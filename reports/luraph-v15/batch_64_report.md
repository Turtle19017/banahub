# Batch 64 — mode-aware correction of child404

Date: 2026-09-23

Scope: static factory2801 source audit + previously recovered child404 rows. No gameplay/Roblox execution.

## Correction trigger

Earlier Batch35 accidentally applied the **mode2** semantics of opcode81 to child404 after it had already switched into **mode81**. Factory2801 has four distinct dispatch loops, so the same numeric opcode cannot be interpreted across modes.

Mode boundaries in the extracted factory2801 source are approximately:

```text
mode2   starts at inner-source offset   539
mode171 starts at                    14431
mode51  starts at                    27159
mode81  starts at                    43446
```

The host-load statement `R[p]=b[B]` lies inside mode2. Mode81's direct host load is opcode115 (`R[B]=b[p]`).

## Correct mode81 opcode81

In the mode81 loop, opcode81 executes the open-upvalue cleanup path and then returns with no explicit values. It does **not** read `b[B]`.

Thus child404's already recovered post-XOR row:

```text
PC9 = (opcode81, O=0, B=0, p=0)
```

must be interpreted as a mode81 return, not `R0=b[0]`.

## Corrected child404 path

The previously verified path up to the mode switch remains unchanged:

```text
mode2 PC20..24 bootstrap
 -> PC3/4 gate
 -> PC5 capture-cell clear (requires valid F[1])
 -> PC6 R2=1
 -> trampoline
 -> PC3/4 second pass
 -> PC13 switch mode2 -> 171
 -> mode171 PC12
 -> mode171 PC14 switch 171 -> 81, W=9
 -> mode81 PC9/op81
 -> RETURN
```

Important invocation qualification: a standalone q-returned child404 still lacks the parent-provided capture cell `F[1]` required earlier at PC5. But **conditional on a valid parent capture vector**, the path reaches mode81 PC9 and returns cleanly there.

## Invalidated older claims

The following Batch35 statements are retracted:

```text
mode81 PC9: R0=b[0]                 -- wrong mode semantics
mode81 PC10: bit32.bnot path reached -- not reached
mode81 PC11: fail because t[11]=nil  -- not reached
```

The `t[11]=nil` observation may still be structurally true as a lazy-overlay probe, but it is not part of child404's corrected reachable path.

## Verdict

```text
child404 mode81 entry opcode: 81
mode81 opcode81 meaning: cleanup + return
host b[0] load on corrected child404 path: NO
PC10/PC11 reached: NO
```
