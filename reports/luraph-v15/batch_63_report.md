# Batch 63 — complete fresh-state audit of all register-valid raw mode2 host-load candidates

Date: 2026-09-23

Scope: close the remaining candidates from Batch59 using the already verified entry-normalization rules. No gameplay execution.

## Remaining candidates

Batch59 found seven raw mode2 opcode81 rows with destination register inside descriptor capacity, spread across six child descriptors:

```text
sid133  PC168  -> b[20]  = coroutine.status
sid140  PC45   -> b[0]
sid357  PC32   -> b[36]  = xpcall
sid378  PC59   -> b[25]  = error
sid394  PC270  -> b[91]  = select
sid435  PC223  -> b[228]
sid435  PC225  -> b[228]
```

Child140 was traced in Batch60, child357 in Batch61, and child378 in Batch62. This batch closes sid133, sid394 and sid435.

## sid133

Descriptor:

```text
entry = PC323
regs  = 38
rows  = 527
```

Entry is already:

```text
opcode128 O=0 B=32 p=1
```

Fresh invocation requires:

```text
1 <= R0
```

but factory2801 begins with `R0=nil`. Therefore sid133 stops at entry before PC168 can be reached.

## sid394

Descriptor:

```text
entry = PC495
regs  = 133
rows  = 506
```

Entry is:

```text
opcode128 O=0 B=25 p=2
```

Fresh invocation requires:

```text
2 <= R0
```

with `R0=nil`, so sid394 also stops at entry before the PC270 `select` host-load candidate.

## sid435

Descriptor:

```text
entry = PC30
regs  = 45
rows  = 314
```

Raw entry:

```text
opcode84 O=121 B=81 p=22329
```

Exact opcode84 arithmetic gives:

```text
opcode128 O=0 B=36 p=7
```

So fresh execution immediately requires:

```text
7 <= R0
```

with `R0=nil`. Both PC223 and PC225 are therefore unreachable in the fresh descriptor state.

## Complete result for the seven raw register-valid mode2 host loads

| sid | PC | host entry | fresh-state status |
|---:|---:|---|---|
| 133 | 168 | `b[20]=coroutine.status` | blocked at entry opcode128/O0 |
| 140 | 45 | `b[0]` | blocked earlier at mode171 PC30 invalid closure descriptor operand |
| 357 | 32 | `b[36]=xpcall` | requires parent capture `F[5]` at PC56 before candidate |
| 378 | 59 | `b[25]=error` | blocked at normalized entry opcode128/O0 |
| 394 | 270 | `b[91]=select` | blocked at entry opcode128/O0 |
| 435 | 223 | `b[228]` | blocked at normalized entry opcode128/O0 |
| 435 | 225 | `b[228]` | blocked at normalized entry opcode128/O0 |

## Verdict

For the q-returned descriptor states currently reconstructed:

```text
raw register-valid mode2 host-load rows: 7
proven reachable host-load rows:          0
hard fresh-entry blocked rows:            5
blocked earlier by invalid closure state: 1
requires unresolved parent capture:       1
```

No direct mode2 bridge from a parsed child descriptor into the host library is currently proven reachable.

This does not exclude later descriptor mutation or a valid parent capture path for sid357; it closes the raw/fresh-state candidate census without overclaiming dead code.

Artifacts:
- `b63_remaining_mode2_hostload_audit.tsv`
- `b63_child435_rows.out`
