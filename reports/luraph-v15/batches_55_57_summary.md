# Batches 55–57 summary

The three-batch reachability census separates source presence from proven runtime execution.

- Prior lexical inventory: 137 numeric function definitions in `g_raw.lua`.
- Bounded extractor: 108 clean wrapper factories; 79 directly touch Roblox/runtime globals.
- q prototype census: 35 normal parsed descriptors (root + 33 D2 + tag09 child113), all factory2801/mode2.
- Verified root semantic opcode set contains no arbitrary host-table wrapper load and no VM-register function call.
- Therefore **0 of the 79 extracted runtime-touching wrappers are reached by the proven top-level path**.

Current architecture:

```text
g_raw host table
├─ VM/parser/factory machinery
│   └─ factory2801 ← all 35 parsed normal prototypes
└─ plaintext runtime/gameplay wrapper library
    └─ present, but not selected/called by verified root execution
```

The next high-value question is no longer whether plaintext gameplay code exists—it clearly does—but **what mutation or alternate reachable path, if any, ever bridges factory2801 into that host wrapper library**.
