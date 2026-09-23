# Batch 62 — child378 host-load candidate is blocked at its normalized entry gate

Date: 2026-09-23

Scope: exact entry self-decoder arithmetic plus bounded fresh-descriptor reachability.

## Candidate from Batch59

Child378 metadata:

```text
entry = PC6
mode  = 2
regs  = 16
rows  = 133
```

Its structurally valid raw mode2 host-load candidate is:

```text
PC59: opcode81 O=99 B=25 p=0
```

If reached in mode2 without further transformation:

```lua
R0 = b[25]
```

and `b[25]` is the host primitive `error`.

## Exact entry normalization

PC6 raw row:

```text
opcode84 O=121 B=114 p=22332
```

Running the exact opcode84 uint32 arithmetic gives:

```text
opcode84
  -> opcode128 O=0 B=7 p=2
```

This independently matches the earlier all-child normalization census.

## Fresh invocation semantics

Mode2 opcode128 is:

```lua
if p[pc] <= R[O[pc]] then
    pc = B[pc]
end
```

So child378 begins with:

```text
2 <= R0
```

Factory2801 creates a fresh empty register table and does not populate R0 before dispatch. Therefore:

```text
R0 = nil
```

and the first normalized instruction cannot execute as a fresh q-returned descriptor.

## Consequence

The candidate host load at PC59 is unreachable in the fresh descriptor state:

```text
child378 / PC59 error bridge:
FRESH-STATE UNREACHABLE
blocked at normalized entry opcode128/O0
```

As with the other opcode128/O0 descriptors, a pre-invocation descriptor mutation could in principle change the entry state; no such mutation is currently proven for child378.

Artifacts:
- `b62_check_op84_all.lua`
- `b62_child378_rows.out`
- `b62_child378_gate.tsv`
