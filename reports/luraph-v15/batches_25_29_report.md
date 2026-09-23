# Batches 25–29 — cross-runtime q verification, entry normalization, and child 162 trace

Date: 2026-09-23

Scope remains offline. No Roblox/gameplay execution and no original VM factory execution.

## Batch 25 — q oracle re-run under a fresh Lua 5.4 host

A ctypes runner over system `liblua5.4.so.0` plus a bounded 32-bit `bit32` shim re-ran the existing `q_all_children.lua` harness. The 37 semantic outcomes match the prior harness: 33 normal D2 records parse to factory 2801/mode 2 descriptors and the four compact records 148/189/359/430 fail exactly at EOF offset 14. Differences are only script line numbers in the error strings.

Verdict: CROSS_RUNTIME_REPRODUCED_FOR_Q_DATA_PATH.

## Batch 26 — root return overlay resolved

For root PC4960, all four lazy overlay accesses succeed reproducibly:

- P[4960] = 1706231706
- C[4960] = 1846159626
- w[4960] = 4244215898
- t[4960] = 767502879

The opcode62 return established earlier is therefore `return R0, 4244215898` for the second value under the q proxy oracle. Snapshot/diff around each access shows zero lookup-table mutations and zero payload-buffer mutations. Thus this PC's overlay values are computed/read without triggering the F7/D2 lazy-materialization path.

R0 is still not recovered here.

## Batch 27 — entry-row census over all 33 normal D2 children

Raw entry-opcode distribution:

- opcode84: 13
- opcode128: 8
- opcode9: 6
- opcode91: 4
- opcode112: 2

All 4 overlay tables (P/C/w/t) were accessed at the entry PC for all 33 children: 132/132 accesses returned `nil` without errors. Initial entry dispatch therefore depends on the direct O/B/X/p arrays, not a materialized lazy operand.

## Batch 28 — normalize entry self-decoders

Every one of the 13 opcode84 entry rows self-decodes to opcode128. The exact arithmetic was re-run under Lua 5.4 for all 13 rows. All normalized opcode128 rows have O=0.

The four opcode91 entries normalize to opcode9:

- sid140 -> `(9,2,0,3)`
- sid188 -> `(9,1,0,2)`
- sid197 -> `(9,0,0,2)`
- sid283 -> `(9,0,0,2)`

The two opcode112 entries are explicit vararg materializers from the factory source:

- sid249, entry38: O=1/B=2 -> first vararg is placed at R2.
- sid357, entry53: O=2/B=37 -> arg1 is copied to R37 and arg2 to R38.

After normalization the 33 normal child entries collapse to exactly:

- 21 × opcode128 conditional gates
- 10 × opcode9 immediate initializers
- 2 × opcode112 vararg loaders

The 21 opcode128 gates all read `R0` (`O=0`) immediately. The factory allocates its register table with `table.create(capacity)` and does not itself populate R0 before entry. This is evidence that these descriptors either require external/pre-execution patching/context or are not meant to be invoked fresh in their q-returned state. This report does not label them dead code.

## Batch 29 — first concrete child path: symbol 162

Child 162: factory 2801, mode2, entry PC22, 23 rows, 8-register capacity. Its P/C/w/t overlays are nil for all PCs 1..23, so the path below needs only direct arrays.

Trace:

1. mode2 PC22/op9: `R2=1`.
2. mode2 PC23/op104: switch to mode171 and set W=18.
3. mode171 PC18/op10: `R7 = (R2 <= 0) = false`.
4. PC19/op28: false arm routes to PC21.
5. PC21/op28: false arm routes to PC8.
6. PC8/op1: `R7 = b[102]`; the top-level table maps b[102] to `coroutine.running`.
7. PC9/op115: `R7 = coroutine.running()`.
8. PC10/op133: accesses `F[1]` where factory upvalue F is the top-level `b.a` context table, and stores the current coroutine/thread into `F[1][7][F[1][6]]`.

Trace stops there because `b.a[1]` is outer runtime context not reconstructed by q alone. This establishes that child162 is a coroutine/context bookkeeping path, not merely random instruction data.

## Net progress

- q child parsing now reproduces in a fresh Lua host.
- root PC4960 second return value is resolved as 4244215898 under the lazy-overlay oracle, without mutating lookup/payload state.
- child entry bootstraps have a compact three-family normalized grammar.
- all opcode84 entry self-decoders converge to opcode128.
- child162 has a concrete cross-mode path to `coroutine.running()` and the shared `b.a` context registry.

Unresolved: R0 provenance for opcode128-gated children, semantics/provenance of the root return token, construction of `b.a[1]`, and complete child execution beyond the context-registry boundary.
