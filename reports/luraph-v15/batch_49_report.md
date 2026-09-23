# Batch 49 — opcode128 entry descriptors require pre-invocation mutation

Date: 2026-09-23

Scope: factory2801 invocation semantics, grounded in the recovered factory source and parsed child descriptors.

## Factory call order

The factory captures descriptor metadata first, including:

```text
entry PC
initial mode
register capacity
O/B/X/p arrays
P/C/w/t overlays
capture vector F
```

But the returned closure begins each invocation with:

```lua
local W = entry_pc
local R = table.create(register_capacity)
local mode = initial_mode
-- dispatch starts immediately
```

No positional argument, vararg, capture cell, or environment value is copied into `R0` before the first instruction executes. The capture vector `F` is a separate structure and does not initialize the register table.

## Opcode128 entry semantics

In mode2, opcode128 is:

```lua
if p[pc] <= R[O[pc]] then
    pc = B[pc]
end
```

For the normalized gate children, `O=0`. Therefore the very first operation is:

```text
positive_integer <= R0
```

with `R0=nil` in a fresh invocation.

## Affected descriptors

Batch34 established 21 normal D2 children whose normalized first instruction is opcode128/O=0. Batch37 added tag09 symbol113:

```text
symbol113 entry PC773:
opcode128 O=0 B=27 p=1
```

Total currently proven set:

```text
22 descriptors
= 21 D2 children
+ 1 tag09 child113
```

The full set is in `b49_fresh_call_impossible_set.tsv`.

## Stronger conclusion

The earlier wording "these children need context" was too broad. Ordinary invocation context alone cannot fix this: the VM allocates the empty register table *inside the call* and dispatches before any generic argument-loading opcode can run.

Therefore, if one of these descriptors is genuinely executed, at least one of the following must occur **before the closure call**:

1. its entry instruction / direct arrays are mutated so opcode128 is no longer first;
2. its entry PC is changed;
3. some other descriptor-level transformation changes the bootstrap state;
4. the descriptor is never invoked in this q-returned state.

Captures such as child113's `F[1] -> parent R145` do not by themselves initialize `R0`.

## Verdict

```text
fresh call of unmodified opcode128/O0 descriptor: structurally impossible
arguments alone can repair R0 before first dispatch: no
capture vector alone can repair R0 before first dispatch: no
pre-invocation descriptor mutation required if reachable: yes
```

This shifts the next reverse-engineering target from register initialization to **who mutates the child descriptor before factory invocation**.
