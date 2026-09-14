# Open Questions

Working document for the learning branch. One question at a time; write the
answer under the question. When an answer is settled, it moves into an ADR or
into the branch plan, and the question is deleted from here.

## Q1. Does readability-over-brevity need an ADR?

Section 2 of the branch plan is deleted. Its three bullets all already appear
in the same document: readability in section 1, RV32I-only scope in section 3,
debugger parity in section 5. So nothing is lost — except the record of what
was rejected.

- Option A: write ADR-0005 (`Scope: learning`) with the alternatives — terse
  idiomatic Rust, table-driven decode, speed as a goal — and their
  consequences.
- Option B: leave it as the single sentence in section 1. No record.

Answer:

## Q2. What shape is `DecodedInsn`?

- Option A: one variant per mnemonic, e.g. `Addi { rd, rs1, imm }`, with a
  `match` in `execute` that mirrors the spec's instruction listing.
- Option B: a shared shape, e.g. `IType { op, rd, rs1, imm }`, used by every
  I-type ALU instruction, with `op` choosing the operation.

Answer:

## Q3. How do `i32` and `u32` meet at the interface?

Decode produces sign-extended immediates; execute does wrapping `u32`
arithmetic.

- Option A: `imm: i32` in `DecodedInsn`, converted with `as u32` inside each
  instruction in `execute`.
- Option B: `imm: u32`, already holding the sign-extended two's-complement
  pattern, so `execute` never converts.

Answer:
