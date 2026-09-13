# Architecture Decision Records

One decision per record. Each record states its `Scope` — `project`, or the
product branch it belongs to — because shared and branch-local decisions sit
side by side in this directory but travel differently:

- `Scope: project` records are committed on `main` and reach branches by
  merging `main` in.
- Branch-scope records are committed on their branch only.

The filenames are the index: `NNNN-slug.md`, one global number sequence, never
reused. There is deliberately no table listing the records — such a table
would be edited on `main` and on every branch, and would conflict at every
merge.

Numbering, statuses, superseding, deleting rejected proposals, and spec
versioning are governed by [process.md](../process.md).
