# Batches 61–63 summary

This pass closes the seven **register-valid raw mode2 opcode81 host-load rows** identified in Batch59.

- **child357**: candidate `PC32: R0=b[36]=xpcall` is not reachable from descriptor state alone. Entry reaches PC56 first, which requires parent capture cell `F[5]`; provenance is unresolved.
- **child378**: entry opcode84 self-decodes exactly to `opcode128 O=0 B=7 p=2`; fresh `R0=nil` blocks before `PC59: R0=b[25]=error`.
- **child133**: direct entry `opcode128 O=0 B=32 p=1`; blocked immediately.
- **child394**: direct entry `opcode128 O=0 B=25 p=2`; blocked immediately.
- **child435**: opcode84 entry self-decodes to `opcode128 O=0 B=36 p=7`; both raw `b[228]` candidates are blocked immediately.
- **child140** was already traced in Batch60 and is blocked before its PC45 host load by an invalid mode171 closure-descriptor operand.

Complete bounded result:

```text
7 raw register-valid mode2 host-load rows
0 proven reachable
5 hard-blocked by fresh R0=nil entry gate
1 blocked earlier by invalid closure descriptor operand
1 depends on unresolved parent capture F[5]
```

So the parsed child corpus still contains **no proven reachable direct mode2 bridge into the host wrapper library**.
