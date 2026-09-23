# Batch 75 — bounded parent-prototype search finds no child240 reference in 4,971 w-overlay slots

Date: 2026-09-23

Scope: q-overlay reference census over a bounded descriptor subset. No factory/gameplay execution.

## Why scan `w`

Both verified nested-closure constructors use the lazy `w[pc]` operand as the child descriptor:

```text
mode2  opcode158: J = w[pc]
mode171 opcode159: J = w[pc]
```

Therefore a parent instruction that directly materializes child240 through the current q overlay must eventually cause a `w[pc]` access to invoke `q(host, 240)`.

## Instrumented scan

The q method was wrapped only after the parent descriptor had been materialized. During each `w[pc]` access, nested q calls were logged before child construction and aborted, avoiding side effects from recursively materializing the referenced prototype.

Completed descriptors:

```text
133: 527 rows
138: 134
140: 45
162: 23
181: 34
185: 154
188: 22
191: 4032
----------------
total: 4971 w slots
```

Observed nested q calls: 37. Referenced symbols:

```text
113: 28
191: 5
133: 2
187: 2
240: 0
```

So none of these eight fully scanned q-returned descriptors directly materializes child240 through its `w` overlay. This subset is especially useful because child191 alone accounts for 4,032 rows and many prior raw closure-looking rows.

## Boundary

This is not a complete parent-map proof. The remaining descriptors, the root descriptor, and any parent-side mutation before the `w` access remain candidates. The result does, however, remove 4,971 concrete `w` slots from the search space.

Verdict: `NO_CHILD240_W_QREF_IN_SCANNED_8_DESCRIPTOR_SUBSET`.

Artifact: `b75_w_overlay_qrefs.tsv`.
