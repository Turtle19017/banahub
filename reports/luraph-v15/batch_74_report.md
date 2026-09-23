# Batch 74 — child240 is blocked at PC709 independently of capture values

Date: 2026-09-23

Scope: bounded q lazy-overlay oracle for child240 plus exact mode51 semantics. No VM/gameplay execution.

## Oracle reconstruction

The original 92,516-byte first E buffer was rebuilt exactly as `E_raw[1392:93908]`. Symbol240 is re-materialized from its D2 record (`record offset 85058`, payload offset 85061, length 7455, XOR key 175). A Lua 5.4 compatibility harness evaluates only requested lazy overlay cells.

A compatibility correction was made to the buffer-to-string shim so a buffer containing already-decoded one-byte string cells is rendered as bytes rather than rejected by `string.char`. This changes the earlier harness failure into concrete strings and does not alter q/factory semantics.

## Relevant lazy values

At PC709:

```text
P[709] = -10
w[709] = false
C[709] = "GetChildren"
t[709] -> q-oracle attempts readu8 at 339380279 and fails out of the 92516-byte buffer
```

The exact mode51 opcode93 body is:

```lua
R[p[pc]] = t[pc] + C[pc]
```

so the transformed PC709 attempts:

```text
R5 = t[709] + "GetChildren"
```

In the q-returned descriptor state this cannot be a valid arithmetic step. Even if hypothetical parent captures make PC706 and PC707 succeed, execution meets this independent lazy-operand/type blocker two instructions later.

This materially weakens the prior PC942 bridge candidate: capture recovery alone is not sufficient to make `R24=b[26]` reachable. A parent/pre-entry mutation must also alter descriptor/lazy state or control flow before PC709.

Verdict: `CHILD240_PC942_NOT_UNLOCKED_BY_CAPTURES_ALONE`.

Artifact: `b74_child240_lazy_probe.tsv`.
