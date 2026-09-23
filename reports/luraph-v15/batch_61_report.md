# Batch 61 — child357 is blocked by capture F[5] before xpcall host-load

Date: 2026-09-23

Scope: bounded fresh-state trace of normal D2 child357. No gameplay/Roblox execution.

Child357 has `entry=PC53`, `mode=2`, `regs=44`, `rows=61`. Batch59 identified a structurally valid raw host-load at `PC32`: mode2 opcode81 with `B=36,p=0`, i.e. `R0=b[36]`; the recovered host table maps `b[36]` to `xpcall`.

From the actual entry, the direct path is:

```text
PC53/op112 O=2 B=37
  R37 = arg1
  R38 = table.pack(args2..)

PC54/op115 O=1 B=37
  R1 = R37

PC55/op152 B=1 p=1
  R1 = not R1

PC56/op75 O=1 B=5 p=1
  J = F[5]
  R1 = J[7][J[6]][R1]
```

Thus PC56 requires capture cell `F[5]`. A direct/fresh invocation of the q-returned descriptor has no parent-created capture vector, so it cannot continue at PC56. No backward jump to PC32 occurs before this barrier.

Verdict: `PC32 -> b[36]=xpcall` is not reachable in the fresh q-returned child357 state. It could only become relevant if a separately proven parent closure-construction path supplies the required capture ABI.
