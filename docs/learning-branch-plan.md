# Learning Branch Development Plan

Status: draft for review (document status; project status is in
[project-plan.md](./project-plan.md) section 5)

Branch: `learning`, branched from `main` at `4f76348`. Project-level
context — goals, branch strategy, repo workflow, shared toolchain — lives in
[project-plan.md](./project-plan.md). This document covers the learning
branch only.

## 1. Purpose

A teaching-oriented RV32I user-level simulator written in Rust, with an
NEMU-style debugger plus breakpoints. The implementation favors readability
over brevity: code is written to be read.

## 2. Locked Decisions

- **Teaching clarity over brevity.** Small modules, comments explain *why*,
  no cleverness for its own sake.
- **The branch ends at RV32I user-level.** No privileged machinery, ever, on
  this branch.
- **Debugger = NEMU `sdb` parity, plus breakpoints.**

## 3. Scope

### In scope

- All 40 RV32I base instructions, user-level only.
- `fence` / `fence.i`: decoded and treated as no-ops (single hart, in-order,
  no memory model needed — note *why* in a comment).
- `ecall` / `ebreak`: decoded, then halt with an explanatory message (trap
  machinery is out of scope; halting honestly is better than faking it).
- Flat byte-addressable memory, little-endian, unaligned accesses supported.
- ELF32 loading for static, position-dependent executables (`ET_EXEC`).
- The debugger per [debugger-spec.md](./debugger-spec.md).
- Passing the `riscv-tests` user-level `rv32ui-p-*` suite.

### Out of scope (explicitly deferred)

- RV64, C (compressed), M, F/D extensions, Zicsr/Zifencei.
- Privileged modes, CSRs, trap handling, PMP.
- MMU / virtual memory.
- Devices (serial/UART, timer), interrupts.
- JIT, cycle-accuracy, performance tuning, multiple harts.
- Differential testing against Spike (optional later, not required for done).

## 4. Architecture

### Principles

- Module boundaries mirror the conceptual parts of a CPU.
- The debugger drives; the CPU is passive: `step()` + `state()`.
- No `unsafe`. No panics in normal execution paths — errors are returned or
  reported and the run stops gracefully.
- Dependency policy: std-only until M3 (Ctrl-C), then the small `ctrlc`
  crate. Line editing starts with plain stdin; `rustyline` is an optional
  later upgrade if arrow-key history is wanted.

### Module layout

```
src/
  main.rs            entry: CLI (image path, --trace), boot, enter REPL
  cpu.rs             Cpu: regfile (32 x u32 + pc), step(), run(), state()
  decode.rs          word -> DecodedInsn (pure, unit-testable)
  execute.rs         DecodedInsn + &mut Cpu + &mut Memory -> step effects
  memory.rs          flat Vec<u8>, unaligned-aware loads, little-endian
  loader.rs          minimal ELF32 loader (M3)
  sdb/
    mod.rs           REPL: prompt, command table, dispatch, Ctrl-C flag
    expr.rs          tokenizer + recursive-descent parser + evaluator
    watchpoint.rs    watchpoint & breakpoint lists + check hooks
```

### Key seams

- `enum StepEvent { Ran, Halted(u32), WatchpointFired(usize) }` — `run()`
  returns this; the debugger derives the breakpoint-or-watchpoint label from
  the entry's expression (ADR-0002) and decides what to print and do next.
- Every pool entry is evaluated before each instruction and compared with the
  value stored at the previous check; a change halts and returns control to
  the debugger. There is no second mechanism for breakpoints (ADR-0001,
  ADR-0002).
- **Halt convention:** the instruction `0x0000_006b` is invalid in RV32I, so
  it is reserved as a halt sentinel (`nemu_trap`, same trick as NEMU).
  Execution of it halts the CPU; register `a0` carries the exit code.

### Design choices and alternatives

- **Flat memory vs paged:** flat. Page tables are an OS-tier concern; this
  branch teaches instruction semantics, not address translation.
- **`match`-based decode vs table-driven dispatch:** `match`. Reading the
  decoder top-to-bottom mirrors the spec's own instruction listing — the
  teaching win. Table dispatch is a performance tier, deferred.
- **Recursive-descent expression parser vs NEMU's dominant-operator
  recursion:** recursive descent. Easier to read and unit-test in Rust.

## 5. Debugger Spec

Locked at v2. The full spec — command table, expression grammar, evaluation
semantics — lives in [debugger-spec.md](./debugger-spec.md). Highlights:
NEMU `sdb` parity plus breakpoints (`b` / `info b`); breakpoints are
implemented as watchpoints on `$pc == ADDR`, classified by a derived
predicate (ADR-0001 to ADR-0003).

## 6. Milestones

Each milestone ends in something you can see run.

### M0 — Skeleton, minimal core, REPL shell

- `cargo init`; module skeleton; `Cpu` with regfile and pc; `StepEvent` enum;
  decode/execute for a small subset (`addi`, `add`, a few ALU ops); halt
  sentinel handling.
