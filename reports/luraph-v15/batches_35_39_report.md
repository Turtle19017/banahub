# Batches 35–39 — mode81, nested q materialization, tag09 descriptor, and fresh-call constraints

Date: 2026-09-23

Scope remains offline. No Roblox/gameplay execution. Analysis uses the already-decompressed sample, q-data oracle, and isolated factory arithmetic/state semantics.

## Batch 35 — full mode81 and corrected child404 boundary

The full factory2801 source contains the complete mode81 dispatch. Child404 reaches mode81 with W=9.

After its verified XOR/self-decode layer:

- PC9  = opcode81, O=0, B=0, p=0
- PC10 = opcode145, O=0, B=49, p=2
- PC11 = opcode126, O=51, B=0, p=0
- PC12 = opcode28, O=4, B=9, p=13

In mode81, PC9 performs:

```lua
R[p[pc]] = b[B[pc]]
```

thus:

```text
R0 = b[0]
```

This is the first concrete provenance for R0 in a traced child path.

PC10 computes:

```text
R145 = bit32.bnot(R2)
R2 = 1
=> R145 = 0xfffffffe
```

PC11 requires `t[11]`:

```lua
R0[t[11]] = R0
```

The q overlay oracle returns `t[11] = nil`, and P/C/w/t are nil at PC3..14. Also `b[0]` is function-valued in the recovered top-level table. Therefore a fresh invocation cannot safely continue beyond PC11.

Correction: child404 does not reach PC12 on a fresh q-returned descriptor. It reaches a mode81 access that requires parent-side/runtime preparation not present in the bare descriptor.

## Batch 36 — verified root.w[333] -> q(host,113) mapping and cache

Instrumenting the q lazy overlay callback gives a direct nested-descriptor materialization event:

```text
root.w[333]
  -> q(host, 113)
```

The returned descriptor is:

- factory 2801
- entry PC 773
- mode 2
- register capacity 43
- 788 instruction rows

A subsequent access to `root.w[356]` returns the same descriptor table object and does not call q again.

Thus:

```text
w[333] --first access--> q(host,113) --> descriptor113
w[356] -------------------------------> same cached descriptor
```

This is a real parent/data-layer mapping, stronger than raw opcode scans.

## Batch 37 — tag09 symbol113 is descriptor-producing

Symbol113 resolves to a tag09 record.

Observed framing:

- lookup[113] = 47868
- record starts `09 9F 15 ...`
- length bytes `9F 15` decode to 3989
- total framed size = 3992 bytes

During `root.w[333]` materialization, q mutates 3960/3992 bytes of this record; 3960 bytes become exactly `0x76`, and the record contains 3976 bytes of `0x76` afterward.

The host cursor remains unchanged at 93908.

q returns a 788-row factory2801 descriptor whose entry row is:

```text
PC773:
opcode128
O=0
B=27
p=1
```

Therefore tag09 symbol113 is another descriptor-producing record family in this sample. This is not generalized beyond the single observed tag09 occurrence.

## Batch 38 — child188 reaches a second independent context-dependent failure

Child188 metadata:

- entry PC2
- 22 rows
- register capacity 10

Trace:

1. PC2/op91 self-decodes to opcode9 and sets `R2=1`.
2. PC3/op91 self-decodes to `(104,O=171,B=0,p=9)`, switching to mode171 with W=10.
3. mode171 PC10/op10 computes `R5=(R2<=0)=false`.
4. PC11/op28 false arm routes to PC4.
5. PC4/op145 unmasks PC13..22 using key `(71 XOR s)&127`, then becomes opcode54.
6. PC5/op45 requires `R61` in arithmetic: `R11 = 7768 ^ R61` where Lua `^` is exponentiation.

Fresh register state has not initialized R61, so the direct fresh invocation stops at this point.

This independently confirms the same architectural theme seen in child404: q can return a structurally valid prototype that is not standalone-callable without additional parent/runtime context.

## Batch 39 — invocation architecture

Evidence now points to a distinct separation between structural decode and callable runtime state.

Observed constraints:

- 21 normal D2 children begin with opcode128 and immediately require R0.
- tag09 symbol113 also begins with opcode128 and requires R0.
- child404 bootstraps far enough to assign `R0=b[0]`, then requires missing `t[11]`.
- child188 bootstraps/self-unmasks, then requires uninitialized R61.
- child162 requires a parent-provided capture cell F[1].

Current model:

```text
q(...)
  -> structurally valid descriptor
  -> not necessarily directly callable
       |
       + parent capture vector F
       + register/context initialization
       + instruction self-patching
       + lazy overlay materialization
       -> callable runtime state
```

This does not establish that constrained children are dead/decoy code. The stronger next target is to recover the actual parent instruction/context that consumes `root.w[333]` / `root.w[356]` and instantiates symbol113, closing the chain from parent instruction to child invocation.
