# Batch 48 — exhaustive broad-self-decode coverage of PC1394

Date: 2026-09-23

Scope: exhaustive static scan of the root instruction arrays after the two verified global XOR layers.

## Goal

Determine whether any other statically visible bulk instruction decoder could transform PC1394 before the verified return, beyond the already known PC4999 / PC14 layers.

Two broad-XOR handlers are relevant in factory2801:

- mode2 / mode81 opcode34: targets `O+1 .. O+B`, key `(p XOR s)&127`;
- mode171 opcode146: targets `O+1 .. O+p`, key `(B XOR s)&127`.

## Exhaustive scan result

Exactly three opcode34 rows have ranges containing PC1394:

| PC | O | B | p | target range | XOR key at PC1394 |
|---:|---:|---:|---:|---|---:|
| 4999 | 0 | 4977 | 2 | 1..4977 | 112 |
| 14 | 23 | 4944 | 57 | 24..4967 | 98 |
| 4966 | 23 | 4936 | 123 | 24..4959 | 32 |

No mode171 opcode146 row has a target interval containing PC1394.

## Reachability classification

- PC4999: VERIFIED EXECUTED. This is the first global XOR layer.
- PC14: VERIFIED EXECUTED. This is the second global XOR layer.
- PC4966: structurally valid third-layer candidate, but Batch47 proves it is skipped by the entire verified bootstrap through PC4960 return.

## Consequence

At the verified return boundary, there is **no remaining statically visible broad-range decoder** that can further transform PC1394.

PC1394 itself is currently opcode34, not a current-row self-decoder such as the known opcode91/107/84 families. Therefore the current evidence rules out a hidden third broad decode of PC1394 during the verified bootstrap.

This means the `w[1394]=child113` / `C[1394]={145,3}` pair is not consumed as a closure site during the currently traced bootstrap invocation.

A later invocation or a dynamically created decoder could still change it, but that would require new reachability evidence.
