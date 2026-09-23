# Batch 69 — child240 multi-mode trace stops before its post-unmask gameplay host-load

Date: 2026-09-23

Scope: bounded fresh-state trace of child240 using the independently parsed arrays plus exact factory semantics. No gameplay/Roblox execution.

Child240 metadata:

```text
entry PC1446
mode2
regs 26
rows 1452
```

## Mode2 bootstrap and first local unmask

```text
PC1446/op9    -> R8 = 28
PC1447/op110  -> paired register moves (no control dependency)
PC1449/op110  -> paired register moves
PC1451/op34   -> XOR/self-patch PC5..1445
PC1452/op104  -> switch mode2 -> mode171, W=614
```

## Short mode171 segment

After the PC1451 layer, PCs614..618 are opcode56 register moves. PC619 becomes:

```text
mode171 opcode126
O=51 B=683 p=0
```

Mode171 opcode126 switches mode:

```text
mode = 51
W = 684
```

Thus the raw mode171 gameplay-load candidates previously seen elsewhere in child240 are not encountered on this entry segment.

## Mode51 control / local decode chain

Mode51 uses `O[pc]` as the opcode field.

Known state: `R8=28`.

```text
PC684/op109: 28 <= 30 -> jump to PC485
PC485/op13 : R22 = (R8 <= 14) = false
PC486/op110: XOR/self-patch PC688..1386
PC487/op158: R22=false -> branch to PC882
PC882/op109: 28 <= 22 false -> PC883
PC883/op94 : R18 = 997
PC884/op166: jump through X=0 -> PC1
PC1/op102 : self-decodes to (X=18,O=75,B=0,p=0)
PC1/op75  : W = R18 = 997 -> PC998
PC998/op13: R23 = (R8 <= 26) = false
PC999/op110: XOR/self-patch PC692..881
PC1000/op158: R23=false -> PC701
PC701/op13: R7 = (R8 <= 28) = true
PC702/op158: R7=true -> PC695
PC695/op110: XOR/self-patch PC703..752
PC696/op110: XOR/self-patch PC692..694
PC697/op158: R7=true -> PC703
PC703/op13: R24 = (R8 <= 27) = false
PC704/op110: XOR/self-patch PC706..752
PC705/op158: R24=false -> PC706
```

After the PC704 transform, PC706 is mode51 opcode199:

```lua
J = F[X[706]]      -- X=2
R[p[706]] = J[7][J[6]]
```

So execution requires parent capture cell `F[2]`. A fresh q-returned child240 has no parent-built capture vector, and the bounded trace stops here.

## Gameplay bridge candidate beyond the stop

After all local transforms that have actually executed before the stop, a later row remains structurally interesting:

```text
PC942: mode51 opcode147
X=26, p=24
=> R24 = b[26]
```

Host id 26 is in the Batch55 runtime/gameplay wrapper set. Its destination register 24 is valid for a 26-register descriptor.

However PC942 is not reached: fresh execution stops at PC706 on missing `F[2]`, 236 PCs earlier.

Verdict:

```text
child240 demonstrates real mode2 -> 171 -> 51 execution
local self-unmask layers are reachable and materially change host-load candidates
post-unmask gameplay host-load PC942 exists structurally
fresh reachability to PC942: NO (blocked at capture F[2])
```

Artifact: `b69_child240_trace.tsv`.
