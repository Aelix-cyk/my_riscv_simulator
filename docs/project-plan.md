# Project Plan

Status: living document

## 1. Overview

`my_riscv_simulator` — a RISC-V simulator written in Rust. Shared material
lives on `main`; each product goal gets its own branch, created from `main`.
Branch-local documents exist only on their own branch, so this plan names
them as paths rather than linking to them:

- `main` — shared material only: README, scaffold, this plan, the process
  rules, and project-scope ADRs. No product implementation.
- `os-booting` — the OS-booting simulator, the flagship. Grows through all
  tiers: user-level instructions → privileged machinery → MMU/devices. Not
  yet created (section 5).
- `learning` — teaching-oriented simulator, user-level RV32I only. Branch
  plan: `docs/learning-branch-plan.md` on that branch.
- `user-programs` — runs real compiled user programs; stops before privileged
  machinery. Created lazily from `main`. Branch plan
  (`docs/user-programs-plan.md`) is written on that branch.

The product branches are deliberately **different implementations**, not
feature subsets of each other. This is why they are branches rather than
milestones on one line (see section 2).

## 2. Decisions

Project-wide decisions are recorded as ADRs under `docs/adr/` — index and
status in [adr/README.md](./adr/README.md). The rationale, the rejected
alternatives, and the consequences belong there; this plan carries scope and
status only.

- Branch model (`Scope: project`):
  [ADR-0004](./adr/0004-shared-main-one-branch-per-goal.md) — shared material
  on `main`, one branch per product goal.
- Branch-scope decisions (`Scope: learning`, and later the other product
  branches) live in that branch's `docs/adr/`. The learning branch's debugger
  decisions — ADR-0001 to ADR-0003, accepted and incorporated as its debugger
  spec v2 — are recorded on that branch.

## 3. Branch & Repo Workflow

1. Scaffold: `cargo init` produces `Cargo.toml`, `.gitignore`, and a
   placeholder `src/`. It lives on `main` and is shared by every branch.
   `learning` was branched before it landed, so `learning` merges `main` once
   to receive it.
2. Product branches are created from `main` and never merged back.
   `learning` was branched from `main` at `4f76348`; `os-booting` and
   `user-programs` are created lazily, from a `main` that already has the
   scaffold.
3. Shared material — this file, `process.md`, project-scope ADRs, the
   scaffold — is edited on `main` only, and reaches a branch by merging `main`
   into it. Branch material — branch plan, branch specs, branch-scope ADRs,
   `src/` — is edited on its branch only.
4. Per-branch plan docs live in `docs/`; this file holds project-wide
   context, branch files hold branch-specific scope only.
5. Promoting branch material to shared is a deliberate commit on `main` with
   the record's `Scope` updated — never a merge of a product branch.

## 4. Shared Environment & Toolchain

- Rust 1.97 (cargo 1.97.1) installed.
- `clang` available — can assemble RISC-V object files for small test
  programs.
- Not installed: GNU RISC-V cross-toolchain (`gcc-riscv64-unknown-elf`),
  `spike`, `llvm-mc`. A branch that needs them states the requirement and the
  milestone in its own plan; installation may require approval.

## 5. Status

Single source of project-level status. Branch plans carry branch progress
only.

- `main`: shared material only — README, scaffold, this plan, `process.md`,
  `adr/README.md`, and `adr/0004-*`. Unpushed relative to `origin/main`.
- `learning`: active, and in sync with `main`. Shared material plus its
  branch plan, debugger spec v2, and ADR-0001 to ADR-0003, all committed.
  M0 not started.
- `os-booting`: not created. Created from `main` when OS-tier work begins.
- `user-programs`: not created.

Open items: none at present.
