# Batch 50 — root R0 remains nil through the verified bootstrap

Date: 2026-09-23

Scope: bounded dataflow audit over the already verified root bootstrap path.

## Goal

The verified return boundary at PC4960 is:

```lua
return R0, w[4960]
```

This batch asks whether any instruction on the actual traced path writes R0 before that return.

## Register initialization

Factory2801 creates a fresh register table on invocation:

```lua
local R = table.create(register_capacity)
```

No fill value is supplied, so R0 begins nil.

The root entry is PC4994.

## Executed writes on the verified path

The path contains only the following register writes before PC4960:

```text
PC4994: R15 = table.pack(select(1,...))

PC4995/4996:
  R96 = R1
  R97 = R86

PC4997/4998:
  R98 = R89
  R73 = R110

PC5000: R128 = 16

PC17/18:
  R92 = R28
  R68 = R65

PC19/20:
  R99 = R27
  R9  = R67

PC21/22:
  R100 = R24
  R128 = 4967

PC4968/4969:
  R101 = R41
  R102 = R18

PC4970/4971:
  R103 = R28
  R104 = R13

PC4972/4973:
  R105 = R86
  R106 = R71

PC4974:
  R12 = R91

PC4975/4976:
  R10  = R54
  R128 = 0

PC1/2:
  R95 = R40
  R16 = R14

PC3/op118:
  R62 = {n=0}

PC4:
  R93 = 136

PC5/6:
  R58  = R14
  R128 = 7

PC8/9:
  R1 = R126
  R2 = R11

PC10/11:
  R3 = R125
  R4 = R50

PC12/13:
  R5 = R108
  R6 = R66

PC15:
  R128 = 4959
```

The remaining executed instructions are self-decoders, instruction-array XORs, direct jumps, indirect jumps, or the final return.

**No executed instruction writes R0.**

## Return state

After the second broad XOR, PC4960 is:

```text
opcode62 O=7 B=109 p=0
```

and mode2 opcode62 returns:

```lua
return R[p[pc]], w[pc]
```

Therefore the first return value is:

```text
R0 = nil
```

for the verified invocation state.

Batch26 independently resolved the second value as:

```text
w[4960] = 4244215898
```

with no lookup/payload mutation during that overlay access.

So the bounded return is:

```text
nil, 4244215898
```

## Verdict

```text
R0 initialized before dispatch: no
R0 written on verified bootstrap: no
R0 at PC4960: nil
```

Artifact:
- `b50_r0_dataflow.json`
- `b50_r0_dataflow.txt`

This initially appears surprising relative to the outer `:H()(...)` syntax; Batch51 audits that call chain directly.
