# ADR-0004: `main` holds shared material; each product goal gets its own branch

- Status: accepted
- Date: 2026-09-13
- Scope: project
- Supersedes: the "Decisions (locked)" section of `project-plan.md`. Replaces
  an earlier draft of this record, which made `main` the OS-booting
  implementation; rewritten before first commit (see process.md)
- Related: `project-plan.md` sections 1 and 3

## Context

The simulator serves three product goals that pull the implementation in
different directions: booting an OS, teaching user-level RV32I, and running
real compiled user programs. They need three implementations, because the
learning simulator is deliberately built differently — readability over
brevity, no privileged machinery ever.

Two things then need a home:

1. Material that binds every branch — the project plan, the process rules,
   project-wide decisions.
2. The implementations themselves.

An earlier draft of this record put the OS-booting implementation on `main`.
That fails both counts. `main` is the default branch and the ancestor of
every product branch, so a product implementation there is simultaneously
ancestor and sibling; and project-wide material was left living on
`learning`, which made one product branch the owner of project-wide truth.

## Decision

- `main` holds shared material only: `README.md`, the scaffold
  (`Cargo.toml`, `.gitignore`, a placeholder `src/`), `docs/project-plan.md`,
  `docs/process.md`, and project-scope ADRs. It holds no product
  implementation.
- Each product goal gets its own branch, created from `main`:
  - `os-booting` — the OS-booting simulator, the flagship. Grows through all
    tiers, user-level instructions to privileged machinery and devices.
    Created lazily from `main` when OS-tier work begins.
  - `learning` — the teaching-oriented RV32I user-level simulator. First, and
    currently active.
  - `user-programs` — runs real compiled user programs; stops before
    privileged machinery. Created lazily from `main`.
- Product branches never merge into `main`. Shared material flows one way:
  `main` into the branch. A branch-local artifact that should become shared
  is promoted by a deliberate commit on `main`, with its `Scope` updated.
- The product branches are deliberately different implementations, not
  feature subsets of one another. Duplication between them is accepted as
  the cost of keeping each honest.

## Alternatives considered

- **`main` as the OS-booting implementation** (the earlier draft): rejected.
  `main` is the ancestor of every product branch; putting a product
  implementation there makes it ancestor and sibling at once, and leaves
  project-wide material homeless.
- **No shared home — project docs live on `learning`**: rejected. One product
  branch would own project-wide truth, contradicting the rule that the
  rulebook is the same everywhere.
- **A single `main` with milestone tags per tier**: rejected. The learning
  branch is a different internal design, so one history would misrepresent
  the relationship between the versions.
- **One Cargo workspace, three crates in one branch**: rejected. Code sharing
  between the implementations is minimal by design, and a workspace invites
  exactly the accidental shared abstractions the branch split exists to
  prevent.

## Consequences

- Only the scaffold is shared. Each product branch's source layout diverges
  from its first commit, and `main` builds a placeholder crate.
- Shared files are edited on `main` and reach branches by merging `main` in.
  Editing a shared file on a product branch does not fail immediately; it
  conflicts at the next merge. The rule exists to prevent that.
- Reading `main` tells you what binds every branch and nothing about the
  implementations; reading a product branch tells you that product's scope
  and progress.
- `learning` was branched before the scaffold landed, so it must merge `main`
  once to receive it. Later product branches are created from a `main` that
  already has it.
- Reversal: if one product implementation must become the ancestor of
  another, that is a new ADR superseding this one.
