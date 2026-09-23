# Batch 58 — exact VM primitives that can bridge into the host table

Date: 2026-09-23

Scope: static audit of `factory2801_full.lua`; no VM/gameplay execution.

## Goal

Batches 55–57 established that many plaintext Roblox/gameplay wrappers exist in the top-level host table `b`, while the verified root path never selects one. This batch identifies the exact factory2801 instructions capable of loading arbitrary host-table values into VM registers.

## Direct host-load primitives

### Mode 2

Only one direct indexed host load exists:

```lua
-- opcode 81
R[p[pc]] = b[B[pc]]
```

### Mode 171

Two indexed loads plus one whole-table load exist:

```lua
-- opcode 1
R[B[pc]] = b[O[pc]]

-- opcode 63
R[B[pc]] = b[w[pc]]

-- opcode 94
R[p[pc]] = b
```

### Mode 51

Two indexed loads plus one whole-table load exist:

```lua
-- opcode 37
R[B[pc]] = b[P[pc]]

-- opcode 147
R[p[pc]] = b[X[pc]]

-- opcode 148
R[p[pc]] = b
```

### Mode 81

One indexed load exists:

```lua
-- opcode 115
R[B[pc]] = b[p[pc]]
```

## What is not counted as a gameplay bridge

The factory also contains calls of the form:

```lua
b[J[J[2]]](b,J,nil,captures,nil)
```

inside nested-closure construction. Here `J` is a serialized child descriptor and `J[J[2]]` resolves its factory id (2801 for every normal prototype observed so far). These are **prototype-factory invocations**, not arbitrary loads of plaintext gameplay wrappers.

Likewise, writes such as `b[index]=value` mutate the host table but do not themselves load/call a wrapper.

## Consequence

A VM path cannot call one of the plaintext numeric wrapper closures unless it first obtains that value through one of the host-load primitives above (or loads the whole `b` table and indexes it later).

For descriptors beginning in mode2, the most direct bridge signature is therefore:

```text
reachable opcode81
  -> B[pc] identifies a host-table entry
  -> R[p[pc]] receives b[B[pc]]
  -> later reachable register-call opcode invokes that value
```

This gives a precise structural target for the next census instead of searching all call-like opcodes blindly.

Artifact: `b58_host_bridge_primitives.tsv`.
