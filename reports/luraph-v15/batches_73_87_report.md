# Luraph v15 — batches 73–87 — stateful closure provenance and child240 parent search

Date: 2026-09-23

Scope: offline/static reverse engineering of the already recovered Luraph v15 sample. No Roblox/gameplay execution. This report consolidates the requested 15-batch continuation (73–87) into one commit. The work remains offline/static: no Roblox/gameplay execution.

## Starting frontier

Batch72 left the strongest gameplay bridge candidate in child240:

```text
mode51 PC942 / opcode147
X=26 p=24
=> R24 = b[26]
```

where `b[26]` is a large farming/gameplay controller. The fresh child240 trace stops earlier at PC706/mode51 opcode199 on unresolved parent capture `F[2]`.

## Batch 73 — correct mode-specific nested-closure capture source

Re-reading the exact factory2801 source corrects an over-generalization from the prior frontier note.

### mode2 opcode158

Exact source begins:

```lua
J = w[W]
local S,I = C[W],F
...
L = b[J[J[2]]](b,J,nil,n,nil)
l[p[W]] = L
```

Therefore mode2/op158 uses:

```text
child descriptor = w[pc]
capture spec     = C[pc]
```

### mode171 opcode159

Exact source begins:

```lua
J = w[W]
local S,I = P[W],F
...
L = b[J[J[2]]](b,J,nil,n,nil)
l[B[W]] = L
```

Therefore mode171/op159 uses:

```text
child descriptor = w[pc]
capture spec     = P[pc]
```

Both constructors interpret the capture-spec as `(slot,kind)` pairs. The four kinds are unchanged:

```text
0 -> parent register value
1 -> new open cell {[6]=slot,[7]=parent_registers}
2 -> inherited parent F[slot]
3 -> cached/shared open cell for parent register slot
```

This matters for child240: parent discovery must probe `C[pc]` for a mode2/op158 site, but `P[pc]` for a mode171/op159 site.

**Later correction (Batch83):** these are two constructor forms, not the complete constructor inventory. Factory2801 contains five closure-construction forms across its four modes; Batch83 supersedes any exclusivity implied here.

Verdict:

```text
MODE2_CAPTURE_SPEC_SOURCE   = C[pc]
MODE171_CAPTURE_SPEC_SOURCE = P[pc]
PREVIOUS_P_ONLY_SHORTCUT    = CORRECTED
```

## Batch 74 — bind child240's lazy D2 record and the verified w-selector pattern

The existing lookup/prototype corpus gives the exact child240 record identity:

```text
symbol id          = 240
raw lookup[240]    = 339443166
pointer xor mask   = 339358108
resolved offset    = 85058
record length      = 7455
payload offset     = 85061
inferred D2 xor    = 175
plaintext sha256   = 200d6e5fba3c9b6ae9c54c7fe565685409ad2e0e9d3d06dc35cb62f72d82ad2e
```

Mechanical check:

```text
339443166 XOR 339358108 = 85058
```

The previously verified symbol113 materialization supplies an important selector constraint. Root rows PC333 and PC356 both have:

```text
O[pc] = 113
```

and both `w[pc]` accesses return the same cached symbol113 descriptor; the first access invokes `q(host,113)`.

Thus in the observed descriptor-producing `w` path, the direct O-field carries the child symbol selector. This is used only as an empirically verified filter, not as a universal proof for every lazy-handler branch.

Verdict:

```text
CHILD240_D2_RECORD_IDENTITY = BOUND
OBSERVED_W_DESCRIPTOR_SELECTOR = O[pc]
```

## Batch 75 — root O=240 candidate audit

The independently parsed root has 5,018 rows. Filtering its direct O array for value 240 yields exactly 11 rows:

```text
1437, 1457, 1600, 1668, 2542,
3194, 3384, 3786, 3831, 3978, 4042
```

Each candidate was probed through the q lazy overlay on a fresh root descriptor.

Result for all 11:

```text
w[pc] -> failure at readu8(339443166)
```

The value `339443166` is exactly the unresolved/raw child240 lookup value. None of these 11 base-state root rows performs the pointer normalization + D2 materialization needed to return the child240 descriptor.

Therefore:

```text
root direct O=240 rows             = 11
w materializes child240 descriptor = 0
```

This does not rule out a root parent after instruction mutation/self-decode; it rules out the 11 base-state direct-O candidates.

Verdict:

