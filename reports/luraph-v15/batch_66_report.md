# Batch 66 — VM→host bridge mechanism is proven in child162, but only to a core primitive

Date: 2026-09-23

Scope: mode-aware reconciliation of the already traced child162 path with the exact factory2801 source.

## Why this matters

Recent reports used "no bridge" as shorthand for "no bridge to the runtime/gameplay wrapper set." The VM→host mechanism itself is actually already demonstrated end-to-end by child162.

## Exact source semantics in mode171

Factory2801 mode171 contains:

```lua
-- opcode1
R[B[pc]] = b[O[pc]]

-- opcode115
R[p[pc]] = R[O[pc]]()
```

These are in the mode171 dispatch region, not borrowed from another mode.

## Child162 verified sequence

The prior bounded trace reaches mode171 deterministically:

```text
PC22/mode2 op9   -> R2=1
PC23/mode2 op104 -> switch mode171, W=18
PC18/op10        -> R7=false
PC19/op28        -> PC21
PC21/op28        -> PC8
PC8/op1          -> R7 = b[102]
PC9/op115        -> R7 = R7()
```

The recovered top-level host table defines:

```text
b[102] = coroutine.running
```

Therefore:

```text
mode171 PC8  loads coroutine.running from host table
mode171 PC9  calls it through a VM register
```

This occurs before PC10 touches capture `F[1]`. So even a trace that later stops for missing parent capture has already proven the load+call bridge itself.

## Refined architecture

What is proven:

```text
factory2801 VM
  -> indexed host-table load
  -> register-held function
  -> register call
  -> native/core host primitive executes
```

What is not yet proven:

```text
factory2801 VM
  -> load one of the plaintext Roblox/gameplay wrappers
  -> call that wrapper on a reachable path
```

Thus future reports should distinguish:

- **HOST BRIDGE MECHANISM: VERIFIED** (child162 -> coroutine.running)
- **GAMEPLAY WRAPPER BRIDGE: UNPROVEN**

This correction makes the search target narrower: we no longer need to prove that factory2801 *can* call host entries; we need only find a reachable load/call pair whose host index belongs to the gameplay wrapper set.
