# Batch 53 — verified root path is argument-independent

Date: 2026-09-23

Scope: bounded dependency analysis over the verified root invocation.

## Entry argument handling

Root entry PC4994 is opcode112 with:

```text
O=1
B=15
```

Using the corrected opcode112 semantics:

```text
R15 = table.pack(select(1,...))
```

So all arguments passed by the outer `(...)` call are captured into R15.

## Does R15 affect execution?

No verified instruction before PC4960 reads R15.

The path copies several initially nil registers into other registers, but none of those copied values is used in a conditional branch or indirect target before return.

All control-changing uses of R128 are sourced from immediate constants on the same path:

```text
PC5000      -> R128 = 16
PC21/22     -> R128 = 4967
PC4975/4976 -> R128 = 0
PC5/6       -> R128 = 7
PC15        -> R128 = 4959
```

PC4978 uses those deterministic values as the indirect trampoline target.

The self-decoder instructions (91/107) also depend only on their serialized instruction operands and fixed uint32 arithmetic, not on call arguments.

## Consequence

For the recovered descriptor and verified VM semantics:

```text
changing argument count: no effect on control path
changing argument values: no effect on control path
arguments reach gameplay/native calls: no
arguments affect root return values: no evidence
```

The verified root path is therefore deterministic with respect to `...`.

Combined with Batch52, this means the externally inert root behavior is not merely a special case caused by a particular argument vector.

## Bounded verdict

Within the recovered root descriptor state:

```text
all argument vectors
  -> same verified bootstrap path
  -> same PC4960 return boundary
```

This does not prove hypothetical pre-invocation descriptor mutation is impossible in some other caller/state. It does prove the actual outer argument forwarding alone cannot select a different path.

Artifact:
- `b53_argument_dependency.tsv`
