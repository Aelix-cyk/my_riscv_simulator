# ADR-0002: Breakpoints are classified by a derived predicate

- Status: accepted
- Date: 2026-09-13
- Scope: learning
- Amends: [debugger-spec.md](../debugger-spec.md) section 1
  (`info b` / `info w` semantics)
- Related: [ADR-0001](./0001-unified-watchpoint-pool.md),
  [ADR-0003](./0003-compacting-breakpoint-and-watchpoint-ids.md)

## Context

Because breakpoints are stored as watchpoints on `$pc == ADDR`, the debugger
must decide which pool entries appear under `info b`. Two mechanisms are
available: remember how the entry was created (an origin tag), or compute the
answer from the expression (a derived predicate).

An origin tag would make `b` a category with a hidden field rather than
syntax sugar: `b 0x1000` and `w $pc == 0x1000` produce the same stored entry
but would be listed in different views depending on how they were typed.

## Decision

Classification is derived, never stored. An entry is listed under `info b`
if and only if its parsed expression mentions `$pc` at all. Nothing about how
the entry was created is remembered.

As a result, all of these appear under `info b`:

```
b 0x1000
w $pc == 0x1000
w $pc >= 0x1000
w $pc != $a0
w *($pc + 4) == 0x1234
```

**Invariant:** classification is presentation-only. Behavior is defined by a
single evaluation rule applied to every entry — evaluate the expression, and
halt when its value differs from the value stored at the previous check. The
label must never select the checking discipline. If it did,
`w $pc >= 0x1000`, being labeled a breakpoint, would halt before every
instruction in the region instead of once when the pc first enters it.

## Alternatives considered

- **Exact shape match, `Eq(Reg(pc), Imm)` (plus a commutativity rule for
  `0x1000 == $pc`)**: rejected. It would file `w $pc >= 0x1000` as a plain
  watchpoint although it is a condition on the pc, contradicting the rule
  chosen here. It also demands a normalisation rule for parentheses and
  operand order.
- **Origin tag** (remember the creating command): rejected. It makes the
  classification depend on typing history, sends identical stored entries to
  different views, and weakens the "`b` is sugar for `w`" invariant that
  section 2 of the spec exists to teach.

## Consequences

- The `info` views depend on the shape of parsed expressions, so the debugger
  listing is coupled to parser internals. Contained by keeping the predicate
  in one function rather than spreading the test across the views.
- That predicate has two consumers — the listings and the run loop. They must
  call the same function, or `info b` and actual behavior can disagree.
- `info b` can contain entries that are not addresses, and entries whose
  `$pc` mention is incidental. `w *($pc + 4) == 0x1234` is a watch on memory
  at the pc's current address, not a watch on the pc; it is nevertheless
  listed as a breakpoint. This is the accepted cost of the rule.
- The run-loop description in `learning-branch-plan.md` section 4
  ("watchpoints evaluated before and after each instruction; breakpoints
  checked against `pc` before execution") describes two mechanisms. Under
  this ADR there is one: evaluate before each instruction and compare against
  the stored previous value, which subsumes the before/after pair.

## Applied

1. `learning-branch-plan.md` section 4: one evaluation rule covers every
   entry, and `StepEvent` no longer carries a breakpoint/watchpoint kind.
2. `debugger-spec.md` sections 1 and 2: the derived-predicate rule is stated
   where the views are documented.