```text
ROOT_BASE_STATE_O240_PARENT_CANDIDATES = 0/11 MATERIALIZED
```

## Batch 76 — all normal-D2 O=240 candidate audit

The 33 independently parsed normal D2 children contain 21 direct-array rows with `O=240`, distributed as:

```text
child133: PC204, PC243
child138: PC33
child191: PC453, PC2025, PC2150, PC2837, PC3245, PC3865, PC4013
child240: PC401, PC771, PC943, PC1393
child334: PC405
child394: PC329, PC412, PC415, PC416, PC419, PC434
```

Each was probed through its q-returned lazy `w` overlay.

Results:

```text
external children: w access reaches raw lookup 339443166 and fails
child240 self rows: w access does not return a descriptor either
child240 descriptor materializations: 0/21
```

Combining the bounded root and normal-child base-state audit:

```text
root O=240 candidates     = 11
normal-child candidates  = 21
-----------------------------
base-state candidates    = 32
child240 materialized    = 0/32
```

The direct-array/base-state parent hypothesis is therefore strongly narrowed. The true creation site, if it is represented in this recovered corpus, likely requires a transformed row, lazy operand state, or another descriptor state before dispatch.

Verdict:

```text
BASE_STATE_DIRECT_O240_W_MATERIALIZATION = 0/32
```

## Batch 77 — child240's reached opcode158 rows are mode51 branches, not closure constructors

The verified child240 fresh trace before the PC706 blocker reaches these opcode158 rows:

```text
PC487, PC1000, PC702, PC697, PC705
```

All are executed in **mode51**. Exact mode51/op158 semantics are:

```lua
W = if R[p[W]] then B[W] else X[W]
```

so these are conditional branches, not nested-closure constructors.

The earlier reached phases contain no active closure constructor:

```text
mode2 reached:   PC1446,1447/48,1449/50,1451,1452
                 -> no mode2 opcode158

mode171 reached: PC614..618 opcode56, PC619 opcode126
                 -> no mode171 opcode159
```

Therefore no parent/child provenance can be extracted from child240's own reached path before PC706 using the two constructor forms known at this stage. The then-current target search was:

```text
mode2  opcode158  with w[pc] -> child240 and capture spec C[pc]
or
mode171 opcode159 with w[pc] -> child240 and capture spec P[pc]
```

and the base-state `O=240` census shows that such a site is not already exposed as a simple direct row in the currently tested root/normal-child descriptors. **Batch83 later broadens this search to mode2/op28, mode51/op72 and mode81/op135 as well.**

Verdict:

```text
CHILD240_PREBLOCK_REACHED_CLOSURE_CONSTRUCTORS = 0
MODE51_OPCODE158_ROWS = BRANCHES_NOT_CONSTRUCTORS
```

## State after batch 77

The strongest gameplay target remains child240 PC942 -> `R24=b[26]`, but the parent provenance question is now more sharply constrained.

```text
PC706 F[2] capture-cell dependency                 VERIFIED
mode2 parent capture-spec source                  C[pc]
mode171 parent capture-spec source                P[pc]
child240 lazy D2 record offset                    85058
base-state direct O=240 w candidates tested       32
base-state candidates materializing child240      0
child240 reached pre-PC706 closure constructors   0
```

Highest-value next direction for batches 78+:

1. search **post-self-unmask / post-self-decode active rows**, not raw rows, for mode2/op158 or mode171/op159;
2. prioritize transformed rows whose effective `O` becomes 240 or whose `w` access reaches the child240 D2 record;
3. once a site is found, recover mode-correct `C[pc]`/`P[pc]` capture pair #2 and its parent-register provenance;
4. feed that cell provenance into child240 PC706 and continue toward PC942.

---

## Batch 78 — stateful lazy overlay correction: child140 PC30 really materializes child113

Batch60 had traced child140 through its PC27 XOR layer and correctly obtained the effective row:

```text
PC30 / mode171 = opcode159, O=113, B=9, p=39
```

However the earlier `w[30]` probe was made against the q-returned base descriptor before applying the PC27 mutation to the direct X/O/B/p arrays. Re-running the q oracle with the mutation applied first changes the result materially.

After PC27 XORs PCs29..44 and patches itself:

```text
w[30] -> table
factory = 2801
entry   = 773
mode    = 2
regs    = 43
rows    = 788
```

This is exactly the previously identified symbol113 descriptor. The mode-correct capture source is also concrete:

