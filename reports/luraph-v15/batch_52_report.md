# Batch 52 — verified root path is externally inert

Date: 2026-09-23

Scope: bounded side-effect audit of the already verified root bootstrap path through PC4960.

## Goal

Now that Batch51 established the root return values are simply discarded by the outer chunk, this batch asks whether the root invocation performs any externally observable action before returning.

## Executed instruction categories

Every reachable instruction on the verified path falls into one of these categories:

- pack/copy values inside the fresh VM register table;
- write immediate integers into VM registers;
- self-decode one current instruction row;
- XOR/self-patch ranges of the root's own O/B/X/p instruction arrays;
- direct/indirect control-flow jumps;
- final return.

No reachable instruction on this path:

- calls a gameplay closure;
- accesses Roblox globals/services;
- invokes a remote;
- creates a nested closure;
- writes through a parent capture cell;
- calls a user/native function from a VM register;
- writes into `b.a`;
- invokes any of the plaintext gameplay factories in `g_raw.lua`.

## Internal mutations

The path does mutate the root descriptor's own instruction arrays:

- PC4999 applies the first broad XOR;
- PC23, PC3, PC4, PC7, PC15, PC16 self-patch individual rows;
- PC14 applies the second broad XOR.

These are VM-internal descriptor mutations, not external program side effects.

## Final lazy overlay read

PC4960 returns:

```lua
return R0, w[4960]
```

Batch26 independently measured:

```text
w[4960] = 4244215898
lookup changes = 0
payload-buffer changes = 0
```

So the final overlay access does not mutate the host lookup table or the 92,516-byte serialized payload buffer.

The lazy overlay may cache its resolved scalar inside descriptor-local proxy state; that is not promoted to an external side effect.

## Bounded verdict

For the verified top-level root invocation:

```text
external gameplay/network side effects: none observed on reachable path
host lookup mutations: none on final overlay access
host payload mutations: none on final overlay access
descriptor/register self-mutation: yes
return: nil, 4244215898
```

This means the currently verified root path is externally inert: it performs VM-local setup/self-decoding and returns.

This conclusion is intentionally limited to the proven reachable bootstrap path. It does not by itself prove every serialized child/gameplay closure is dead under every hypothetical descriptor mutation/state.

Artifact:
- `b52_root_side_effect_audit.tsv`
