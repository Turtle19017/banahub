# Batch 56 — all observed serialized prototypes use factory 2801

Date: 2026-09-23

Scope: census of prototypes that the isolated q parser has actually reconstructed.

## Inputs

The census includes every structurally parsed prototype currently available:

- root symbol 187;
- 33 normal D2 child prototypes accepted by q;
- tag09 symbol113, materialized by the root lazy overlay during structural probing.

The four `D2_COMPACT_14` records are excluded because forcing them through the normal prototype grammar ends exactly at their 14-byte boundary and does not produce a normal prototype descriptor.

## Result

Total normal prototype descriptors:

```text
1 root
33 normal D2 children
1 tag09 child113
-------------------
35 descriptors
```

Every one of the 35 has:

```text
factory id   = 2801
initial mode = 2
```

No other numeric factory ID appears in the prototype factory slot of any descriptor parsed so far.

The descriptors vary widely in entry PC, register capacity and instruction count, but they all instantiate the same VM factory implementation.

## Architectural consequence

This separates two very different numeric namespaces that were easy to conflate earlier:

```text
serialized prototype factory slot
    -> 2801 for all 35 observed normal prototypes

host table numeric wrapper entries
    -> many other IDs, including plaintext Roblox/gameplay wrappers
```

Thus the plaintext numeric wrappers found in `g_raw.lua` are not observed as alternative serialized prototype factories. They are host-library values that the VM may load/call through its instruction set when a reachable path requests them.

That distinction is important because source-level inventory alone greatly overstates runtime reachability.

## Verdict

```text
q-parsed normal descriptors: 35
unique factory IDs:          {2801}
unique initial modes:        {2}
observed gameplay-wrapper IDs in factory slot: 0
```

Artifact:

- `b56_descriptor_factory_census.tsv`
- `b56_descriptor_factory_census.json`
