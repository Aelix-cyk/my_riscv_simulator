# Development Process

Status: draft for review

Applies to every branch. This file is shared material: edit it on `main`, then
merge `main` into a branch to refresh that branch's copy. The decisions
themselves live in [docs/adr/](./adr/README.md).

## 1. Shared and branch-local material

| Tier | Contents | Edited where |
|---|---|---|
| Shared | `README.md`, scaffold (`Cargo.toml`, `.gitignore`, placeholder `src/`), `project-plan.md`, `process.md`, project-scope ADRs | `main` |
| Branch | branch plan, branch specs, branch-scope ADRs, the real `src/` | that branch |

Rules:

1. Edit a shared file on `main` only; shared changes reach a branch by
   merging `main` into it.
2. Edit a branch file on its branch only. Product branches are never merged
   into `main`.
3. Promotion — a branch artifact that should bind every branch — is a
   deliberate commit on `main` that adds the artifact and updates its
   `Scope`. Promotion is never a branch merge.
4. Editing a shared file on a product branch does not fail immediately; it
   conflicts at the next merge from `main`. That is why rule 1 exists.

## 2. ADRs

- One decision per record, in `docs/adr/`, named `NNNN-slug.md`.
- Header fields: `Status`, `Date`, `Scope`, `Related`; add `Supersedes` when
  replacing an earlier record.
- Numbering is one sequence for the whole repository, never reused and never
  renumbered. Before allocating a number on a branch, merge `main` in and
  take the highest existing number plus one.
- Statuses: `proposed` while under review, `accepted` once the decision is in
  force. A rejected proposal is deleted.
- A rejected proposal may be deleted only after its reasoning is preserved
  elsewhere — normally in the accepted record's "Alternatives considered"
  section, or in a commit. A deleted uncommitted file is unrecoverable.
- Immutability begins at first commit. Before that a record is still in
  review and may be rewritten. Afterwards only the status line and the
  `Supersedes` links may change.
- A decision earns a record when any of these holds: a competent reader would
  ask "why is it done this way?"; undoing it would touch several files or a
  locked spec; or a genuinely tempting alternative was rejected.
- There is no index table. The filenames are the index; a table would be
  edited on `main` and on branches, and would conflict at every merge.

## 3. Specs

A spec states what is true now, for its scope — project or branch.

- Header: `Status: <draft|locked>, vN`, listing the ADRs it incorporates, for
  example `Status: locked, v2 — incorporates ADR-0001, ADR-0002`.
- `locked` means changing the spec requires an accepted ADR first. It does not
  mean the spec never changes.
- An accepted ADR that changes a spec is applied by editing the spec in place
  and bumping `vN`. The spec never narrates its own history: git holds the
  exact diff, the ADR holds the why.
- Public specs follow the ADR tier rule: they live on `main`; branch specs
  live on their branch. There are no public specs yet — write one when
  something genuinely binds more than one branch, not to fill the tier.

## 4. Open items

- Version-to-code traceability: nothing yet tells a reader whether the code
  on a branch matches spec `vN` or `vN-1`. Candidates: a line in the branch's
  README, or a module header. Decide before M2 of the learning branch.
