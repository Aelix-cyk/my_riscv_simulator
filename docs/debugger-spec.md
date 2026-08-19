# Debugger Spec (Learning Branch)

Status: locked

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
| watchpoints | `info w` | List numbered watchpoints (expression + current value) |
| breakpoints | `info b` | List numbered breakpoints |
| expression | `p EXPR` | Evaluate and print EXPR |
| scan memory | `x N EXPR` | Print N 4-byte words at address EXPR, hex |
| watchpoint | `w EXPR` | Pause when EXPR's value changes |
| delete watchpoint | `d N` | Remove watchpoint N |
| breakpoint | `b ADDR` | Add breakpoint at address ADDR |
| delete breakpoint | `bd N` | Remove breakpoint N |

## 2. Implementation note (teaching decision)

Breakpoints are stored internally as watchpoints on `$pc == ADDR` — one
unified mechanism, with `b` as sugar. This makes the "breakpoint is a special
watchpoint" insight concrete. `w $pc == 0x80000000` and `b 0x80000000` behave
identically.

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
