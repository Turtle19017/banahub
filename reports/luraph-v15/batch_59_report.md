# Batch 59 — raw mode2 host-load census across normal D2 children

Date: 2026-09-23

Scope: structural census of the 33 normal D2 child descriptors accepted by q. No child is assumed reachable merely because a raw opcode value is present.

## Why opcode81 is the first useful filter

All 33 normal D2 descriptors start in mode2. Batch58 established that mode2's direct indexed host load is:

```lua
-- opcode81
R[p[pc]] = b[B[pc]]
```

So every raw `X[pc]==81` row was collected together with its host index `B[pc]`, destination register `p[pc]`, and descriptor register capacity.

## Census result

Across the 33 normal D2 children:

```text
71 raw opcode81 rows
```

Sixteen of those 71 rows have `B[pc]` equal to one of the runtime/global-touching numeric wrapper IDs identified in Batch55. The observed runtime-wrapper host IDs are:

```text
52, 78, 87, 88, 98, 100, 101, 110, 121
```

Examples include wrappers whose source touches Roblox/game/runtime state.

However, every one of those 16 rows has:

```text
p[pc] >= register_capacity
```

so if interpreted literally as mode2 opcode81 in the q-returned descriptor, it would write outside the descriptor's valid register domain.

Result:

```text
raw opcode81 rows targeting known runtime wrappers: 16
with destination inside register capacity:             0
```

## Structurally plausible raw opcode81 rows

Only **7/71** raw opcode81 rows have `p[pc] < register_capacity`:

| sid | PC | host B | dest p | reg capacity |
|---:|---:|---:|---:|---:|
| 133 | 168 | 20 | 20 | 38 |
| 140 | 45 | 0 | 0 | 10 |
| 357 | 32 | 36 | 0 | 44 |
| 378 | 59 | 25 | 0 | 16 |
| 394 | 270 | 91 | 105 | 133 |
| 435 | 223 | 228 | 0 | 45 |
| 435 | 225 | 228 | 0 | 45 |

None of these host indices belongs to the 79 runtime-touching wrapper set extracted in Batch55.

Several are recognizable host primitives rather than gameplay wrappers in `g_raw.lua`, for example:

```text
b[20] = coroutine.status
b[25] = error
b[36] = xpcall
b[91] = select
```

`b[0]` is function-valued in the recovered host table; host id 228 is left unclassified here.

## Interpretation

This census gives strong negative evidence against a **direct, already-decoded mode2 load of a known gameplay wrapper** in the q-returned child descriptors:

```text
known runtime-wrapper target + valid register destination = 0 rows
```

But it does not prove such a bridge can never appear. Many rows are self-decoded or bulk-XOR transformed before execution, and descriptors can switch modes. Therefore raw opcode values outside a proven reachable path remain structural candidates only.

Batch60 traces one of the seven structurally plausible rows (`child140 / PC45`) from its actual entry to test whether it is reachable in the fresh descriptor state.

Artifacts:

- `b59_scan_mode2_hostloads.out`
- `b59_mode2_hostload_candidates.tsv`
