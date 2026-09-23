# Batches 30–34 — capture provenance, child162 cycle, child404 trace, and entry corrections

Date: 2026-09-23

Scope remains offline. No Roblox/gameplay execution. The original protected file is not modified. `q` data decoding and isolated VM arithmetic/state semantics are used as oracles; no full target gameplay path is executed.

## Starting point

Batch 29 had traced child 162 to mode171 PC10/opcode133 and described `F[1]` as if it were necessarily `b.a[1]`. Re-reading factory 2801 shows that this is only true for the root factory invocation. Nested factories receive a newly constructed capture array as their fourth argument. This correction changes the interpretation of child upvalue access.

## Batch 30 — exact nested-closure capture protocol

Root factory construction uses:

```lua
factory(b, root_descriptor, nil, b.a, nil)
```

Because the extracted factory signature contains duplicate parameter names, the fourth argument is the effective local `F`; for the root this is `b.a`.

Nested closure creation is different. Mode2 opcode158 and mode171 opcode159 both build an array `n` from a serialized capture specification and then call:

```lua
child_factory(b, child_descriptor, nil, n, nil)
```

Therefore, inside a nested child:

```text
F = capture-array n
```

and is **not automatically** the top-level `b.a` table.

Four capture kinds are implemented:

| kind | materialization |
|---:|---|
| 0 | by-value copy of `parent_register[slot]` |
| 1 | new open cell `{[6]=slot,[7]=parent_register_table}` |
| 2 | inherited parent capture `parent_F[slot]` |
| 3 | shared open cell, reused/created through cache `c[slot]` |

Kinds 1 and 3 explicitly produce the `{[6],[7]}` cell layout later consumed by child opcodes such as 133.

Verdict: `CHILD_F_IS_CAPTURE_VECTOR`, high confidence from factory source.

## Batch 31 — raw closure-opcode candidates rejected as provenance

After applying the two already-verified root XOR layers, the 5,018-row root contains exactly one raw opcode158:

```text
PC4710: opcode158 O=127 B=81 p=29
```

However the q lazy proxy gives:

```text
w[4710] = -6
```

where a real mode2 opcode158 requires `w[pc]` to be a child descriptor table. Thus PC4710 is not usable as a verified closure-creation site in the recovered state.

Across all 33 normal child prototypes, raw scans found 37 rows whose unexecuted X field equals opcode158 or opcode159. They occur in 10 prototype symbols, heavily concentrated in large child 191. For all 37 rows:

```text
P[pc] = nil
C[pc] = nil
w[pc] = nil
t[pc] = nil
```

under the q overlay oracle.

This independently reinforces the earlier rule: a raw opcode value alone does not establish reachability or semantic validity before self-decode/mode control has been resolved.

Verdict: `RAW_158_159_SCAN_NOT_A_PARENT_MAP`.

## Batch 32 — child162 continuation and capture-cell semantics

Child 162 metadata:

```text
factory 2801
entry PC22
mode 2
23 rows
8-register capacity
```

The previous trace reached mode171 PC10/opcode133. The corrected semantics are:

```lua
J = F[p[pc]]
J[7][J[6]] = R[B[pc]]
```

At PC10:

```text
p=1
B=7
```

so this writes the result of `coroutine.running()` into **capture cell `F[1]`**.

The deterministic continuation is:

```text
PC10/op133: capture F[1] := current coroutine
PC11/op143: R2=0; R5=17; paired immediate, skips PC12
PC13/op11 : direct W=19 -> PC20
PC20/op120: W=R5=17 -> PC18
PC18/op10 : R7=(R2<=0)=true
PC19/op28 : true arm -> PC6
PC6/op145 : one-time XOR patch of PC1, then opcode6 becomes 54
PC7/op45  : R26 = 12116 XOR R11
PC8/op1   : R7=coroutine.running
PC9/op115 : R7=coroutine.running()
PC10/op133: write current coroutine to F[1] again
...
```

After the one-time PC6 patch, opcode54 at PC6 has no state-changing body in mode171. With the recovered arrays and a valid capture cell `F[1]`, the control state repeats:

```text
PC6 -> 7 -> 8 -> 9 -> 10 -> 11 -> 13 -> 20 -> 18 -> 19 -> PC6
```

No P/C/w/t lazy operand is present anywhere in child162 (already verified in batch29), so this cycle is not waiting on a hidden lazy constant.

Interpretation is deliberately bounded: **if child162 is invoked in the recovered state with a valid F[1] capture cell, its recovered path becomes a tight coroutine/context-update cycle.** This does not prove the original program actually invokes child162 in that state; the parent may patch it or leave it unreachable.

