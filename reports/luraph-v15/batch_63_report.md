# Batch 63 — all remaining structurally valid raw mode2 host-load candidates fail before reachability

Date: 2026-09-23

Scope: close the seven-row structurally valid opcode81 candidate set from Batch59.

Batch60 handled child140, Batch61 child357, and Batch62 child378. The remaining valid-destination raw opcode81 candidates occur in child133, child394, and child435.

Their normalized entry states are:

```text
child133: entry PC323 -> opcode128 O=0 B=32 p=1
child394: entry PC495 -> opcode128 O=0 B=25 p=2
child435: entry PC30 raw opcode84
          -> opcode128 O=0 B=36 p=7
```

Factory2801 starts each invocation with an empty register table, so all three immediately require an invalid comparison against `R0=nil`:

```text
child133: 1 <= R0
child394: 2 <= R0
child435: 7 <= R0
```

Consequently their candidate host-load rows are not reached in fresh state:

```text
child133 PC168 -> b[20]  (coroutine.status)
child394 PC270 -> b[91]  (select)
child435 PC223 -> b[228]
child435 PC225 -> b[228]
```

Together with batches 60–62, this closes all seven raw mode2 opcode81 rows from Batch59 whose destination register was inside descriptor capacity: **0/7 are reachable in the q-returned fresh state**.

This does not rule out later descriptor mutation, parent capture setup, or a different mode making host-library access possible. It does remove the entire direct fresh-mode2 candidate set as evidence of a VM→host bridge.
