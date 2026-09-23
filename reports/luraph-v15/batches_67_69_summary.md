# Batches 67–69 summary

This pass adds an independent parser and a mode-correct multi-mode trace.

- Batch67 independently parses X/O/B/p for all 33 normal D2 children directly from recovered binary blocks; cross-checks against child252/426/162 pass.
- Batch68 corrects the opcode-selector model: mode51 uses O as the opcode field, while modes2/171/81 use X. Raw direct-host-load census finds 1 mode2, 14 mode171, 0 mode51, and 35 mode81 runtime-target rows with in-range destinations, but these remain structural until reachable.
- Batch69 traces child240 through mode2 -> mode171 -> mode51 and several reachable local XOR/self-decode layers. A post-unmask gameplay host-load survives at PC942 (`R24=b[26]`), but fresh execution stops earlier at PC706 because mode51 opcode199 requires parent capture `F[2]`.

Current refined verdict: VM->host bridging is proven in general (child162), and local transforms can create plausible gameplay host-loads, but no gameplay wrapper load has yet been reached from a fresh q-returned child state.
