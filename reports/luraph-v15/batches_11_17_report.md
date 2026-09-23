# Luraph v15 — batches 11–17

Scope: sample-local/static analysis only. No gameplay, Roblox, or original VM payload execution.
Ground truth: `E_raw.bin` SHA-256 `4e8818f7d6cb28b9145bed26c576c686f889af4d7b418d3d08f06c308349309b`; verified root SHA-256 `d65760495a18cbcf6ea82ca2c861952555c351b2e356cab95054cda371cb0c3c`.

## Batch 11 — test root lazy-handler candidates against recovered keys

Factory 2801 contains four lazy-materialization branches in mode 2:

- opcode 103: lookup symbol = `O[pc]`, XOR key = `p[pc]`
- opcode 77: lookup symbol = `O[pc]`, XOR key = `p[pc]`
- opcode 49: lookup symbol = `p[pc]`, XOR key = `B[pc]`
- opcode 69: lookup symbol = `p[pc]`, XOR key = `O[pc]`

Scanning the statically unmasked root rows gives 12 rows whose symbol operand happens to point at an XOR-obfuscated lookup record. However, every row for which the true key is now independently recoverable disagrees with the instruction operand key. Result: **0/12 key matches**.

This is useful negative evidence: merely landing in a source branch after specializing an opcode value does not make that row a valid/reachable lazy materializer. It directly confirms Astra's earlier warning that 256-way handler specialization contains fall-through/default aliases and must not be treated as a valid opcode table without reachability evidence.

See `b11_root_lazy_candidates.tsv`.

## Batch 12 — normal D2 child blocks structurally validate the inferred XOR keys

37 XOR-resolved lookup records use tag `D2`. Previously the extra lazy XOR byte was inferred only by forcing decrypted byte 0 to `0x02`.

After applying that inferred byte:

- 33 records are non-compact (`length > 14`)
- all **33/33** have `00 00 00 00` at plaintext offsets `[6:10]`
- all **33/33** contain byte sequence `95 71`, the sample's custom integer encoding of factory id **2801**, inside the expected prototype body
- 32/33 contain exactly one such marker; symbol 191 contains two occurrences because it is a much larger 20,062-byte block

The joint invariants make accidental success extremely implausible. For these 33 normal D2 records, the previously inferred byte is now **structurally validated for this sample**, even though it has not yet been tied to the exact runtime lazy-load instruction operand.

See `b12_d2_structural_validation.tsv` and `d2_recovered.zip`.

## Batch 13 — the four 14-byte D2 records are a real compact subtype

Symbols `148, 189, 359, 430` are exactly 14 bytes after D2 decryption. They do not have the normal zero-run/factory marker, but all four independently share the same template after applying their inferred XOR byte:

- byte 0 = `02`
- bytes `[1:7]` are a permutation of exactly `{10,11,13,14,16,17}` hex
- byte 7 = `02`
- byte 10 = `03`

Thus the four former "exceptions" are not failed decryptions. They are a separate **D2_COMPACT_14** record shape. Their precise semantic meaning is still unresolved.

## Batch 14 — 31/54 lazy F7 strings recovered exactly from sample-local corpus

For an XOR-resolved F7 record, removing the normal F7 LCG layer leaves plaintext XORed by one constant byte. Instead of guessing English strings, a corpus was built only from:

- lexical identifiers already present in `g_raw.lua`
- quoted strings already extracted from `g_raw.lua`
- the 159 already-verified direct F7 strings

For each lazy F7 record, all same-length corpus entries were tested for a constant XOR relation over every byte. **31 records have exactly one corpus match**, yielding both plaintext and lazy XOR key with no external dictionary.

Recovered examples:

| symbol | key | plaintext |
|---:|---:|---|
| 4 | 220 | `unpack` |
| 6 | 36 | `getgenv` |
| 68 | 254 | `Connect` |
| 145 | 175 | `writeu32` |
| 196 | 205 | `readf32` |
| 245 | 60 | `Vector2` |
| 317 | 198 | `__index` |
| 327 | 210 | `Vector3` |
| 343 | 29 | `setmetatable` |
| 405 | 99 | `lrotate` |

Full list: `b14_lazy_f7_exact.tsv`.

Status after this batch: **190/213 F7 records have exact plaintext** (`159 direct + 31 lazy`), leaving 23 lazy F7 records unresolved.

## Batch 15 — bounded identifier-form analysis for the remaining F7 records

No external name dictionary was introduced. For each unresolved F7 payload, all 256 XOR keys were tested only against the lexical shape `[A-Za-z_][A-Za-z0-9_]*`.

Only two records shrink to five or fewer identifier-shaped candidates:

- symbol 28, length 4: `xyk5`, `yxj4`, `tug9`, `utf8`
- symbol 340, length 18: five candidates including `Path2DControlPoint`

`utf8` and `Path2DControlPoint` are semantically plausible, but this batch intentionally leaves them **INFERRED**, not verified, because the shape constraint alone is insufficient to choose one candidate.

See `b15_identifier_candidates.json`.

## Batch 16 — root lazy-looking rows are proven false correlations

Cross-checking the exact F7 keys and structurally validated D2 keys against the 12 root rows from Batch 11 gives **0/12 matches**. Examples:

- root PC 235 points at symbol 173 with operand key 129, but symbol 173's exact key is 206 (`writeu8`)
- root PC 2975 points at symbol 145 with key 3, but exact key is 175 (`writeu32`)
- root PC 3034 points at symbol 227 with key 69, but exact key is 85 (`lshift`)
- root PC 1698 points at D2 symbol 148 with key 50, while the structurally validated D2 key is 56

Therefore those rows must not be used to recover lazy constants unless they are shown reachable in the active control-flow/mode state. This removes a misleading branch of analysis.

## Batch 17 — record framing taxonomy refined

Testing the lookup records structurally shows these tag families are uniformly `[tag][custom-varint length][payload]` for every occurrence in this sample:

`02, 07, 09, 23, 75, 79, D2, D4, F7`

Counts of the major verified framed families:

- `F7`: 213/213 length-framed
- `D2`: 38/38 length-framed
- `D4`: 27/27 length-framed, lengths 1..19

By contrast, these tags are **not** uniformly length-framed and must not inherit the same parser:

`0B, 13, 26, 4D, 50, 6F, 84, BD, EE`

This is another direct reason the old "one record layout for all 450 lookup entries" hypothesis fails.

See `b17_tag_framing.tsv`.

## State after 17 total batches

- outer unpack / LZMA: solved and independently verified
- E envelope: solved
- root prototype: verified
- bootstrap: traced through a real VM return boundary
- F7 strings: **190/213 exact**, 23 unresolved lazy records
- D2: **37 lazy child blocks decrypted with structural validation**
  - 33 normal prototype-shaped blocks
  - 4 `D2_COMPACT_14` blocks
- root-mode lazy-row shortcut: rejected (`0/12` key agreement)
- other lookup tag families: framing partially classified, semantics not yet named

Highest-value next step is no longer to brute-force the remaining strings. It is to recover instruction arrays from one small, structurally validated normal D2 child block (for example symbol 252, 47 bytes, or symbol 426, 70 bytes), then locate a **reachable** lazy-materialization opcode whose operand key can be compared against the recovered F7/D2 key set. That would establish the child serialization layout and close the key provenance loop.
