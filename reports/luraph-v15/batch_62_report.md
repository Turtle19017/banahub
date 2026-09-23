# Batch 62 — child378 is blocked at its normalized entry before error host-load

Date: 2026-09-23

Scope: bounded fresh-state trace of normal D2 child378.

Child378 has `entry=PC6`, `mode=2`, `regs=16`, `rows=133`. Batch59 identified a valid raw mode2 host-load candidate at `PC59`: `B=25,p=0`, which would load `b[25]=error` into `R0` if reached in mode2.

The entry row is the already verified opcode84 self-decoder:

```text
before: PC6 = (84, O=121, B=114, p=22332)
after : PC6 = (128,O=0,   B=7,   p=2)
```

The self-decoder re-executes the same PC. Mode2 opcode128 then evaluates:

```text
2 <= R0
```

Factory2801 creates a fresh empty register table and no instruction runs before PC6 that initializes R0. Therefore `R0=nil`, so a fresh q-returned descriptor cannot pass the first normalized gate.

Verdict: `PC59 -> b[25]=error` is not reachable in the fresh child378 state.
