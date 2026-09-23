# Batch 72 — semantic map of the strongest gameplay host targets

Date: 2026-09-23

Scope: direct static source classification of host wrappers referenced by structural gameplay-load candidates. Source presence is kept separate from reachability.

## Host targets

- `b[0]` (source offset ~428503): mob-spawn selector / nearest-spawn helper using TableMobSpawn, Part checks and positional distance.
- `b[12]` (~853): enumerates Workspace.Enemies and optionally Workspace.Characters.
- `b[26]` (~36899, ~6.1 KB): main farming controller with quest/material/boss selection, ReplicatedStorage remote invocation, movement, mob/boss loops, attack/mastery and inventory/world-travel logic.
- `b[95]` (~57411): checks for another character with HumanoidRootPart within 300 studs.
- `b[110]` (~319209): selects the nearest enabled tagged chest via CollectionService.

## Strongest current structural candidate

After the actually executed local transforms in child240, Batch69 identifies:

```text
PC942 / mode51 opcode147
X=26 p=24
=> R24 = b[26]
```

This is the strongest gameplay host-load candidate found so far because b[26] is a substantial farming controller and the row appears only after real reachable multi-mode/self-unmask processing.

However fresh child240 stops first at:

```text
PC706 / mode51 opcode199
requires parent capture F[2]
```

so PC942 remains structurally valid but unreachable in the current fresh state.

Raw mode171 candidates targeting b[12], b[95] and b[110] are closed as fresh-unreachable by Batch70. child140's b[0] load is closed by Batch60.

Current status:

```text
host bridge mechanism                    VERIFIED
gameplay wrapper bodies                  VERIFIED PRESENT
structural VM operands targeting them    VERIFIED
fresh reachable gameplay wrapper load    NOT YET VERIFIED
```

Highest-value next target: recover child240 parent capture F[2] and parent-register provenance.

Artifact: `b72_gameplay_host_targets.tsv`.