```text
P[30] -> empty table
#P[30] = 0
```

Thus PC30 is not an invalid closure row. It is a real stateful nested-closure materialization:

```text
child140 PC30 --post-XOR--> construct child113 closure
```

This corrects the stale/base-state probe conclusion from Batch60.

Verdict:

```text
CHILD140_PC30_POSTXOR_W = CHILD113_DESCRIPTOR
CHILD140_PC30_CAPTURE_SPEC = EMPTY
BATCH60_BASE_STATE_W30_BLOCKER = SUPERSEDED
```

Artifact: `b78_child140_stateful_overlay.tsv`.

## Batch 79 — first concrete child invocation site: child140 calls child113 with its parent descriptor

The now-valid PC30 constructor unlocks the next instruction sequence.

Post-PC27 rows:

```text
PC29 = opcode23,  O=0,   B=7, p=0
PC30 = opcode159, O=113, B=9, p=39
PC31 = opcode33,  O=0,   B=7, p=9
```

Exact mode171 semantics:

```text
PC29/op23:
    R7 = descriptor Z

PC30/op159:
    child = w[30] = child113 descriptor
    capture spec = P[30] = {}
    R9 = factory2801(child113, captures={})

PC31/op33:
    R9(R7)
```

Therefore child113 finally has a mechanically identified parent-side invocation protocol:

```text
parent      = child140
closure PC  = 30
call PC     = 31
captures    = none
argument 1  = child140 descriptor Z
```

This directly answers the earlier Batch39 target of finding an actual invocation site for a q-materialized nested descriptor.

Verdict:

```text
CHILD113_PARENT = CHILD140_FOR_THIS_REACHED_PATH
CHILD113_CONSTRUCTOR_PC = 30
CHILD113_CALL_PC = 31
CHILD113_CAPTURE_COUNT = 0
CHILD113_ARG1 = PARENT_DESCRIPTOR_Z
```

Artifact: `b79_child113_invocation.tsv`.

## Batch 80 — the discovered call protocol still does not seed child113 R0

Child113's materialized descriptor has:

```text
entry PC773
mode2
opcode128
O=0
p=1
```

so its first semantic action remains:

```text
if 1 <= R0 then ...
```

The PC31 parent call supplies one Lua argument (`R7`, the parent descriptor), but factory2801 allocates the child's register table freshly before dispatch. Varargs are not automatically copied into VM registers; that only happens through explicit instructions such as opcode112. Child113 enters directly at opcode128, not a vararg materializer.

Its capture array is also empty, so no capture can supply R0 before entry.

Thus the newly recovered real invocation protocol does **not** explain the R0 gate:

```text
child113 called with 1 arg
child113 captures = 0
fresh VM R0       = nil
entry             = 1 <= R0
```

Unless the descriptor is mutated/prepared through another mechanism before this call, the invocation still blocks at entry.

Verdict:

```text
CHILD113_REAL_PARENT_CALL_FOUND = YES
CHILD113_R0_PROVENANCE_FROM_ARG1 = NO
CHILD113_FRESH_ENTRY_GATE_REMAINS = BLOCKED
```

Artifact: `b80_child113_entry_gate.tsv`.

## Batch 81 — child140 PC45 host-load candidate is definitively outside the reached mode path

For completeness, assume child113 somehow returns successfully from PC31. The parent continuation is deterministic enough to classify the old PC45 candidate.

Mode171 continuation:

```text
PC32/op31 -> clear R1..R3
PC33/op31 -> clear R4..R6
PC34/op31 -> clear R9..R10
PC35/op31 -> clear R7..R8
PC36/op49 -> R5 = 470216389
PC37/op126 -> switch to mode51, next PC41
```

Mode51 then executes:

```text
PC41/op94  -> R3 = 142135
PC42/op110 -> XOR/selfpatch PC38..40; O42 -> 170
PC43/op94  -> R6 = 37
PC44/op166 -> direct jump, next PC9
PC9/op102  -> selfdecode to opcode75, X=6,B=0,p=0
PC9/op75   -> W=R6=37, next PC38
```

The transformed loop is:

```text
PC38/op6   -> R5 = R5 + R3
PC39/op94  -> R6 = 37
PC40/op166 -> jump to PC9
PC9/op75   -> jump back to PC38
```

with:

```text
R3 = 142135
R5 starts at 470216389
```

so R5 increases by 142135 each cycle.

