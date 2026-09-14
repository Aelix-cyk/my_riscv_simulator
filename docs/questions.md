# Open Questions

Working document for the learning branch. One question at a time; write the
answer under the question. When an answer is settled, it moves into an ADR or
into the branch plan, and the question is deleted from here. Numbers are
stable: a settled question is deleted, leaving a gap.

## Q4. Where do the shift instructions go?

`slli`, `srli`, and `srai` are I-type encodings but carry a shift amount, and
`srli`/`srai` differ only in `funct7`, so they do not fit `AluImm`. Needed at
M1, not M0.

- Option A: separate variants, `Slli { rd, rs1, shamt }` and so on.
- Option B: one `Shift { kind, rd, rs1, shamt }` variant.

Answer:
