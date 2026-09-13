# Debugger Spec (Learning Branch)

Status: locked, v2 — incorporates ADR-0001, ADR-0002, ADR-0003

Part of the [learning branch plan](./learning-branch-plan.md). The learning
branch's debugger is NEMU `sdb` parity plus breakpoints. Other branches may
adopt or extend this spec later; until then it is owned by the learning
branch.

## 1. Commands

| Command | Format | Behavior |
|---|---|---|
| help | `help` | Print command help |
| continue | `c` | Run until breakpoint, watchpoint, halt, or Ctrl-C |
| quit | `q` | Exit |
| step | `si [N]` | Execute N instructions (default 1), then pause |
| registers | `info r` | Print all 32 regs with ABI names + pc (GDB-style layout) |
| watchpoints | `info w` | List pool entries whose expression does not mention `$pc` |
| breakpoints | `info b` | List pool entries whose expression mentions `$pc` |
| expression | `p EXPR` | Evaluate and print EXPR |
| scan memory | `x N EXPR` | Print N 4-byte words at address EXPR, hex |
| watchpoint | `w EXPR` | Pause when EXPR's value changes |
| delete | `d N` | Delete pool entry N (indices are compacted) |
| breakpoint | `b ADDR` | Add breakpoint at address ADDR |

All entries live in one pool with one index space. `d N` deletes pool entry
`N` regardless of which command created it, and indices are compacted after
every deletion: `0 <= N < count` always holds, so deleting `0` renumbers every
later entry (ADR-0001, ADR-0003). `b` and `w` remain separate commands
because their input syntax differs — an address versus an expression — not
because their storage differs.

## 2. Implementation note (teaching decision)

Breakpoints are stored internally as watchpoints on `$pc == ADDR` — one
unified mechanism, with `b` as sugar. This makes the "breakpoint is a special
watchpoint" insight concrete. `w $pc == 0x80000000` and `b 0x80000000` behave
identically.

Which entries are presented as breakpoints is derived, not stored: an entry
appears under `info b` if and only if its expression mentions `$pc`
(ADR-0002). `w $pc >= 0x1000` and `w *($pc + 4) == 0x1234` are therefore
listed as breakpoints, although neither is an address comparison.

Classification is presentation only. Every entry is evaluated by one rule:
evaluate the expression before each instruction, and halt when its value
differs from the value stored at the previous check. The label never selects
the checking discipline — otherwise `w $pc >= 0x1000` would halt before every
instruction in the region instead of once when execution enters it.

## 3. Expression grammar

Documented precedence, low to high:

1. `||`
2. `&&`
3. `|`
4. `^`
5. `&`
6. `==` `!=`
7. `<` `<=` `>` `>=`
8. `<<` `>>`
9. `+` `-`
10. `*` `/` `%`
11. unary `+` `-` `!` `~` `*` (dereference)
12. primary: decimal, hex (`0x...`), `$reg` (`$0`..`$31`, ABI names, `$pc`), `(expr)`

## 4. Evaluation semantics

- `u32` wrapping arithmetic (matches CPU semantics).
- Division/modulo by zero: evaluation error, command aborts with a message —
  never a crash.
- Dereference reads 4 bytes little-endian from memory; out-of-bounds is an
  error, not a panic.
