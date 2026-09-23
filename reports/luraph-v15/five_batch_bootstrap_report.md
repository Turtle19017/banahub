# Luraph v15 sample — five bounded bootstrap batches

Scope: static/sample-local VM analysis only. No gameplay, Roblox, network, or execution of the original VM payload.

Ground truth input for this continuation:
- verified `root_correct.bin` SHA-256: `d65760495a18cbcf6ea82ca2c861952555c351b2e356cab95054cda371cb0c3c`
- recovered instruction arrays: 5,018 rows
- starting point: Astra's verified stop at PC3 after `(91,88,31,7852) -> (118,62,63,62)`

The Python reproducer implements the exact uint32 arithmetic for mode-2 opcodes 91 and 107 from factory 2801 (`bit32.band/rshift/lshift/bxor/bnot` plus the sample's `A1` 32-bit multiplication helper).

## Batch 1 — resolve opcode 118 at PC3

PC3 initially:

```text
(91, O=88, B=31, p=7852)
```

Opcode 91 self-decodes its own row to:

```text
(118, O=62, B=63, p=62)
```

Mode-2 opcode 118 has source semantics:

```lua
J,u = B[W],O[W]
L = {["n"] = u-J+1}
table.move(registers,J,u,1,L)
registers[p[W]] = L
```

For this row, `J=63`, `u=62`, so the range is empty. The result is:

```text
R62 = { n = 0 }
next PC = 4
```

This resolves the exact blocker identified in the Astra report.

## Batch 2 — PC4..PC7 and trampoline back to PC8

PC4 self-decodes through opcode 91:

```text
(91,238,32,8527) -> (9,136,0,93)
R93 = 136
```

PC5 opcode 99 consumes PC5 and PC6 as a paired assignment:

```text
R58  = R14
R128 = 7
next PC = 7
```

PC7 self-decodes through opcode 107:

```text
(107,42,9884,4936) -> (124,0,0,4977)
```

Correction to an earlier interpretation: **mode-2 opcode 124 is a direct jump**, `W=p[W]`. It does not call an iterator.

Therefore:

```text
PC7 opcode124: W=4977
loop increment -> PC4978
PC4978 opcode162: W=R128=7
loop increment -> PC8
```

So this path is fully deterministic.

## Batch 3 — register shuffles and second instruction XOR layer

PC8/9, PC10/11, PC12/13 are paired register moves through opcode 110:

```text
R1 = R126
R2 = R11
R3 = R125
R4 = R50
R5 = R108
R6 = R66
```

PC14 is:

```text
(34, O=23, B=4944, p=57)
```

Mode-2 opcode 34 performs:

```text
for s = 1..4944:
    pc  = 23+s             # PC24..PC4967
    key = (57 XOR s) & 127
    X[pc] ^= key
    O[pc] ^= key
    B[pc] ^= key
    p[pc] ^= key
X[14] = 92
```

This establishes the **second instruction-unmasking layer** over PC24..PC4967.

Example after layer 2:

```text
PC24   -> (87,114,42,3118)
PC4960 -> (62,7,109,0)
PC4967 -> (171,171,0,68)
```

## Batch 4 — continuation target 4959

PC15 self-decodes through opcode 107:

```text
(107,4981,7196,185) -> (9,4959,0,128)
R128 = 4959
```

PC16 uses the same canonical opcode-107 self-decoder as PC7:

```text
(107,42,9884,4936) -> (124,0,0,4977)
```

Then:

```text
PC16 opcode124 -> W=4977
increment       -> PC4978
PC4978 opcode162 -> W=R128=4959
increment       -> PC4960
```

Again, no dynamic branch is required to reach PC4960.

## Batch 5 — verified return boundary at PC4960

After the second XOR layer:

```text
PC4960 = (62, O=7, B=109, p=0)
```

The factory source for mode-2 opcode 62 is exactly:

```lua
return l[p[W]], w[W]
```

Thus the current VM invocation returns:

```text
return R0, w[4960]
```

There is no normal PC increment after this instruction because it returns from the VM closure.

The value of `w[4960]` is not recovered in this five-batch pass; identifying the `w` constant array requires continuing the q/prototype constant decoding rather than executing more mode-2 instructions.

## Deterministic path added by these five batches

Starting at Astra's previous blocker:

```text
PC3 (91)
 -> PC3 (118)
 -> PC4 (91)
 -> PC4 (9)
 -> PC5 (99; consumes PC6)
 -> PC7 (107)
 -> PC7 (124)
 -> PC4978 (162, R128=7)
 -> PC8 (110; consumes PC9)
 -> PC10 (110; consumes PC11)
 -> PC12 (110; consumes PC13)
 -> PC14 (34, second XOR)
 -> PC15 (107)
 -> PC15 (9, R128=4959)
 -> PC16 (107)
 -> PC16 (124)
 -> PC4978 (162, R128=4959)
 -> PC4960 (62)
 -> RETURN R0, w[4960]
```

## Corrections / confidence

High confidence for this sample:
- opcode 91 and 107 self-decode arithmetic reproduced directly from source using uint32 semantics;
- opcode 118 empty pack at PC3;
- opcode 124 = direct jump;
- opcode 162 = indirect jump through `R[B]` for this mode/source branch;
- second XOR layer PC24..PC4967;
- opcode 62 at PC4960 = actual VM return boundary.

Not established in this pass:
- semantic meaning/value of `R0`;
- value/type of `w[4960]`;
- whether another caller invokes this closure again or consumes the two return values;
- full OPAL/ONYX opcode mapping.
