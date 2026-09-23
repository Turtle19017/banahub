# Batch 73 — exact replay extends child240 past the old F[2] frontier structurally

Date: 2026-09-23

Scope: offline replay of child240 instruction-array mutation only. No Roblox/gameplay execution.

## Goal

Batch69 stopped at mode51 PC706 because opcode199 needs parent capture `F[2]`. This batch replays every mutation layer actually executed before that point directly from the independently parsed Batch67 arrays and the exact factory2801 transform semantics.

## Replayed mutation layers

The child begins at PC1446 and executes the already established path into mode51. The following mutation instructions were applied mechanically in order:

```text
mode2  PC1451/op34   -> XOR rows 5..1445
mode51 PC486/op110   -> XOR rows 688..1386
mode51 PC999/op110   -> XOR rows 692..881
mode51 PC695/op110   -> XOR rows 703..752
mode51 PC696/op110   -> XOR rows 692..694
mode51 PC704/op110   -> XOR rows 706..752
```

For mode2/op34 the per-row key is `(p[pc] XOR i) & 127`; for mode51/op110 it is `(B[pc] XOR i) & 127`. All four direct arrays X/O/B/p are XORed.

The replay independently reproduces the Batch69 checkpoints, including PC619 -> mode51, PC706/op199, and PC942/op147.

## New post-transform rows

After the final PC704 layer:

```text
PC706: X=2 O=199 B=0 p=9
       J=F[2]; R9=J[7][J[6]]

PC707: X=1 O=199 B=0 p=11
       J=F[1]; R11=J[7][J[6]]

PC708: X=0 O=144 B=0 p=13
       R13={}

PC709: X=42 O=93 B=122 p=5
       R5=t[709]+C[709]
```

Thus the old statement “child240 needs F[2]” was incomplete. If PC706 is satisfied, PC707 immediately requires a second capture cell `F[1]`.

The structural gameplay target remains:

```text
PC942: X=26 O=147 B=26 p=24
       R24=b[26]
```

Verdict: `CHILD240_REPLAY_REPRODUCED_AND_CAPTURE_FRONTIER_EXTENDED_TO_F2_THEN_F1_THEN_LAZY_PC709`.

Artifact: `b73_child240_postcapture_slice.tsv`.
