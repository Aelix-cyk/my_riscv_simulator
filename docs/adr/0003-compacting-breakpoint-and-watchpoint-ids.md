# ADR-0003: Breakpoint and watchpoint ids are compacted after deletion

- Status: accepted
- Date: 2026-09-13
- Scope: learning
- Amends: [debugger-spec.md](../debugger-spec.md) section 1
  (numbering rule for `d N`)
- Related: [ADR-0001](./0001-unified-watchpoint-pool.md),
  [ADR-0002](./0002-breakpoint-classification-by-derivation.md)

## Context

With one pool and a shared `d N` (ADR-0001), deleting entry `N` forces a
choice about the numbers of the entries after it: renumber them, or leave
the gap.

The user sees these numbers in `info b` and `info w` and types them back as
`d N`, so the rule is directly observable and cannot be left implicit.

## Decision

Ids are compacted. After any deletion, entries are renumbered from `0` in
pool order, so `0 <= N < count` always holds. This matches NEMU's behavior.

Breakpoints have no separate numbering: they share the pool and the index
space (ADR-0001), so a breakpoint id is compacted by the same rule.

## Alternatives considered

- **Stable monotonic ids, never reused**: rejected for this branch. It removes
  the stale-reference surprise, but adds a counter to the pool, produces
  listings with gaps, and lets ids grow while the pool shrinks — noise in a
  debugger whose pool is typically tiny. Worth revisiting only if stale ids
  cause real confusion during M2.
- **Tombstones** (deleted entries keep their slot, marked dead): rejected.
  Strictly more state than stable ids, with the same gappy listing, and no
  benefit at this scale.

## Consequences

- An index written down before a deletion can refer to a different entry
  afterwards. This is documented behavior, not a bug, and it must be stated
  in the spec so it is not mistaken for an off-by-one error.
- Combined with ADR-0001's filtered views, the surprise is amplified: deleting
  index `0` while looking at `info w` renumbers a breakpoint to `0` under
  `info b`. A kind indicator in the listing is what makes this readable.
- Scripted REPL tests in the testing strategy must not assume indices are
  stable across deletions; a script that deletes and then references an index
  should re-read the listing between the two steps.

## Applied

1. `debugger-spec.md` section 1: the numbering rule is stated where `d N` is
   documented.
