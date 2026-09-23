# Batch 68 — corrected mode-aware direct host-load census

Date: 2026-09-23

Scope: structural scan of the 33 normal D2 base arrays recovered in Batch67. Raw opcode presence is not treated as reachability.

## Opcode selector correction

Factory2801 does not use the same instruction array as the opcode selector in every mode:

```text
mode2   : H = X[pc]
mode171 : H = X[pc]
mode51  : H = O[pc]
mode81  : H = X[pc]
```

Therefore direct host-load scans must use:

```text
mode2   opcode81  : R[p] = b[B]
mode171 opcode1   : R[B] = b[O]
mode51  opcode147 : R[p] = b[X]
mode81  opcode115 : R[B] = b[p]
```

The earlier mode51 shorthand that searched `X==147` is corrected here; a mode51 opcode147 row is identified by `O==147`.

## Raw structural census

Using the runtime/gameplay wrapper ID set from Batch55 plus corrected host id 0:

| Mode/op | raw rows | runtime target | dest in range | runtime + dest in range |
|---|---:|---:|---:|---:|
| mode2 / 81 | 71 | 17 | 7 | 1 |
| mode171 / 1 | 100 | 18 | 64 | 14 |
| mode51 / 147 | 17 | 0 | 8 | 0 |
| mode81 / 115 | 81 | 39 | 45 | 35 |

The large mode81 count is dominated by raw rows with `p=0`, i.e. host id 0; it is only a structural candidate set because those rows are not necessarily executed in mode81.

Mode171's 14 runtime+valid rows occur only in children 133, 240, 394 and 446. Examples include host ids 12, 95 and 110.

No raw normal-D2 mode51 opcode147 row directly targets the current known gameplay/runtime wrapper set before local self-unmasking.

Conclusion: mode-aware structural scanning produces many potential bridge rows, but the candidate set is still far larger than the reachable set. Local XOR/self-decode and actual mode control remain mandatory before promotion.

Artifacts:
- `b68_mode_aware_direct_hostloads.tsv`
- `b68_mode_aware_direct_hostload_stats.json`
