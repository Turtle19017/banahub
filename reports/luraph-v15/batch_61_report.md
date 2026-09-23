# Batch 61 — child357 host-load candidate depends on parent capture before reachability

Date: 2026-09-23

Scope: bounded fresh-descriptor trace of normal D2 child357. No Roblox/gameplay execution.

## Candidate from Batch59

Child357 metadata:

```text
entry = PC53
mode  = 2
regs  = 44
rows  = 61
```

Its structurally register-valid raw host-load candidate is:

```text
PC32: opcode81 O=99 B=36 p=0
```

If reached in mode2 without further transformation:

```lua
R0 = b[36]
```

and the recovered host table maps `b[36]` to `xpcall`.

## Actual entry path

PC53 is opcode112 with `O=2, B=37`. Using the corrected opcode112 semantics:

```text
R37 = arg1
R38 = table.pack(select(2,...))
```

Execution falls through to PC54.

PC54 is mode2 opcode115:

```lua
R[O] = R[B]
```

so:

```text
R1 = R37 = arg1
```

PC55 is mode2 opcode152:

```lua
R[B] = not R[p]
```

with `B=1,p=1`, therefore:

```text
R1 = not R1
```

PC56 is mode2 opcode75:

```lua
cell = F[B]
R[O] = cell[7][cell[6]][R[p]]
```

For child357:

```text
B=5
O=1
p=1
```

thus it requires:

```text
F[5]
```

to be a valid parent capture/upvalue cell before execution can continue.

## Consequence

No host-table load occurs before this capture dependency. In particular, the raw PC32 `xpcall` load is not reachable from the q-returned descriptor alone.

The correct classification is:

```text
child357 / PC32 xpcall bridge:
NOT PROVEN REACHABLE
BLOCKED BY PARENT CAPTURE F[5] PROVENANCE
```

This is weaker than the hard `R0=nil` entry failures seen in gate128 children: a correctly constructed parent capture vector could allow child357 to continue. The parent closure site/capture spec for child357 has not yet been established, so the trace stops at PC56 rather than inventing F[5].

Artifacts:
- `b61_child357_rows.out`
- `b61_child357_trace.tsv`
