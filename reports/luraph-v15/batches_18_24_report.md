# Batches 18–24 — child D2 parser oracle and lazy instruction overlays

Date: 2026-09-23

Scope: offline reverse engineering of the already-decompressed Luraph v15 sample. No Roblox/gameplay execution and no original VM factory execution. The q harness stops at data/prototype reconstruction.

## Starting point

Previous work had recovered the E envelope, root prototype, factory 2801 and root bootstrap, classified F7/D2 records, recovered 190/213 F7 strings, and structurally validated 37 lazy D2 records. The remaining important uncertainty was whether inferred child-D2 keys actually produce prototypes accepted by Luraph's own q parser and how q wires instruction arrays/lazy fields.

## Batch 18 — randomized descriptor field IDs

The verified root plaintext starts with:

```
02 16 15 19 1F 1E 00 00 00 00 1C ...
```

Tracing q shows the five bytes after type `02` are not arbitrary payload. They are randomized descriptor-field IDs. For root:

- byte1 `0x16` (22) becomes the backing key for fixed descriptor slot 9;
- byte2 `0x15` (21) -> fixed slot 16;
- byte3 `0x19` (25) -> fixed slot 5;
- byte4 `0x1F` (31) -> fixed slot 8;
- byte5 `0x1E` (30) -> fixed slot 15.

The following four zero bytes are read as a u32 value. The next ID `0x1C` (28) is the backing key for fixed slot 6; in the fully parsed root, descriptor[28] = entry PC 4994.

This establishes that D2 records randomize physical table keys while q exposes stable semantic slots through indirection.

## Batch 19 — minimal q parser oracle

A standalone Lua 5.3 harness was built from q and only its data-decoder helpers. Buffer/bit32/table primitives were emulated and all numeric factories/gameplay paths remained absent.

The harness reproduced the previously verified root descriptor exactly:

- factory = 2801
- entry PC = 4994
- mode = 2
- register capacity = 163
- O/B/X/p arrays = 5018 rows each

Root plaintext SHA-256: `d65760495a18cbcf6ea82ca2c861952555c351b2e356cab95054cda371cb0c3c`.

Because these values independently match the prior Luau q harness, the reduced parser is accepted as an oracle for this sample's D2 data path.

## Batch 20 — two small child prototypes parse end-to-end

Using fresh copies of the 92,516-byte payload, each lazy D2 pointer was resolved, its inferred constant XOR byte was removed, then the actual q parser was invoked.

### Symbol 252

- factory 2801
- entry PC 1
- mode 2
- register capacity 7
- 3 instruction rows
- O = [121, 6, 0]
- B = [115, 0, 0]
- X = [84, 148, 31]
- p = [22335, 1, 0]

### Symbol 426

- factory 2801
- entry PC 1
- mode 2
- register capacity 12
- 8 instruction rows
- O = [121,120,11,83,0,7,171,117]
- B = [126,117,8,9,7,1,0,117]
- X = [84,84,21,151,52,34,104,8]
- p = [22335,20665,1,0,0,116,7,114]

This is the first end-to-end confirmation that the inferred lazy D2 key produces a prototype accepted by the sample's own q parser.

## Batch 21 — all 37 lazy D2 records tested with q

Results:

- 33/37 normal D2 records parse successfully.
- Every successful record returns factory 2801 and mode 2.
- Every successful record has equal O/B/X/p row counts.
- The 4 known 14-byte records (148, 189, 359, 430) fail exactly at EOF offset 14 when forced through the normal prototype parser.

Thus the 33 normal children are parser-validated, while the four 14-byte records are independently confirmed to be a different compact subtype rather than bad keys.

Largest decoded child in this set is symbol 191 with 4032 instruction rows; the smallest is symbol 252 with 3 rows.

## Batch 22 — stable 16-slot descriptor schema

Across all 33 normal child prototypes, q exposes the following stable semantic schema despite randomized backing IDs:

| Fixed slot | Meaning / invariant |
|---:|---|
| 1 | constant 2 |
| 2 | factory id = 2801 |
| 3 | constant 1 |
| 4 | table with 5 entries |
| 5 | empty table |
| 6 | entry PC |
| 7 | initial dispatch mode = 2 |
| 8 | O instruction-field array, N rows |
| 9 | B instruction-field array, N rows |
| 10 | X/opcode instruction-field array, N rows |
| 11 | auxiliary/lazy P table |
| 12 | auxiliary/lazy C table |
| 13 | p instruction-field array, N rows |
| 14 | auxiliary/lazy w table |
| 15 | register capacity |
| 16 | auxiliary/lazy t table |