Critically, PC45 is never dispatched in mode2. After PC37 the active selector is mode51/O, and the control flow enters the PC38/39/40/9 loop. The old raw PC45 `X=81,O=0,B=0,p=0` structural interpretation as `mode2 opcode81 -> R0=b[0]` is therefore not on this reached path.

Verdict:

```text
CHILD140_PC45_MODE2_HOSTLOAD = NOT_REACHED
CHILD140_POST_CHILD113_CONTINUATION = MODE51_LOOP_PC38_39_40_9
```

Artifact: `b81_child140_mode51_loop.tsv`.

## Batch 82 — stateful child240 post-XOR O=240 rows do not expose a parent constructor

The Batch78 correction shows that lazy overlays must be evaluated **after** direct-array mutation. Therefore the Batch75/76 base-state `O=240` audit is retained only as a base-state result, not a global parent exclusion.

Applying child240's actually executed PC1451 mode2/op34 layer gives five rows whose effective O field becomes 240:

```text
PC853  -> X=86, O=240, B=86,  p=86
PC872  -> X=86, O=240, B=86,  p=86
PC994  -> X=86, O=240, B=86,  p=86
PC1117 -> X=18, O=240, B=105, p=230
PC1230 -> X=86, O=240, B=86,  p=86
```

None has the X opcode required for either of the two lazy-X constructor forms known at this point:

```text
mode2 constructor   requires X=158
mode171 constructor requires X=159
```

and the actually reached child240 path switches to mode51 at PC619, where O is the opcode selector rather than a child-symbol operand. Thus an effective `O=240` later in this child does not identify a child240 parent site. **Batch83/84 later audit the mode51 constructor opcode72 explicitly.**

The same post-PC1451 transformed corpus contains constructor-shaped X rows at PCs129,325,455,740, but their O fields are respectively:

```text
231, 76, 21, 221
```

not 240; none is reached as an active mode2/mode171 child240-parent constructor before the PC706 blocker.

Verdict:

```text
CHILD240_PC1451_POSTXOR_EFFECTIVE_O240_ROWS = 5
POSTXOR_O240_CLOSURE_CONSTRUCTORS = 0
KNOWN_REACHED_CHILD240_PARENT_CONSTRUCTOR = STILL_NOT_FOUND
```

Artifact: `b82_child240_postxor_o240.tsv`.

## State after batch 82

The most important new result is not merely another negative census: the stateful re-probe fixes a prior false blocker and reveals a **real nested closure creation + invocation chain**:

```text
child140
  PC27 self-unmask
    -> PC30 constructs child113 with zero captures
    -> PC31 calls child113(parent_descriptor)
```

This establishes that lazy `w/P/C` operands must be probed in the exact mutated descriptor state in which the instruction executes. Base-state overlay probes can be stale after self-unmasking.

For child240 the target remains:

```text
PC706 -> F[2] capture cell
...
PC942 -> R24 = b[26]
```

but the parent search must now be fully stateful:

1. reproduce a candidate parent's reached mutations;
2. only then probe `w[pc]` plus mode-correct `C[pc]`/`P[pc]`;
3. require `w[pc]` to identify child240;
4. require capture pair #2 to yield/inherit a cell compatible with PC706.

---

## Batch 83 — complete factory2801 closure-constructor inventory: five forms, not two

A source-wide search for the nested-factory call shape `b[J[J[2]]](b,J,nil,captures,nil)` finds **five** constructor sites across the four dispatch modes. This corrects the narrower model used in Batches73/77/82.

The complete inventory is:

| mode | opcode | child descriptor source | capture-spec source | closure destination |
|---:|---:|---|---|---|
| 2 | 28 | `R[p[pc]]` | `w[pc]` | `R[B[pc]]` |
| 2 | 158 | `w[pc]` | `C[pc]` | `R[p[pc]]` |
| 171 | 159 | `w[pc]` | `P[pc]` | `R[B[pc]]` |
| 51 | 72 | `P[pc]` | `C[pc]` | `R[X[pc]]` |
| 81 | 135 | `w[pc]` | `C[pc]` | `R[p[pc]]` |

All five use the same four capture kinds already recovered:

```text
kind0 -> copy parent register value
kind1 -> new open cell { [6]=slot, [7]=parent_registers }
kind2 -> inherit parent capture F[slot]
kind3 -> cached/shared open cell for parent register slot
```

Two consequences matter immediately:

