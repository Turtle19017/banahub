# Batch 65 — corrected runtime-wrapper census for raw mode2 host loads

Date: 2026-09-23

Scope: correction/extension of Batch59 using direct `g_raw.lua` source evidence.

## `b[0]` is a gameplay/runtime wrapper

Direct source at `g_raw.lua` offset ~428503 defines:

```lua
[0]=function(...)
    return function(Z,F,c,l)
        -- enumerate TableMobSpawn
        -- normalize names containing "Lv."
        -- match spawn Parts by normalized/original name
        -- optionally choose nearest spawn to F.HumanoidRootPart
        -- otherwise test matching spawns within 200 studs of l.Position
    end
end
```

It references `TableMobSpawn`, Roblox `Part:IsA`, humanoid-root positions, distances and `table.insert`. Therefore host id 0 must be classified as a runtime/gameplay wrapper.

Batch55's bounded wrapper extractor did not include id0, so Batch59's `runtime_wrapper` column undercounted this one host id.

## Corrected Batch59 counts

Across the 33 normal D2 children there are still:

```text
71 raw mode2 opcode81 rows
```

After adding host id 0 to the runtime-wrapper set:

```text
17/71 raw rows target known runtime/gameplay wrapper IDs
```

Breakdown:

```text
16/17 -> destination register >= descriptor capacity
 1/17 -> destination in range
```

The sole in-range runtime target is:

```text
child140 PC45
opcode81 B=0 p=0
=> R0 = b[0]
```

But Batch60 already traced child140 from its real entry and showed execution switches to mode171 and stops at PC30/op159 because `w[30]` is numeric rather than a child descriptor. PC45 is not reached in the fresh q-returned state.

## Corrected verdict

The earlier phrase "none of the seven structurally valid host-load rows points to a runtime wrapper" was wrong. One does: child140 PC45 -> b[0].

The stronger reachability result remains:

```text
known runtime-wrapper raw targets: 17
structurally invalid destinations: 16
structurally valid runtime target: 1
reachable fresh runtime target: 0
```

So there is still no proven fresh mode2 gameplay-wrapper bridge, but the negative result now rests on the correct classification.
