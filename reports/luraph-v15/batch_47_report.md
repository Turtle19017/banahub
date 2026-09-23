# Batch 47 — PC4966 is skipped by the verified bootstrap

Date: 2026-09-23

Scope: bounded control-flow audit using only previously verified bootstrap transitions.

## First trampoline pass

After the first global XOR layer, the verified root rows include:

```text
PC21 = opcode99
PC22 = opcode9, O=4967, p=128
PC23 = self-decoder -> opcode124 direct jump to 4977
PC4978 = opcode162, B=128
```

PC21/22 sets:

```text
R128 = 4967
```

Then:

```text
PC23/op124: W = 4977
loop increment -> PC4978
PC4978/op162: W = R128 = 4967
loop increment -> PC4968
```

Therefore this verified pass enters at PC4968 and skips both PC4966 and PC4967.

## Second trampoline pass

Later, after the second broad XOR at PC14:

```text
PC15 self-decodes -> opcode9 O=4959 p=128
R128 = 4959
PC16 self-decodes -> opcode124 jump to 4977
PC4978/op162 -> W=4959
loop increment -> PC4960
```

This pass enters at PC4960 and again does not execute PC4966.

PC4960 is the already verified opcode62 return boundary for this invocation.

## Consequence for the PC4966 third-XOR candidate

Batch46 identified PC4966 as a structurally valid opcode34 broad-XOR row after the two known global layers. This batch establishes that the entire verified bootstrap path through its return does **not** execute it.

Therefore the PC4966 transform must remain hypothetical/dormant. It must not be applied to PC1394 or the child113 alias set when describing the verified bootstrap state.

## Verdict

```text
PC4966 reachable in verified bootstrap: NO
PC4966 executed before PC4960 return: NO
third broad XOR verified: NO
```

Any future use of the PC4966 layer requires a new, independently established control-flow path (for example a later invocation/state), not extrapolation from the first bootstrap.
