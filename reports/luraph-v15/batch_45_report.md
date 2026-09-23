# Batch 45 — PC1394 mode semantics audit

Date: 2026-09-23

Scope: offline factory/source + q-overlay analysis.

## Starting evidence

After the two globally verified XOR layers, root PC1394 is:

```text
X=34 O=99 B=32 p=34
```

The same PC has lazy operands:

```text
w[1394] = cached child113 descriptor
C[1394] = {145,3}
```

where `{145,3}` is a valid one-capture specification (kind 3, parent slot 145).

## Opcode 34 in all factory2801 modes

### mode 2

Opcode34 performs a range XOR over instruction arrays:

```text
for s=1..B[pc]:
    target = O[pc] + s
    key = (p[pc] XOR s) & 127
    O/B/p/X[target] ^= key
X[pc] = 92
```

It does not consume `w[pc]`, `C[pc]`, `P[pc]`, or `t[pc]`.

### mode 81

Opcode34 has the same range-XOR/self-patch behavior as mode2. It also does not consume the child/capture overlays.

### mode 171

Opcode34 is a call/pack operation over registers:

```text
J=B[pc]
u=O[pc]
L=p[pc]
result = table.pack(R[J](R[J+1]..R[J+u]))
move result into register range starting at J
```

It again does not consume `w[pc]` or `C[pc]`.

## Verdict

With `X[1394]=34`, **no valid dispatch mode interprets PC1394 as nested-closure creation**.

Therefore the verified data pair

```text
w[1394] = child113
C[1394] = {145,3}
```

is dormant at the current two-layer instruction state. To become a closure-instantiation site, PC1394 must be transformed again before execution (or those overlays are data reused for a different later state).

This sharply narrows the next task to additional self-decode/reachability transforms rather than more lookup-table discovery.
