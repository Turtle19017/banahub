# Batch 60 — child140 never reaches its structurally plausible host-load PC45

Date: 2026-09-23

Scope: bounded fresh-state trace of normal D2 child symbol140. No Roblox/gameplay execution.

## Why child140

Batch59 found only seven raw mode2 opcode81 rows whose destination register lies inside the descriptor's register capacity. Child140 has the smallest/cleanest candidate near the end of a 45-row prototype:

```text
child140
entry = PC1
mode = 2
regs = 10
rows = 45

PC45 raw:
opcode81 O=0 B=0 p=0
```

If PC45 were reached in mode2 without further transformation, it would perform:

```lua
R0 = b[0]
```

The question is whether the fresh descriptor can actually reach PC45.

## Deterministic mode2 bootstrap

Exact self-decoder arithmetic gives:

```text
PC1  91 -> (9,   O=2,  B=0, p=3)   => R3=2
PC2  91 -> (9,   O=15, B=0, p=6)   => R6=15
PC3  91 -> (124, O=0,  B=0, p=11)
PC12 84 -> (162, O=0,  B=6, p=0)
PC10 84 -> (154, O=23, B=12,p=5)
```

The resulting reachable sequence is:

```text
PC1 -> PC2 -> PC3
    -> PC12 -> PC16 -> PC17 -> PC18
    -> PC12 -> PC22 -> PC23 -> PC10
    -> PC24 -> PC25 -> PC26
    -> PC12 -> PC16 -> PC17 -> PC18
    -> PC12 -> PC22 -> PC23
    -> PC27 -> PC28
```

The state changes that control the loop are:

```text
first phase:  R3=2, R6=15/21, R5=false
second phase: R3=1, R6=15/21, R5=true
```

## PC27 self-unmask layer

PC27 is:

```text
opcode34 O=28 B=16 p=56
```

It XOR/self-patches PCs 29..44 with:

```text
key(s) = (56 XOR s) & 127
```

and changes its own opcode to 92.

Important post-XOR rows include:

```text
PC29 -> (23,  O=0,         B=7, p=0)
PC30 -> (159, O=113,       B=9, p=39)
...
PC45 -> unchanged (81, O=0,B=0,p=0)
```

PC45 is outside the PC29..44 XOR range.

## Mode switch at PC28

Mode2 opcode104 uses:

```lua
mode = O[pc]
W = p[pc] + 1
break
```

For PC28:

```text
O=171
p=28
```

so execution continues at:

```text
mode171 / PC29
```

Mode171 PC29/opcode23 performs:

```text
R7 = descriptor Z
```

and falls through to PC30.

## PC30 is a real closure-construction opcode shape, but its descriptor operand is invalid

After the PC27 XOR, PC30 is mode171 opcode159. Its handler expects:

```lua
child_descriptor = w[30]
capture_spec     = P[30]
child_factory    = b[child_descriptor[child_descriptor[2]]]
```

A direct lazy-overlay oracle on fresh child140 returns:

```text
w[30] = 2280377406   -- number
```

`P[30]` exceeds the current compatibility shim during decode, but this does not affect the verdict: opcode159 already requires `w[30]` to be a descriptor table, and it is definitively numeric in the q-returned state.

Therefore fresh execution stops at PC30 before it can instantiate a child closure.

## Consequence for PC45

```text
PC45 mode2 host-load candidate reachable in fresh child140: NO
```

The candidate `R0=b[0]` at PC45 is structural/dormant in the q-returned descriptor state.

This is another example of why a raw host-load opcode is not enough to establish a bridge to the host library: reachability, mode, self-decode state, and operand types must all agree.

## Verdict after batches 58–60

So far there is still **no proven reachable bridge from factory2801 into any of the 79 runtime-touching plaintext wrappers**:

- verified root path: no host load at all;
- normal D2 raw mode2 scan: no valid direct host load targeting a known runtime wrapper;
- child140's plausible host load: blocked earlier at a malformed/dormant closure-construction state.

Artifacts:

- `b60_child140_rows.out`
- `b60_child140_postxor.tsv`
- `b60_child140_pc30_overlays.out`
- `b60_child140_trace.tsv`
