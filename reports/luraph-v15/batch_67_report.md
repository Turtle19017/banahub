# Batch 67 — independent child instruction-array parser

Date: 2026-09-23

Scope: offline parsing of the 33 normal recovered D2 child blobs. No VM/gameplay execution.

A standalone Python parser now reconstructs the four base instruction arrays `X/O/B/p` directly from each recovered `symbol_*.bin`, without invoking the Lua q harness.

The parser uses only already established serialization facts:

- normal D2 begins with type `0x02`;
- factory 2801 is encoded as byte sequence `95 71`;
- three randomized field IDs immediately precede that factory marker;
- register capacity is the custom integer immediately before those three IDs;
- the X array is the unique sequence of `rows` custom integers ending at the register-capacity field;
- after `95 71`, one randomized field ID precedes O;
- O has exactly `rows` integers;
- after fixed descriptor metadata and mode value 2, B and p each contain exactly `rows` integers.

Result:

```text
33/33 normal D2 children parsed
33/33 equal X/O/B/p row counts
mode-near-array metadata = 2 for all 33
```

Cross-checks against earlier q-derived ground truth pass exactly:

- child252: X `[84,148,31]`, O `[121,6,0]`, B `[115,0,0]`, p `[22335,1,0]`;
- child426: X `[84,84,21,151,52,34,104,8]` and matching O/B/p;
- child162: PC22 `(9,1,0,2)`, PC23 `(104,171,0,17)`.

This creates a reproducible instruction corpus independent of q for all subsequent structural scans.

Artifacts:
- `b67_parse_d2_arrays.py`
- `b67_child_arrays.json`
- `b67_child_array_validation.tsv`
