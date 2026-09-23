# Batch 70 — all raw mode171 gameplay host-load candidates are fresh-unreachable

Date: 2026-09-23

Scope: mode-aware reachability audit using the independent child arrays from Batch67 and already verified fresh traces. No gameplay/Roblox execution.

Batch68 found 14 raw mode171 opcode1 rows whose host index belongs to the known runtime/gameplay wrapper set and whose destination register is inside descriptor capacity. They occur only in child133 (1), child240 (7), child394 (4), and child446 (2).

Mode171 opcode1 is a direct host load:

```lua
R[B[pc]] = b[O[pc]]
```

child133, child394 and child446 are blocked at their normalized mode2 opcode128/O0 fresh entry because R0 is nil.

child240 is different: Batch69 proves a real mode2 -> mode171 -> mode51 path. Its only reached mode171 segment is PC614..619. None of the seven raw gameplay-load candidates (PC423, 561, 576, 578, 670, 1206, 1211) lies on that segment. PC619 switches to mode51, and the fresh path later stops at mode51 PC706 because opcode199 requires parent capture F[2].

Verdict:

```text
raw mode171 runtime+valid candidates: 14
fresh-reachable as mode171 host loads: 0
```

Artifact: `b70_mode171_gameplay_candidates.tsv`.
