# Batches 70–72 summary

- Mode171: 14 runtime+valid raw gameplay host-load candidates across child133/240/394/446; 0/14 are fresh-reachable.
- Mode81: 35 runtime+valid raw gameplay host-load candidates across 17 children; 0/35 are fresh-reachable as mode81 host loads. New traces show child138 stops on F[1] in mode2 and child339 stops on F[1] in mode171 without entering mode81.
- Host target semantics are now mapped for b[0], b[12], b[26], b[95], b[110]. The strongest candidate is child240 post-unmask PC942 -> R24=b[26], where b[26] is a large farming controller, but fresh execution stops earlier at PC706/F[2].

Current frontier: child240 parent capture F[2].
