# Project Plan

Status: living document

## 1. Overview

`my_riscv_simulator` — a RISC-V simulator written in Rust, developed as three
product lines, one git branch each:

- `main` — OS-booting version. The flagship; grows through all tiers
  (user-level instructions → privileged machinery → MMU/devices).
- `learning` — teaching-oriented simulator, user-level RV32I only. Currently
  the active branch. Plan: [learning-branch-plan.md](./learning-branch-plan.md).
- `user-programs` — runs real compiled user programs; stops before privileged
  machinery. Created lazily when that tier actually starts. Plan: future
  `user-programs-plan.md`.

The three are deliberately **different implementations**, not feature subsets
of each other. This is why they are branches rather than milestones on one
line (see Decisions).

## 2. Decisions (locked)

- Three branches, one per goal, because the implementations genuinely differ.
- `main` is the OS-booting version.
- `learning` starts first (active now).
- `user-programs` is created lazily from `main` when that tier begins.
- Alternative considered: a single `main` with milestone tags per tier.
  Rejected: the learning branch is a different internal design, so a single
  history would misrepresent the relationship between the versions.

## 3. Branch & Repo Workflow

1. Shared scaffold commit on `main` (`cargo init`, README, `.gitignore`).
2. `git checkout -b learning` from that commit. Learning work happens only on
   that branch; `main` stays clean until OS-tier work starts.
3. `user-programs` created lazily from `main`.
4. Per-branch plan docs live in `docs/`. This file holds project-wide
   context; branch files hold branch-specific scope only.

## 4. Shared Environment & Toolchain

- Rust 1.97 (cargo 1.97.1) installed.
- `clang` available — can assemble RISC-V object files for small test
  programs.
- Not installed: GNU RISC-V cross-toolchain (`gcc-riscv64-unknown-elf`),
  `spike`, `llvm-mc`. Any branch that runs the official `riscv-tests` needs
  the GNU toolchain; install when first required (currently learning M4).
  Toolchain installation may require approval.

## 5. Status

- `main`: initial commit + README only; scaffold pending.
- `learning`: plan drafted; M0 pending.
- `user-programs`: not yet created.