- `sdb`: `help`, `q`, `si [N]`, `info r`, `c` (run to halt), `--trace` mode
  printing pc, instruction, and changed registers.
- **Acceptance:** hand-encoded 5-instruction program; `si` steps one at a
  time; `info r` shows ABI names and correct values; `c` runs to halt.

### M1 — Memory, full ALU/I-type/U-type, `x`

- `memory.rs`: bounds, unaligned access, little-endian loads/stores.
- Full I-type ALU (including shifts), `lui`/`auipc`, all loads/stores
  (`lb/lbu/lh/lhu/lw/sb/sh/sw`).
- `sdb`: `x N ADDR` with bare hex addresses first (mirrors NEMU's incremental
  approach); expression support arrives in M2.
- **Acceptance:** program sums an array in memory; `x` output matches the
  expected bytes.

### M2 — Control flow, full RV32I, expression evaluator, watchpoints

- All six branches, `jal`, `jalr`; `fence` no-op; `ecall`/`ebreak` halt with
  message.
- `expr.rs`: tokenizer, recursive-descent parser, evaluator with the full
  precedence table; unit tests for precedence and dereference.
- `sdb`: `p`, `x N EXPR`, `w`, `d`, `info w`, `b`, `info b`; watchpoint
  hooks wired into the run loop.
- **Acceptance:** hand-assembled `fib(10)` leaves 55 in `a0`; `b` at a loop
  address pauses `c`; `w $a0 == 55` fires; `p (1 + 2) * 3` and `p *0xADDR`
  evaluate correctly; precedence unit tests green.

### M3 — ELF loader, real programs, Ctrl-C

- `loader.rs`: parse ELF32 `ET_EXEC`, map `PT_LOAD` segments, set `pc` to
  entry.
- Toolchain: assemble test programs with `clang --target=riscv32` + `ld.lld`
  and a minimal linker script (verify availability first; fallback is
  installing `gcc-riscv64-unknown-elf` — see project-plan.md).
- Ctrl-C: `ctrlc` crate + `AtomicBool`; run loop returns to the prompt.
- **Acceptance:** a small compiled C/asm program computes a value and halts
  via `nemu_trap`; `c` runs to halt; Ctrl-C during an infinite loop returns
  to the prompt.

### M4 — riscv-tests

- Install the GNU RISC-V toolchain (see project-plan.md).
- Build the `riscv-tests` `rv32ui-p-*` subset.
- Harness: run each test, translate its pass/fail signal, print a summary,
  exit non-zero on failure.
- **Acceptance:** all `rv32ui-p-*` tests pass.

## 7. Cross-Cutting Implementation Decisions

| Topic | Decision | Why / note |
|---|---|---|
| Integer semantics | `u32` wrapping everywhere | RISC-V defines wraparound; debug builds panic on overflow otherwise |
| `x0` | Reads 0, writes ignored | Spec-mandated; dedicated unit test |
| Sign extension | Immediates and `lb/lh/lbu/lhu` per spec | Classic first-bug location; dedicated unit tests |
| Endianness | Little-endian | RISC-V convention |
| Unaligned access | Supported (byte-wise) | Spec-permitted; simpler than trapping |
| PC update | `pc += 4` after computing jump targets from old pc | JAL/JALR subtleties live here |
| Halt | Invalid instruction `0x0000_006b` = `nemu_trap`; exit code in `a0` | Same teaching trick as NEMU |
| Memory size | 128 MiB default, configurable | Out-of-bounds = reported error + halt, not panic |
| Dependencies | std only through M2; `ctrlc` at M3 | `rustyline` optional later |

## 8. Testing Strategy

- Unit tests per instruction, using the examples from the RISC-V spec.
- Precedence and parser tests for `expr.rs` (hand-computed expected values).
- One golden trace for `fib(10)` (register-by-register), then regression.
- Scripted REPL sessions (commands fed via stdin) with expected output, to
  test the debugger itself.
- `riscv-tests` (M4) as the final gate.

## 9. Definition of Done (Learning Branch)

1. All `rv32ui-p-*` riscv-tests pass.
2. Debugger has the full command set of [debugger-spec.md](./debugger-spec.md),
   including breakpoints, watchpoints, expression evaluation, Ctrl-C, and
   `--trace`.
3. Code is teaching-grade: small modules, comments explain *why*, no
   `unsafe`, no panics in normal execution paths.
4. The `learning` branch's README documents usage, architecture, and how to
   assemble test programs.

## 10. Known Risks

- **Immediate sign-extension / field-split bugs** (B/S/J immediates) — the
  most likely source of silent wrongness; mitigated by early unit tests.
- **`riscv-tests` harness convention** (`tohost`/signature plumbing) — set
  aside focused time; it is fiddlier than the CPU.
- **Scope creep** — the temptation to add UART, difftest, or RV64 is real;
  all of it is explicitly deferred to other branches.
- **Toolchain availability at M3/M4** — shared dependency, see project-plan.md.

## 11. Immediate Next Steps

1. M0 implementation — see section 6. Scaffold placement is tracked in
   [project-plan.md](./project-plan.md) section 5.
