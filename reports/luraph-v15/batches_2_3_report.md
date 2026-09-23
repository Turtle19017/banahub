# Luraph v15 sample — Batch 2 + Batch 3 trace

Input: `E_raw.bin` recovered from `bf_main.lua`.

## Batch 2 — exact outer envelope and 450-entry lookup

Recovered by following the initial `oP()` state machine, not by scanning for constants.

- Entry varint: `81 3b` -> `187`
- Flag byte: `00`
- Biased count: `98 82 66` -> `45848`
- Actual lookup count: `45848 - 45398 = 450`
- Lookup starts at `E_raw[6]`
- Lookup ends at `E_raw[1389]` (exclusive)
- Encoded lookup bytes: `1383`
- Entry length distribution: 1-byte=5, 2-byte=52, 3-byte=298, 4-byte=95
- `lookup[187] = 3363`
- `lookup[450] = 82985`

The 450 values are heterogeneous encoded values/pointers; they are not one fixed-size record array.

## Batch 3 — 92,516-byte buffer and 27,068-byte root

Immediately after the lookup table:

- `E_raw[1389:1392] = e4 85 52` -> `92516`
- `first_buffer = E_raw[1392:93908]`
- `oP()` creates a 92,516-byte buffer and copies this exact slice into it.
- `q(descriptor, 187)` uses `lookup[187] = 3363`.
- It starts at `first_buffer[3364]` (`lookup[187] + 1`).
- `first_buffer[3364:3367] = bc 81 53` -> root size `27068`.
- Root ciphertext is `first_buffer[3367:30435]`.
- Same bytes in global E coordinates: `E_raw[4759:31827]`.

Root decryption loop recovered from q states 233 -> 117/376:

```text
key0 = (175 + entry) % 256 = 106
for i = 0 .. 27067:
    key = (key * 237 + 15) % 256
    root[i] = ciphertext[i] XOR key XOR 27068
```

Because bytes are written with `buffer.writeu8`, the XOR result is effectively reduced to the low 8 bits when stored.

Recovered root:

- Size: 27,068 bytes
- SHA-256: `30bd0d8e8f651a00da09e3d6ac7d600204af557d390d9770723c036a2603f5eb`
- First 16 bytes: `11 05 06 0a 0c 0d 13 13 13 13 0f 8d 92 13 0b b4`

After all 27,068 bytes are produced, the loop reaches state 178 exactly.

Observed post-root path:

```text
178: read root[0] = 0x11
276 -> 424 -> 511 -> 201 -> 388
388: read root[1] = 0x05
189 -> 43
43:  read root[2] = 0x06
431 -> 418
418: read root[3] = 0x0a
```

At state 418, the parser performs an actual `buffer.fill` self-patch of the 92,516-byte first buffer:

- first-buffer offsets `167..176` inclusive
- count `10`
- fill byte `0x06`

Then state 235 checks `descriptor[6]`. It is nil on the initial path, so state 367 creates it. State 367 stores a table containing a large nested VM/interpreter closure into `descriptor[6]`, then returns state 78.

This is a useful structural boundary: the root block has been decoded, then the loader starts materializing a runtime VM/prototype object.

## Artifacts

- `batch2_decode_table.py`
- `batch2_table.tsv`
- `batch2_decode_table.txt`
- `batch3_root_probe.py`
- `batch3_root_probe.txt`
- `root_27068.bin`