1. searching only `X=158/159` cannot be a complete parent search;
2. mode51, which is exactly where child240 stops, has its own constructor (`opcode72`) whose descriptor is supplied by **`P[pc]`**, not `w[pc]`.

Verdict:

```text
FACTORY2801_CLOSURE_CONSTRUCTOR_FORMS = 5
BATCH73_TWO_FORM_MODEL                = SUPERSEDED_AS_INCOMPLETE
MODE51_CONSTRUCTOR                    = opcode72 / child=P / spec=C
MODE81_CONSTRUCTOR                    = opcode135 / child=w / spec=C
```

Local reproduction artifact: `b83_constructor_inventory.tsv`.

## Batch 84 — stateful audit of every child240 mode51/op72 row before the PC706 frontier

The exact child240 mutation state was replayed through all transforms already executed before the blocker:

```text
PC1451 mode2/op34
PC486  mode51/op110
PC1    mode51/op102 self-decode
PC999  mode51/op110
PC695  mode51/op110
PC696  mode51/op110
PC704  mode51/op110
```

In that resulting descriptor state, the O array (the mode51 opcode selector) contains six `opcode72` rows:

```text
PC105, PC196, PC249, PC622, PC983, PC1231
```

Mode51/op72 requires:

```text
child descriptor = P[pc]   -- must be a descriptor table
capture spec     = C[pc]
```

Fresh stateful q probes at each row produce:

```text
PC105 : P access fails
PC196 : P = 2                         (number)
PC249 : P access fails
PC622 : P = 1706231706                (number)
PC983 : P access fails
PC1231: P access fails
```

Therefore **0/6** currently materialize any child descriptor table, and none can instantiate child240 in the exact pre-PC706 state.

This is stronger than the earlier `O=240` filter because it tests the actual mode51 constructor primitive itself.

Verdict:

```text
CHILD240_PRE706_MODE51_OP72_ROWS       = 6
VALID_DESCRIPTOR_TABLE_IN_P            = 0/6
MODE51_SELF/NESTED_CONSTRUCTOR_PRE706  = NONE_PROVEN
```

Local reproduction artifact: `b84_child240_mode51_op72.tsv`.

## Batch 85 — mode-aware reached-constructor census: only one proven closure edge

The complete five-form constructor inventory was applied to the concrete execution traces already established for root and the nontrivial children.

The key rule is mode-sensitive: the same numeric opcode can mean something entirely different in another mode. Two useful examples are already present in the corpus:

- child339 reaches **mode171 opcode72**, but mode171/op72 is a comparison/branch, not the mode51 constructor;
- child404 reaches **mode171 opcode28**, but the register-sourced constructor form is **mode2/op28**.

Census through each trace's current stop/cycle boundary:

```text
root      -> no constructor before verified return PC4960
child138  -> no constructor before F[1] blocker
child140  -> YES: mode171 PC30/op159 -> child113
child162  -> no constructor on recovered coroutine/capture path
child188  -> no constructor before R61 arithmetic blocker
child240  -> no constructor before PC706 F[2] blocker
child339  -> no constructor before F[1] blocker
child404  -> no constructor before mode81 PC11 blocker
child357  -> no constructor before F[5] blocker
```

Thus the only currently **reached and semantically valid** closure edge remains:

```text
child140 --PC30/mode171/op159--> child113
```

with the already recovered empty capture spec.

Verdict:

```text
REPLAYED_TRACE_SET_WITH_CONSTRUCTOR_EDGE = 1
PROVEN_EDGE = child140 -> child113
PROVEN_REACHED_EDGE_TO_child240 = 0
```

Local reproduction artifact: `b85_reached_constructor_census.tsv`.

## Batch 86 — child240 requires two parent capture cells, not only F[2]

Reconstructing the exact effective rows after all pre-PC706 transforms sharpens the capture requirement.

The blocker row is:

```text
PC706 / mode51 / opcode199
X=2, p=9
=> J = F[2]
=> R9 = J[7][J[6]]
```

If that succeeds, execution falls through immediately to:

```text
PC707 / mode51 / opcode199
X=1, p=11
=> J = F[1]
=> R11 = J[7][J[6]]
```

Therefore resolving only `F[2]` is insufficient. Any real parent constructor that creates child240 for this path must provide at least:

```text
F[1] = cell-compatible capture
F[2] = cell-compatible capture
```

The capture-spec must therefore contain at least two `(slot,kind)` pairs. Directly cell-producing kinds are:

