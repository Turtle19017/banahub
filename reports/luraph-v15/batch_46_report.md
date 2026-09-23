# Batch 46 — dormant third broad-XOR layer at PC4966

Date: 2026-09-23

Scope: offline instruction-state reconstruction.

## State after the two verified broad XOR layers

Reconstructing from the raw 5018 instruction arrays gives:

```text
PC4966 = (X=34, O=23, B=4936, p=123)
PC4967 = (X=104, O=171, B=0, p=68)
```

In mode2 (and equivalently mode81 for opcode34), PC4966 is another broad instruction XOR:

```text
target PCs = 24..4959
key(s) = (123 XOR s) & 127
X/O/B/p[target] ^= key
X[4966] = 92
```

Thus it is a structurally valid *third* broad-unmask candidate if execution ever reaches PC4966 while in a compatible mode.

## Effect on PC1394

PC1394 after the first two global layers is:

```text
(34,99,32,34)
```

For PC1394, `s = 1394-23 = 1371` and:

```text
(123 XOR 1371) & 127 = 32
```

Applying the PC4966 layer yields:

```text
PC1394: (34,99,32,34)
      -> (2,67,0,2)
```

Opcode2 is not a nested-closure constructor in the relevant factory modes, so even this additional broad layer does not turn PC1394 into the child113 instantiation site.

## Alias-set effect

The artifact `b46_third_layer_alias_effect.tsv` records the same hypothetical PC4966 transform for every currently verified child113 alias. None of the aliases becomes the known mode2 opcode158 or mode171 opcode159 closure constructor solely from this layer.

## Verdict

PC4966 is the only new broad-XOR candidate presently visible after the two global layers that covers PC1394. If it executes, it changes the alias-set substantially, but it still does not directly close the parent->child113 chain.

The next question is reachability: does the verified bootstrap ever execute PC4966?
