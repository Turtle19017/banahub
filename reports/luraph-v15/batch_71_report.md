# Batch 71 — all raw mode81 gameplay host-load candidates are fresh-unreachable

Date: 2026-09-23

Scope: mode-aware fresh-state audit. This batch adds concrete traces for child138 and child339.

Batch68 found 35 raw mode81 opcode115 rows whose host index belongs to the runtime/gameplay wrapper set and whose destination register is in range. Mode81 opcode115 is:

```lua
R[B[pc]] = b[p[pc]]
```

Most candidate-bearing children are blocked at their normalized opcode128/O0 entry: 133, 185, 191, 200, 218, 278, 312, 334, 367, 374, 394, 435, 446.

Other already-bounded cases:
- child240: raw PC1448/1450 are consumed by mode2 paired moves at entry; real execution later reaches mode51 and stops at PC706/F[2].
- child357: fresh execution stops at mode2 PC56 on capture F[5].

## child138

```text
PC5/op9   -> R2=1
PC6/op115 -> R4=R10=nil (mode2 semantics)
PC7/op34  -> XOR PC10..134
PC8/op9   -> R1=108
PC9/op124 -> PC4/op162 -> PC109
PC109/op153 -> branch PC48
PC48/op153  -> fallthrough PC49
PC49/op9    -> R1=130
PC50/op124  -> PC4/op162 -> PC131
PC131/op153 -> branch PC57
PC57/op143  -> requires capture F[1]; stop
```

It never enters mode81.

## child339

```text
PC1/op9   -> R2=1
PC2/op110 -> paired move consumes PC3
PC4/op104 -> switch mode171, PC60
PC60/op10 -> R8=false
PC61/op28 -> PC50
PC50/op145 -> XOR PC53..55
PC51/op145 -> XOR PC5..49
PC52/op28 -> PC47
PC47/op72 -> branch PC35
PC35/op105 -> requires capture F[1]; stop
```

It also never enters mode81. Its raw PC3 mode81-looking row is consumed under mode2 and is never dispatched as a mode81 host load.

Verdict:

```text
raw mode81 runtime+valid candidates: 35
fresh-reachable as mode81 host loads: 0
```

Artifacts:
- `b71_mode81_gameplay_candidates.tsv`
- `b71_child138_trace.tsv`
- `b71_child339_trace.tsv`
