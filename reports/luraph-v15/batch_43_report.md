# Batch 43 — first verified capture-spec companion for child113

Date: 2026-09-23

Scope: offline q/root-overlay analysis.

## Finding

A bounded root.w scan found another descriptor-valued reference at PC1394.

Fresh-root identity test:

```text
rawequal(root.w[333], root.w[1394]) == true
```

So PC1394 references the exact same cached child113 descriptor object established in earlier batches.

Unlike the earlier aliases, its companion C overlay is a valid capture-spec table:

```text
C[1394] = {145, 3}
length  = 2
```

## Capture ABI interpretation

The previously recovered parent closure-creation protocol interprets capture specs as pairs:

```text
{ slot, kind, slot, kind, ... }
```

with kind 3 meaning a shared/open parent-register cell:

```lua
cell = cache[slot]
if not cell then
    cell = {[6]=slot,[7]=parent_registers}
    cache[slot] = cell
end
child_capture[i] = cell
```

Therefore `{145,3}` encodes one capture:

```text
child F[1]
    -> shared/open cell
    -> parent register slot R145
```

This is the first direct data-layer evidence connecting child113 to a specific parent capture slot.

## Important limitation

After the two globally verified root XOR layers, PC1394's static row is:

```text
opcode=34
O=99
B=32
p=34
```

That does not by itself equal the already identified mode2 closure opcode158 or mode171 opcode159. The row may still be subject to a later reachable self-decode/mode-specific interpretation, or the descriptor/capture operands may be data reused by another state.

Therefore this batch establishes:

```text
child113 descriptor + valid capture spec at PC1394
```

but does **not yet** claim PC1394 is a reachable closure-instantiation instruction.

## Consequence

Child113's invocation ABI is now partially constrained:

```text
F[1] = parent shared/open upvalue cell for R145
```

If a reachable closure-creation path using the PC1394 operands is found, the parent-to-child capture provenance will be closed end-to-end.

Artifact: `b43_pc1394_identity_capture.out`.
