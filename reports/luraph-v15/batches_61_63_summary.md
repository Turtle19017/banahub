# Batches 61–63 summary

The remaining structurally plausible direct mode2 host-load candidates from Batch59 were traced/audited.

- child357: entry reaches PC56/op75, which requires parent capture `F[5]`; raw PC32 `b[36]=xpcall` is not reached.
- child378: opcode84 normalizes entry to opcode128/O0 and immediately requires `2 <= R0(nil)`; raw PC59 `b[25]=error` is not reached.
- child133/394/435: all begin (directly or after opcode84 self-decode) with opcode128/O0 and fail the same fresh-R0 gate before their valid raw host-load candidates.

Combined with Batch60 child140, all **7/7** raw mode2 opcode81 rows with in-range destinations are now shown **not reachable in fresh q-returned state**.

Current bounded bridge verdict: no direct fresh-mode2 path among the 33 normal D2 descriptors has been proven to load a host-library function. A real bridge, if present, must depend on descriptor mutation, parent capture/context, or a later mode/state.
