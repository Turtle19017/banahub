# Batch 40 — root.w[333]/w[356] are descriptor aliases, not closure-creation sites

Date: 2026-09-23

Scope: offline q-data/proxy analysis only.

## Goal

Test whether the two root overlay indices already known to materialize child descriptor symbol113 are themselves the actual VM instructions that instantiate the child closure.

Known from batch36:

```text
root.w[333] -> q(host,113)
root.w[356] -> same cached descriptor object
```

A real nested-closure handler in factory2801 requires both:

- `w[pc]` = child descriptor table
- a capture specification table in the companion overlay:
  - mode2/op158 uses `C[pc]`
  - mode171/op159 uses `P[pc]`

## Direct overlay probe

Each PC was evaluated on a fresh root q descriptor.

### PC333

```text
C[333] = 2450515686        (number)
w[333] = descriptor113     (table)
P[333] -> compatibility-harness OOB
t[333] -> compatibility-harness OOB
```

Descriptor113 still resolves to:

```text
factory=2801
entry=773
mode=2
regs=43
rows=788
```

### PC356

```text
P[356] = 4244215898        (number)
w[356] = descriptor113     (same cached table)
C[356] -> compatibility-harness string/char failure
t[356] -> compatibility-harness string/char failure
```

The failed overlay calls are not assigned semantics; they only show the current compatibility layer cannot complete those paths.

## Conclusion

Neither PC333 nor PC356 satisfies the verified closure-creation operand shape:

```text
child descriptor + capture-spec table
```

At PC333, the mode2 companion `C[333]` is numeric, not a capture table.
At PC356, the mode171 companion `P[356]` is numeric, not a capture table.

Therefore these two PCs are **descriptor-reference/materialization aliases**, not proven child113 closure-instantiation instructions.

This is useful negative evidence: finding a q child descriptor at `w[pc]` does not by itself identify the parent closure-creation site.

Artifact: `b40_root_333_356_all_overlays.out`.
