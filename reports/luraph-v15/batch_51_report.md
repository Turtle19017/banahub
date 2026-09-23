# Batch 51 — outer call-chain correction: root nil return is valid

Date: 2026-09-23

Scope: static audit of the outer `bf_main.lua` wrapper and the recovered `g_raw.lua` tail.

## Why this batch was needed

Batch50 proved that the verified root bootstrap reaches PC4960 with:

```text
R0 = nil
return nil, 4244215898
```

At first glance the outer source tail:

```lua
... ):H()(...);
```

could be misread as if the root's first return value were immediately called again. That interpretation is wrong.

## Outer H semantics

The outer `H` method's state machine performs the following relevant operations:

1. decode/decompress the g blob to 435,924 bytes;
2. decode/decompress the E blob to 845,745 bytes;
3. run `loadstring(buffer.tostring(g), "Luraph", nil)`;
4. verify the result is a function;
5. call the compiled g chunk with the E buffer;
6. return the value produced by that g chunk.

The recovered g chunk ends with:

```lua
... :oP(...);
```

and the already verified oP path returns the factory2801 root closure.

Therefore:

```text
H()
  -> compiled_g(E)
  -> oP(E)
  -> root closure
```

## Correct parse of the final syntax

The outer tail:

```lua
object:H()(...);
```

means:

```text
(H())(...)
  |
  + H() returns the root closure
  |
  + (...) invokes that root closure once
```

There is **no third function call** after the root closure returns.

Because the expression appears as a function-call statement at the end of the chunk, the root closure's return values are simply discarded.

Thus this is perfectly valid:

```text
root_closure(...)
  -> nil, 4244215898
```

No attempt is made to call `nil`.

## Correction

Rejected interpretation:

```text
H()
 -> root closure
 -> root closure()
 -> first return
 -> call first return again
```

Correct interpretation:

```text
H()
 -> root closure
 -> root closure(...)
 -> discard return values
```

## Consequence

Batch50's `R0=nil` finding is not a runtime contradiction.

Instead, it opens a more important question: does the verified root path perform any externally observable work before returning? Batch52 audits side effects on that exact reachable path.

Artifacts:
- `b51_outer_H_snippet.txt`
- `b51_call_chain.txt`