For each normal child, slots 8/9/10/13 always have the same N.

## Batch 23 — P/C/w/t are lazy overlays, not separate constant pools

Object-identity checks across all 33 normal child descriptors give 33/33 agreement:

- slot11 P[0] is the same table object as direct p;
- slot12 C[0] aliases direct B;
- slot14 w[0] aliases direct O;
- slot16 t[0] aliases direct X;
- all four overlay tables have metatables.

The original decoded source defines `h1="__index"`; q installs the same callback function as the `__index` metamethod for all four overlays belonging to one prototype.

Therefore the architecture is:

```
direct arrays: O / B / X / p
       ^         ^   ^   ^
       |         |   |   |
lazy overlay: w / C / t / P
```

with `[0]` retaining the underlying direct array and `__index` decoding/materializing non-present fields on demand.

This substantially changes the interpretation of `w[pc]` seen at the root return boundary: `w` is a lazy overlay around O, not a standalone constant pool.

## Batch 24 — bounded lazy-proxy execution and child self-decoder

The q `__index` callback and its transitive decoder-helper closure were isolated. Under the current Lua emulation it can already return concrete values for some child fields, but other accesses fail at buffer/string operations because the compatibility layer is still incomplete. These partial outputs are retained as probe evidence only and are NOT promoted to recovered constants.

Independently, the arithmetic handler for entry opcode 84 was evaluated for both small children with the exact 32-bit W1/A1 helpers from the factory and cross-checked under Lua:

```
child 252 PC1:
  before X=84 O=121 B=115 p=22335
  after  X=128 O=0   B=6   p=1

child 426 PC1:
  before X=84 O=121 B=126 p=22335
  after  X=128 O=0   B=11  p=1
```

The decoded opcode128 branch is register-dependent:

```
pc = (p[pc] <= R[O[pc]]) and B[pc] or pc
```

so for both children it tests `1 <= R0`; the static trace stops there because R0 is invocation-dependent. No branch direction is assumed.


### Root return-overlay probe (bounded)

The same compatibility harness was applied to selected root overlay indices. In particular, slot14 (`w`, overlay over O) at PC4960 returned numeric candidate `4244215898`. This is exactly the lazy field referenced by the previously traced `return R0, w[4960]`. However, other overlay accesses in the same compatibility harness still fail on incomplete string/buffer emulation, so this value is retained as `PROBE_ONLY_NOT_VERIFIED` and is not used as a semantic conclusion.

## Net progress after batch 24

Strongly verified for this sample:

1. Normal lazy D2 keys are no longer merely known-plaintext guesses: 33/33 are accepted by q as complete prototypes.
2. D2 field IDs are randomized physical keys mapped to a stable 16-slot semantic descriptor.
3. O/B/X/p are the four base instruction-field arrays.
4. P/C/w/t are metatable-backed lazy overlays over p/B/O/X respectively.
5. Small children 252 and 426 have fully recovered prototype metadata and base instruction arrays.
6. Their first instruction is a self-decoder: opcode84 deterministically rewrites to opcode128, then execution becomes dependent on R0.
7. Four 14-byte D2 records are confirmed outside the normal prototype grammar.

Unresolved:

- faithful standalone emulation of every lazy `__index` branch;
- meaning/type of every lazily materialized field;
- R0 at child invocation sites;
- compact D2_14 grammar;
- complete reachable opcode semantics / source reconstruction.

## Reproduction artifacts

- `q_harness.lua` — minimal q root oracle.
- `q_all_children.lua/.out` — q test of all 37 lazy D2 children.
- `q_all_children_slots.lua/.out` — stable 16-slot schema evidence.
- `q_alias_check.lua/.out` — 33/33 overlay alias/metatable checks.
- `q_extended.lua` / `q_extended_fixed.out` — bounded lazy proxy probe.
- `check_op84.lua` — independent opcode84 arithmetic check.
- `root_correct.bin` — verified root plaintext.
