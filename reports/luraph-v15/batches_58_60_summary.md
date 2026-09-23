# Batches 58–60 summary

This three-batch pass narrows the VM→host-library bridge substantially.

## Exact host-load primitives

Factory2801 can directly load host-table entries only through:

```text
mode2:   opcode81
mode171: opcode1, opcode63; opcode94 loads whole b
mode51:  opcode37, opcode147; opcode148 loads whole b
mode81:  opcode115
```

## Normal D2 mode2 census

Across 33 normal D2 children:

```text
71 raw opcode81 rows
16 point at known runtime-touching wrapper IDs
0/16 have a destination register inside descriptor capacity
```

Only seven raw opcode81 rows are structurally register-valid; their host IDs are `20,0,36,25,91,228`, not members of the 79 runtime-touching wrapper set. Known examples are primitives such as `coroutine.status`, `error`, `xpcall`, and `select`.

## Child140 reachability test

Child140 contains a structurally valid PC45 mode2 host load (`R0=b[0]`). A full bounded fresh-state trace reaches a mode171 opcode159 closure-construction row at PC30 first. Its required `w[30]` operand resolves to numeric `2280377406`, not a child descriptor table, so execution cannot proceed to PC45 in the q-returned state.

## Current bounded verdict

No path proven so far bridges factory2801 to a runtime-touching plaintext wrapper:

```text
verified root path:                       no host load
raw valid mode2 D2 loads to runtime IDs: 0
traced child140 candidate:                unreachable before invalid closure operand
```

This does not prove the gameplay wrapper library is permanently dead; later self-decodes, descriptor mutation, or another reachable child could still create a valid host load. But the bridge remains unproven after direct source-level and descriptor-level census.
