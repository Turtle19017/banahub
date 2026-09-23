# Batch 57 — verified root never loads or calls the runtime wrapper library

Date: 2026-09-23

Scope: exact opcode-family audit of the already verified top-level root path.

## Goal

Batch55 established that `g_raw.lua` contains many runtime-touching wrapper bodies. Batch56 established that every parsed serialized prototype uses factory2801. This batch checks whether the **actually executed root instructions** ever bridge from factory2801 into those host wrappers.

## Semantic opcode set on the verified path

After accounting for the self-decoders and the second XOR layer, the root execution uses only these semantic opcode families:

```text
9    load immediate
34   broad instruction XOR/self-patch
62   return R[p], w[pc]
91   current-row self-decoder
99   paired register move + immediate
107  current-row self-decoder
110  paired register move
112  vararg prefix + packed tail
115  register move
118  pack register slice
124  direct jump
162  indirect jump through a VM register
```

For mode2, none of those semantics performs either of the two operations needed to reach the plaintext wrapper library:

1. load an arbitrary host-table entry such as `b[operand]` into a VM register;
2. call a function held in a VM register.

The path does use factory2801's fixed implementation helpers (`table.pack`, `table.move`, bit32 operations, buffer helpers, etc.) as VM machinery. Those are not the numeric gameplay wrapper closures inventoried in Batch55.

## Cross-check with earlier side-effect audit

This opcode audit independently matches Batch52:

- no gameplay closure call;
- no Roblox service/global access;
- no nested closure creation;
- no remote/network operation;
- no parent capture write;
- no dynamic host wrapper invocation.

Batch54 also showed that the only final lazy read `w[4960]` resolves a scalar and does not materialize child113.

## Reachability result

Combining batches 55–57:

```text
runtime/global-touching extracted wrappers in g_raw: 79
q-parsed normal prototypes:                         35
prototype factory IDs observed:                     {2801}
arbitrary host-wrapper loads on verified root:      0
register-function calls on verified root:           0
runtime-touching wrappers reached by proven root:   0
```

## Interpretation

For the top-level execution state actually reconstructed so far, the large plaintext Roblox/gameplay portion of `g_raw.lua` behaves as an **unreached host library**. It is loaded as data/functions when the table literal is constructed, but no verified root instruction selects or invokes it.

This does **not** prove those wrappers are permanently dead. A different descriptor state or an independently demonstrated reachable child path could still load them. The correct current label is:

```text
PRESENT IN HOST LIBRARY
NOT REACHED BY VERIFIED TOP-LEVEL PATH
```

Artifacts:

- `b57_root_host_access.tsv`
- `b57_reachability_summary.json`
