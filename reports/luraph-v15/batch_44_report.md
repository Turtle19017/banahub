# Batch 44 — audit of statically visible closure-opcode candidates

Date: 2026-09-23

Scope: offline root instruction/overlay analysis.

## Goal

Test whether child113 can be tied directly to any root row whose opcode, after the two globally verified XOR layers, already equals a known closure-construction opcode.

Known factory closure branches relevant to a child descriptor in `w[pc]`:

- mode2: opcode158 uses `w[pc]` + `C[pc]`
- mode171: opcode159 uses `w[pc]` + `P[pc]`
- mode81 contains a `w[pc]` + `C[pc]` closure-construction branch reached by the specialized 133/135 range; specialization alone does not prove both numeric values are valid reachable opcodes.

## Static candidate census

After applying the two verified root XOR layers, only six rows have X in `{133,135,158,159}`:

| PC | X | O | B | p |
|---:|---:|---:|---:|---:|
| 950 | 159 | 102 | 123 | 19425 |
| 2280 | 133 | 23 | 4964 | 23 |
| 4487 | 135 | 85 | 100 | 215 |
| 4710 | 158 | 127 | 81 | 29 |
| 4781 | 135 | 14 | 131 | 7 |
| 4879 | 159 | 105 | 12 | 127 |

## Overlay audit

### PC4710 / opcode158

```text
w[4710] = -6
P[4710] = 2572873130
```

This independently reconfirms that the statically visible opcode158 row is not a usable mode2 closure site in the current recovered state: `w` is not a descriptor table.

### PC2280 / candidate 133

```text
w[2280] = 4244215898
P[2280] = 4244215898
```

No child descriptor.

### PC4487 / candidate 135

```text
P[4487] = 2635469944
C[4487] = ""
w[4487] = unresolved in current compatibility shim
```

No verified descriptor/capture pair.

### PC4781 / candidate 135

A genuine capture-shaped table appears:

```text
C[4781] = {
  153,3,
  106,3,
   68,3,
    9,3,
   99,3
}
```

This encodes five kind-3 shared/open parent-register captures.

However:

```text
w[4781] = 2450515686
```

so the companion `w` operand is numeric, not a child descriptor. Therefore this row cannot be promoted to a verified `w+C` child-construction site.

This is also useful evidence that a specialized branch shape does not imply the current row is valid/reachable under that VM mode.

### PC950 / PC4879 / opcode159

Their relevant lazy overlay paths exceed the current Lua compatibility shim, so neither is promoted. No evidence from these probes ties them to child113.

## Cross-check against child113 capture evidence

Batch43 established:

```text
w[1394] = child113 descriptor
C[1394] = {145,3}
```

but PC1394's two-layer static opcode is 34, not one of the statically visible closure candidates above.

Conversely, none of the six statically visible closure-opcode candidates currently exposes both:

```text
w = child113
and
capture companion = valid capture table
```

## Conclusion

The parent->child113 chain is now constrained but not closed:

```text
verified:
  child113 cached descriptor
  many root.w aliases
  PC1394 has child113 + capture {145,3}

not yet verified:
  reachable closure-construction opcode using those operands
```

Therefore at least one of these must still be true:

1. a later/local self-decode transforms a child113 alias row before execution;
2. the relevant execution uses a different VM mode/state than the simple two-layer static view;
3. PC1394's descriptor/capture operands are materialized data that are not consumed by that row in the currently visible state.

The next high-value task is reachability/self-decode analysis around the child113 alias set, rather than further blind lookup scanning.

Artifact: `b44_closure_opcode_candidates.out`.
