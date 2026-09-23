# Batch 54 — child113 is not materialized by the verified top-level execution

Date: 2026-09-23

Scope: direct q-oracle test around the only lazy overlay access on the verified root path.

## Goal

Earlier structural exploration found many root.w aliases that can materialize tag09 symbol113 into a 788-row child descriptor.

The verified top-level root path, however, only performs one lazy-overlay read before returning:

```text
w[4960]
```

This batch checks whether root q parsing or that final overlay read indirectly materializes symbol113.

## Direct oracle result

Initial lookup value:

```text
lookup[113] = 47868
```

After parsing the root prototype with q:

```text
lookup[113] = 47868
type = number
```

So ordinary root q parsing does not materialize child113.

Before reading w[4960]:

```text
rawget(w,4960) = nil
```

Reading it returns:

```text
w[4960] = 4244215898
```

Afterward:

```text
rawget(w,4960) = 4244215898
lookup[113] = 47868
```

Thus the access caches the resolved scalar inside the root's local `w` proxy, but it does not touch symbol113.

## Consequence

The verified top-level path never accesses any of the known child113 aliases:

```text
333,356,545,570,577,744,776,782,786,872,1120,1250,1394,...
```

and the final `w[4960]` read does not cause an indirect symbol113 decode.

Therefore:

```text
child113 materialized during verified top-level call: NO
child113 factory invoked during verified top-level call: NO
child113 capture {145,3} consumed during verified top-level call: NO
```

## Interpretation correction

The child113 work from batches 36–49 remains valuable as structural reverse engineering of q's lazy descriptor/capture machinery, but it is **not currently reachable from the actual top-level execution path** that has been proven.

This substantially changes prioritization:

- runtime-behavior analysis should focus on the deterministic inert root path;
- child113/other child prototypes should be treated as dormant/unreached data unless a new reachable path is independently demonstrated.

## Artifact

`b54_child113_reachability.out`:

```text
INITIAL_LOOKUP113        47868
AFTER_ROOT_Q             number 47868
RAW_W4960_BEFORE         nil nil
W4960_VALUE              number 4244215898
RAW_W4960_AFTER          number 4244215898
AFTER_W4960_LOOKUP113    number 47868
LOOKUP113_UNCHANGED      true
```
