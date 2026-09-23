# Batch 41 — additional root aliases for descriptor113

Date: 2026-09-23

Scope: offline q lazy-overlay probing only.

## Finding

A bounded root-w scan beyond the first 700 PCs discovered two additional descriptor-valued indices:

```text
root.w[1120] -> factory2801 / entry773 / mode2 / 788 rows
root.w[1250] -> same descriptor shape
```

These are the same symbol113-derived descriptor family previously observed at root.w[333] and root.w[356].

Companion overlay probes:

### PC1120

```text
P[1120] = 1069115016   (number)
C[1120] = false        (boolean)
w[1120] = descriptor113
t[1120] = false        (boolean)
```

### PC1250

```text
w[1250] = descriptor113
P/C/t paths exceed the current compatibility shim and are left unresolved
```

The successfully decoded PC1120 companions are definitively not capture-spec tables. PC1250 is not promoted to a closure site because the required capture companion was not recovered.

## Static row context

After applying the two already-verified root XOR layers:

```text
PC1120 -> (opcode70, O=99, B=70, p=79)
PC1250 -> (opcode1262, O=99, B=45, p=5)
```

These values are recorded only as static recovered row fields; they are not treated as reachable opcode identities without control-flow evidence.

## Conclusion

Known descriptor113 references now include at least:

```text
333, 356, 1120, 1250
```

None is yet a verified closure-instantiation site.

The repeated child parsing performed by naive w-scans causes substantial memory growth. The next batch switches to intercepting recursive q calls so descriptor references can be enumerated without recursively parsing the child every time.
