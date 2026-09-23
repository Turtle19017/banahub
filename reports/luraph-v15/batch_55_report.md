# Batch 55 — numeric wrapper/runtime census in `g_raw.lua`

Date: 2026-09-23

Scope: static source census of the already recovered `g_raw.lua`; no gameplay execution.

## Goal

Separate the large plaintext host-library portion of `g_raw.lua` from code that is actually reached by the verified top-level VM path.

## Numeric factory definitions

The earlier lexical inventory identified **137 direct numeric function definitions** of the form:

```lua
[id] = function(...)
```

A bounded function-body extractor in this batch successfully isolated **108** of those definitions whose generated syntax could be closed unambiguously without a full Luau AST parser.

All 108 isolated definitions return an inner closure, matching the common host-wrapper pattern:

```lua
[id] = function(captured_host, ...)
    return function(...)
        ...
    end
end
```

The remaining 29 numeric function definitions are left unclassified by this extractor rather than guessed; they include structurally more complex generated entries such as VM/factory machinery.

## Runtime/global-touching bodies

Of the 108 cleanly isolated wrapper bodies, **79/108** directly contain at least one runtime/gameplay marker such as:

- `game`
- `workspace`
- `GetService`
- `ReplicatedStorage`
- `Players`
- `getgenv`
- `require(`
- `BossRuntime`
- `localPlayerFunctions`
- `HttpGet`
- `Remote`

Examples already seen in earlier source review include numeric entries for enemy enumeration, distance calculation, server hopping, combat helpers, boss lookup, movement and automation logic.

This batch deliberately calls these **runtime-touching wrapper bodies**, not “reachable gameplay”, because source presence does not imply execution.

## Relation to the verified root path

The verified top-level root path from batches 50–54 does not execute any nested host wrapper or gameplay call. It only mutates VM-local registers/instruction arrays, performs jumps, reads the final scalar overlay, and returns.

Therefore the 79 runtime-touching wrapper bodies are **present in the loaded host table but not reached by the currently proven top-level execution path**.

## Verdict

```text
numeric function definitions in prior inventory: 137
cleanly extracted wrapper bodies this pass:      108
runtime/global-touching extracted wrappers:       79
runtime-touching wrappers proven reachable:        0
```

The `0` reachability result is bounded to the verified root execution; it does not classify the wrappers as dead under every hypothetical descriptor mutation.

Artifacts:

- `b55_numeric_entries.tsv`
- `b55_numeric_entry_census.json`
