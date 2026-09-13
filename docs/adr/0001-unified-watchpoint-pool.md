# ADR-0001: Breakpoints and watchpoints share one pool and one delete command

- Status: accepted
- Date: 2026-09-13
- Scope: learning
- Amends: [debugger-spec.md](../debugger-spec.md) section 1
  (removes `bd N`), section 2
- Related: [ADR-0002](./0002-breakpoint-classification-by-derivation.md),
  [ADR-0003](./0003-compacting-breakpoint-and-watchpoint-ids.md)

## Context

`debugger-spec.md` v1 lists `bd N` for deleting breakpoints and `d N` for
deleting watchpoints, while its section 2 states that breakpoints are stored
as watchpoints on `$pc == ADDR`. One storage structure is therefore exposed
through two delete commands, which leaves the identifier `N` undefined:

- With a single numbering space, `d` and `bd` are redundant, and a mismatched
  `N` has no defined meaning.
- With two numbering spaces, `d 0` and `bd 0` address different entries, and
  a mismatched `N` must either error or silently delete from the other kind.

The choice also fixes how much the user must remember and whether commands
transfer directly from NEMU, whose `d N` deletes from one pool.

## Decision

One pool of watchpoints, one global index space.

- `d N` deletes pool entry `N`, regardless of whether it was created by `b`
  or by `w`.
- `b` and `w` remain separate creation commands, because their input syntax
  differs (address vs. expression). The split is justified by parsing, not by
  storage.
- `info b` and `info w` remain, as filtered views over the same pool. Both
  print global pool indices; which entries they show is defined by ADR-0002.
- `bd` is removed.

The guiding principle: split commands when the input syntax differs, unify
them when the identifier is shared.

## Alternatives considered

- **Two numbering spaces with `d` and `bd`** (spec v1 as written): rejected.
  It implies a distinction the storage does not have, makes `d 0` ambiguous
  relative to whichever view the user last read, and forces a rule for
  mismatched `N` that no user can predict.
- **One pool, one `d`, one listing (`info w`) with a kind column, no `info b`**:
  not rejected on principle; deferred. A smaller command surface, at the cost
  of the quick filter and of NEMU muscle memory. Revisit if the filtered views
  prove more confusing than useful.

## Consequences

- One delete path to implement and test; `d N` works no matter which view was
  printed last.
- Filtered views show non-contiguous indices (for example `0:` and `2:`),
  which looks like a bug until the rule is known. Listings need a kind
  indicator so a single view is still readable on its own.
- Spec v1 users lose `bd`; users following NEMU's `d`-for-everything habit are
  unaffected.
- The command table in `debugger-spec.md` changes, and the change is recorded
  here rather than in that document's own history.

## Applied

1. `debugger-spec.md` section 1: `bd N` removed; `d N` documented as deleting
   from the shared pool, with the numbering rule.
2. `debugger-spec.md` section 2: shared index space noted.
