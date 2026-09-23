# Batch 42 — verified cached alias set for child113

Date: 2026-09-23

Scope: offline q root-overlay analysis.

## Goal

Determine whether the growing set of root.w indices that produce a 2801/entry773/788-row descriptor are merely similar descriptors or references to the exact same cached child object.

## Bounded w scan

Small fresh-process scans were used to avoid the memory growth seen when scanning hundreds of lazy overlay indices in one process.

Additional descriptor-valued indices found below PC1000:

```text
333
356
545
570
577
744
776
782
786
872
```

The previously discovered higher references `1120` and `1250` were included in the identity test.

## Object identity test

For every candidate, a fresh root descriptor was created, `w[333]` was materialized first, then exactly one candidate was accessed and compared with Lua `rawequal`.

All tests returned true:

```text
w[333] == w[356]
w[333] == w[545]
w[333] == w[570]
w[333] == w[577]
w[333] == w[744]
w[333] == w[776]
w[333] == w[782]
w[333] == w[786]
w[333] == w[872]
w[333] == w[1120]
w[333] == w[1250]
```

Every comparison was performed on a fresh root to avoid later overlay accesses contaminating the compatibility state.

Including the anchor PC333, this establishes a verified alias set of at least **12 root.w indices** pointing to the exact same cached child113 descriptor object.

## Companion observations

Some aliases have successfully decoded non-table companions:

- `C[333] = 2450515686`
- `C[570] = 2450515686`
- `C[744] = 2450515686`
- `C[776] = 2450515686`
- `C[782] = 2450515686`
- `P[1120] = 1069115016`
- `C[1120] = false`
- `t[1120] = false`

None of these decoded companions is a capture-spec table.

## Conclusion

Symbol113 is not materialized for a single root instruction. The q overlay exposes one cached descriptor through many root.w aliases.

This supports a two-layer model:

```text
serialized symbol113
      |
      v
one cached child descriptor object
      |
      +-- root.w[333]
      +-- root.w[356]
      +-- root.w[545]
      +-- ...
      +-- root.w[1250]
```

Therefore locating the true parent closure-instantiation site requires more than finding descriptor-valued `w[pc]`; the companion capture operand and reachable VM mode must also agree.

Artifacts:
- `b42_scan_1_1000.out`
- `b42_alias_companions.out`
- `b42_alias_identity_fresh.out`
