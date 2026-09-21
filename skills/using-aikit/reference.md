# using-aikit — long form

Loaded on demand, never resident. This file holds what `aikit:using-aikit`
points to instead of carrying: the two routing tables in full, what phase 2.5
asks the expert, the rationale behind each archetype's fixed model, and the
failure matrix row by row. Nothing here is new — it is the same text, moved out
of a file that is read in full every time the method loads.

## Fast-Path vs Heavy-Path

`aikit:understanding-need` classifies every build/change/remove request into
one of two routes before phase 2 starts:

| | Fast-Path | Heavy-Path |
|---|---|---|
| When | ≤2 files, no API/contract break, no critical dependency | new component, refactor, contract change, or the estimate is wrong |
| Artifacts | none — no `spec.md`, no `plan.md` | the full chain in `SKILL.md` |
| Execution | one light dispatch, direct | phases 2 through 5 |
| Validation | `git diff --name-only` against the stated file set | `aikit:checking-plan-drift` per task |

Fast-Path is a bet, not a discount on rigor: if the diff exceeds the stated
files, or a second file turns out to need a change the first didn't predict,
that is drift — stop and re-route to Heavy-Path rather than absorbing it
silently. Getting the estimate wrong is cheap; treating the wrong estimate as
right is not.

## Phase 2.5, what the expert is asked

Phase 2.5 asks the expert what a doc cannot answer: *which files does this
spec touch, what are the traps, how would you cut it?* It writes no code, and
it writes no separate file — its answer lands as a section of `spec.md`, the
one artifact phase 2 and 2.5 share.

## Why each archetype gets the model it gets

Opus is not escalation-only here: on a machine with no cheaper evaluative-tier
model (this method originally ran planner/reviewer/probe on **fable**), opus
is also planner and reviewer's standing default — the probe moved to sonnet
instead, since a bounded investigation carries less downside than a bad plan
or a missed review finding. Restore the cheaper tier for all three if one
becomes available again.

Full table with the rationale per role, and the ledger-triggered escalation:
`aikit:executing-plans`.

## The failure matrix, row by row

| What happened | Direction |
|---|---|
| Technical unknown | **down** — probe, bounded. The plan does not move. |
| Execution error (red test, broken build) | **down** — fix rounds, bounded by the attempt budget. |
| Unplanned dependency, task too large | **up** — finish the independents, then re-plan. |
| Ambiguous or contradictory spec | **up** — stop now, back to phase 2 with the human. |

Going down is cheap, going up is expensive — but retrying a wrong plan is the
most expensive thing of all. Never spend an attempt budget on a failure that
belongs upward. **The attempt budget is counted in the ledger, not in your head.**

Full protocol, escalation contract and cycle report: `aikit:routing-failures`.