```text
kind1 -> new open cell
kind3 -> shared/cached open cell
```

`kind2` is acceptable only if the inherited parent capture is itself a cell. `kind0` would work only in the special case where the copied parent register already contains a compatible cell object; kind0 does not create the cell shape itself.

Later transformed rows also contain repeated opcode199 reads of F1/F2 (PC716, PC719, PC721, PC722), although those rows are not promoted to reached status until control flow beyond PC707 is established.

This supersedes the earlier single-capture framing:

```text
old filter: parent must explain F[2]
new filter: parent must explain cell-compatible F[1] AND F[2]
```

Verdict:

```text
CHILD240_MIN_CAPTURE_VECTOR_LENGTH = 2
CHILD240_PC706_REQUIRES = F[2] cell
CHILD240_PC707_REQUIRES = F[1] cell
PARENT_SEARCH_DUAL_CELL_FILTER = REQUIRED
```

Local reproduction artifact: `b86_child240_capture_reads.tsv`.

## Batch 87 — final parent screen for the current replayed state space

The now-correct constructor inventory, stateful lazy probes, and dual-cell filter were combined into one bounded parent screen.

### Reached valid constructor

```text
child140 PC30 / mode171 op159
child = child113
capture spec = empty
```

It cannot be the child240 parent by identity, and its zero-length capture vector cannot satisfy F1/F2 anyway.

### Stateful transformed constructor-shaped rows

- **child138 PC88 / mode2 op158 shape**: not reached before its F1 blocker; `w[88]` does not return a descriptor and `C[88]` is numeric.
- **child240 X=158/159 rows PC129/325/455/740**: no descriptor table is recovered from the required lazy child operand in the replayed state; additionally these rows are not active parent-constructor executions before PC706.
- **child240 mode51/op72 rows**: all six tested rows fail the required `P[pc] is descriptor table` gate (Batch84).
- **root, child162, child188, child339, child404, child357**: no constructor is reached before the already documented return/cycle/blocker.
- **mode2/op28 register-sourced constructor** remains important globally, but none is reached on the current concrete traces before their stopping points. A raw `X=28` row in a different mode is not evidence for this constructor.

Thus, within the execution state space actually replayed in Batches1–87:

```text
proven child240 parent site = NONE
```

This is deliberately **not** a whole-program no-parent claim. The remaining parent frontier is now much better defined:

1. descriptors currently blocked at fresh `R0` gates may require a real parent invocation protocol before they can expose deeper constructors;
2. capture-blocked children may themselves become parents only after their own F vectors are recovered;
3. mode2/op28 can instantiate a descriptor already held in a register, so future parent search must include descriptor-register provenance rather than only lazy `w/P` identity;
4. every lazy operand must be probed after the exact instruction mutations that precede its use.

Verdict:

```text
CHILD240_PARENT_IN_CURRENT_REPLAYED_REACHED_SET = NOT_FOUND
WHOLE_PROGRAM_CHILD240_PARENT                   = UNRESOLVED
NEXT_FRONTIER = parent invocation/R0 provenance + register-sourced constructor flow
```

Local reproduction artifact: `b87_child240_parent_screen.tsv`.

## Consolidated state after batch 87

The 15-batch pass produced two positive architectural results and several important corrections:

```text
1. child140 PC30 -> child113 is a real, reached nested-closure edge.
2. lazy operands must be evaluated in post-mutation state; base-state probes can be stale.
3. factory2801 has five closure-constructor forms, not only op158/op159.
4. child240 mode51/op72 candidates are all invalid in the exact pre-PC706 state.
5. child240 needs both F[2] and immediately F[1] as cell-compatible captures.
6. no currently replayed reached constructor instantiates child240.
```

The strongest gameplay bridge remains structurally unchanged:

```text
child240 PC942 / mode51 opcode147
X=26, p=24
=> R24 = b[26]
```

but reaching it now requires recovering the **real invocation context of child240**, including a minimum two-cell capture vector. The next high-value investigation is therefore not another raw opcode census; it is parent invocation provenance for the currently context-blocked descriptor families, with special attention to mode2/op28's register-sourced child descriptor path.

## Commit scope

This file intentionally consolidates exactly:

```text
batches 73–87 = 15 batches
```

No per-batch reports or TSV evidence files from this pass are intended for the GitHub commit; they remain local reproduction aids so the repository receives one compact report file.
