# Luraph v15 — batches 6–10 continuation

Scope: sample-local/static analysis only. No protected VM/gameplay/Roblox execution.
Ground truth: `E_raw.bin` SHA-256 `4e8818f7d6cb28b9145bed26c576c686f889af4d7b418d3d08f06c308349309b`.

## Batch 6 — Direct F7 string records

The 450-entry lookup was re-read from the verified E envelope. For entries whose raw value is already an in-range offset into the 92,516-byte first buffer and whose record tag is `0xF7`, the record layout is:

```
+0  F7
+1  Luraph 1..4-byte integer length
+N  encrypted payload[length]
```

For these direct records the payload decrypts with:

```
state0 = (49 + symbol_id) & 0xff
state  = (15 + 237*state) & 0xff
plain  = cipher ^ state ^ 49
```

Result: 159/159 direct F7 records decode as valid UTF-8, totaling 3,489 plaintext bytes. Largest = 2,311 bytes.
Examples: symbol 1 `X`, 7 `Destroy`, 27 `The metatable is locked`, 71 `Random`, 209 `readbits`.

## Batch 7 — Normalize all 450 lookup entries

Classification by structural resolution:

- 355 direct offsets into first buffer
- 91 offsets that become valid after `value XOR 339358108`
- 4 special values in `0x1ffffxxx`
- 0 other unresolved values

Special values:

| symbol | raw | `0x20000000 - raw` |
|---:|---:|---:|
| 230 | 536868472 | 2440 |
| 254 | 536867092 | 3820 |
| 267 | 536869709 | 1203 |
| 362 | 536868989 | 1923 |

Meaning of the four special values remains unresolved.

## Batch 8 — Record tag distribution after pointer normalization

446/450 lookup entries resolve to concrete first-buffer records.

| tag | count |
|---|---:|
| F7 | 213 |
| 84 | 62 |
| 13 | 46 |
| D2 | 38 |
| D4 | 27 |
| BD | 18 |
| 50 | 14 |
| 6F | 10 |
| 0B | 3 |
| 07,23,4D,75,79,EE | 2 each |
| 02,09,26 | 1 each |

Strongly grounded names only:
- `F7` = string-family record
- `D2` = prototype/block-family record; root symbol 187 is D2

Other tags remain unnamed.

## Batch 9 — Lazy F7 mechanism

The 91 XOR-resolved pointers split exactly into:

- 54 x F7
- 37 x D2

Multiple VM handlers in `g_raw.lua` have the same lazy materialization pattern:

1. read lookup entry
2. `offset = lookup[index] XOR 339358108`
3. write resolved offset back into lookup table
4. read the record length varint at `offset+1`
5. XOR the payload in-place with an instruction operand key
6. patch the current instruction to a direct/non-lazy opcode form

Therefore the 54 lazy F7 records are not expected to decrypt with the direct `symbol_id` key. Their missing plaintext requires correlating each lazy-load instruction with its operand key.

## Batch 10 — D2 child-prototype candidates

All 37 XOR-resolved D2 records have valid D2 + length framing.

After applying the normal q prototype LCG layer, an additional byte-wise XOR remains. A bounded known-plaintext hypothesis was tested using the observed prototype type byte `0x02`:

```
inferred_lazy_key = predecoded[0] XOR 0x02
plaintext = predecoded XOR inferred_lazy_key
```

Results:

- 37/37 begin with `0x02` by construction
- 33/37 independently also produce `00 00 00 00` at plaintext offsets `[6:10]`, matching the root-like structure
- four exceptions: symbols 148, 189, 359, 430
- all four exceptions are exactly 14-byte records and share a distinct compact form

This is strong evidence that the 37 D2 entries are child-prototype records and that the inferred byte is related to the runtime lazy XOR key, but it is not yet VERIFIED. The key still needs to be tied to the exact instruction operand used by each lazy handler.

## Current picture after 10 total batches

- 213 F7 string-family records
  - 159 direct/decrypted strings
  - 54 lazy-encrypted strings
- 38 D2 prototype-family records
  - 1 direct root prototype
  - 37 lazy child-prototype candidates
- 195 records in other tag families
- 4 special compact/immediate-looking values

Highest-value next target: correlate lazy-load instruction operands with the 91 XOR-obfuscated lookup entries. This should recover the 54 remaining F7 strings and independently verify the 37 child-prototype XOR keys.
