# Batches 64–66 summary

This pass corrects two mode/classification errors and sharpens the bridge question.

1. **Child404 mode correction:** mode81 opcode81 is cleanup + bare return, not the mode2 host-load `R[p]=b[B]`. With a valid parent capture `F[1]`, child404 reaches mode81 PC9 and returns there; old PC10/PC11 mode81 conclusions are retracted.
2. **Runtime host-load census correction:** `b[0]` is a real gameplay/runtime wrapper. Raw mode2 runtime targets are 17, not 16. Sixteen have invalid destinations; the one valid row is child140 PC45 -> b[0], already proven unreachable in fresh state.
3. **Host bridge mechanism verified:** child162 mode171 PC8 loads `b[102]=coroutine.running`, and PC9 calls it through R7. Therefore VM→host load+call works on a traced path. The unresolved question is specifically a **reachable gameplay-wrapper bridge**, not host bridging in general.
