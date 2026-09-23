# Batches 73–75 summary

This pass closes the previous child240 capture-only hypothesis and narrows parent provenance.

- Batch73 mechanically replays the child240 mutation chain from the independently parsed arrays. After PC704, PC706 is `F[2] -> R9`, PC707 is a second capture dereference `F[1] -> R11`, PC708 creates `R13={}`, and PC709 is opcode93. The structural PC942 host load remains `R24=b[26]`.
- Batch74 re-runs the q lazy-overlay oracle against the exact 92,516-byte E buffer. At transformed PC709, `C[709]="GetChildren"` while `t[709]` attempts an out-of-range q lookup. Since opcode93 computes `R5=t[709]+C[709]`, child240 remains blocked even if valid F[1]/F[2] capture cells are supplied. Capture recovery alone therefore cannot make PC942 reachable.
- Batch75 instruments nested q references originating from every `w[pc]` slot in eight completely scanned descriptors (4,971 slots total). The 37 nested references target only symbols 113, 191, 133, and 187; none targets symbol240.

Refined frontier: stop treating child240 `F[2]` as the sole bridge blocker. The next high-value target is the actual parent/pre-entry mutation protocol capable of changing child descriptor/lazy state or control flow, while continuing the bounded `w`-overlay parent-reference census over the remaining descriptors/root.