Verdict: `RECOVERED_STATE_CYCLE`, bounded to the recovered descriptor.

## Batch 33 — child404 deterministic trace and a real capture write

Child 404 is a smaller 24-row prototype:

```text
entry PC20
mode2
register capacity 9
```

Its bootstrap can be traced without gameplay values.

### First stage

```text
PC20/op9 : R2=0
PC21/op34: XOR PC1..14 using key=(60 XOR s)&127; opcode21 -> 92
PC22/op9 : R5=2
PC23/op124 -> PC24
PC24/op162: indirect through R5=2 -> PC3
```

Post-XOR/self-decoded rows:

```text
PC3:  opcode91 -> opcode108 (O=0,B=2,p=4)
PC4:  opcode84 -> opcode154 (O=12,B=4,p=4)
PC5:  opcode91 -> opcode60  (O=1,B=94,p=0)
PC6:  opcode91 -> opcode9   (O=1,B=0,p=2)
```

Execution:

```text
PC3/op108: R4=(R2<=0)=true
PC4/op154: true -> PC5
PC5/op60 : J=F[1]; J[7][J[6]]=C[5]
```

A direct q proxy probe for child404 shows `C[5] == nil`. Therefore this verified instruction clears the captured upvalue cell represented by `F[1]`:

```text
F[1].parent_register[F[1].slot] = nil
```

Then:

```text
PC6/op9 : R2=1
PC7/op9 : R5=2
PC8/op124 -> PC24
PC24/op162 -> PC3
```

Now PC3 makes `R4=false`, so PC4 takes its other arm:

```text
PC4/op154 -> PC13
PC13/op104: switch mode 2 -> 171, W=12
PC12/mode171 op28: R4=false -> PC14
PC14/mode171 op126: switch mode 171 -> 81, W=9
```

The current `factory2801_rough.lua` extraction is truncated before the mode81 dispatch body, so the batch stops exactly at this verified mode transition rather than guessing mode81 semantics.

Verdict: `CHILD404_TRACED_TO_MODE81_BOUNDARY`.

## Batch 34 — fresh-call gate audit and opcode112 correction

### 34A. 21 opcode128-gated children

After entry normalization, 21/33 normal children begin with mode2 opcode128 and all have:

```text
O = 0
```

Mode2 opcode128 performs:

```lua
if p[pc] <= R[O[pc]] then
    pc = B[pc]
end
```

so all 21 immediately read `R0`.

Factory 2801 creates the register table with `table.create(capacity)` and does not populate R0 before entering the dispatch loop. Thus, for a **fresh direct call of the q-returned descriptor**, the first comparison would be `integer <= nil` and would fail unless some pre-entry mutation/context not represented in the plain descriptor changes the situation.

This is evidence against treating all 21 descriptors as directly callable standalone functions. It is not evidence that they are dead code.

### 34B. Correct semantics of opcode112

The earlier batch28 summary simplified opcode112 too aggressively. Exact mode2 semantics are:

```lua
args = {...}
J = O[pc]
L = B[pc]

table.move(args, 1, J-1, L, registers)
registers[L+J-1] = table.pack(select(J, ...))
```

Therefore:

**symbol 249** (`O=1, B=2`):

```text
copies zero raw arguments
R2 = table.pack(select(1,...))
   = packed table containing all varargs
```

**symbol 357** (`O=2, B=37`):

```text
R37 = arg1
R38 = table.pack(select(2,...))
    = packed table of args2..
```

So opcode112 is a **vararg prefix + packed-tail materializer**, not simply a positional vararg copy.

Verdict: `OPCODE112_SEMANTICS_CORRECTED`.

## Net progress after batch 34

Newly established:

1. Nested-child `F` provenance is corrected: it is the capture vector constructed by parent closure creation, not automatically `b.a`.
2. The VM implements four explicit capture kinds, including reusable open-upvalue cells.
3. Raw opcode158/159 values are again shown insufficient for parent/child mapping.
4. Child162 can be traced beyond PC10 and, in its recovered state, settles into a deterministic coroutine/capture-cell update cycle.
5. Child404 gives a second concrete child trace, including a verified write through capture cell F[1] and mode transitions `2 -> 171 -> 81`.
6. All 21 normalized opcode128 entry gates immediately depend on R0; fresh direct invocation remains unsupported by the recovered descriptor alone.
7. Opcode112 is corrected to `prefix copy + table.pack(select(...))` semantics.

Still unresolved:

- the true parent instruction that instantiates child162/404;
- exact serialized capture-spec record associated with each child;
- mode81 dispatch body (the current extracted `factory2801_rough.lua` is truncated there);
- provenance of R0 for opcode128-gated descriptors;
- complete reachable opcode/closure graph.
