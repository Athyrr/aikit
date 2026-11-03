# using-helia — long form

Loaded on demand, never resident. This file holds what `helia:using-helia`
points to instead of carrying: the two routing tables in full, what phase 3
asks the expert, the rationale behind each archetype's model and effort, and the
failure matrix row by row. Nothing here is new — it is the same text, moved out
of a file that is read in full every time the method loads.

## Fast-Path vs Heavy-Path

`helia:understanding-need` classifies every build/change/remove request into
one of two routes before phase 2 starts. The condition is strict, not a
judgement call:

| | Fast-Path | Heavy-Path |
|---|---|---|
| When | **≤2 files changed, and no public API / data-schema change** | new component, refactor, contract change, or the estimate is wrong |
| Artifacts | none — no `spec.md`, no `vault/<project>/<feature>/plan.md`, no intermediate documentation of any kind | the full chain in `SKILL.md` |
| Execution | one light dispatch, direct, in the main session | phases 2 through 5, subagent-driven |
| Validation | `git diff --name-only` against the stated file set | `helia:checking-plan-drift` per task |

Fast-Path is a bet, not a discount on rigor: if the diff exceeds the stated
files, or a second file turns out to need a change the first didn't predict,
that is drift — stop and re-route to Heavy-Path rather than absorbing it
silently. Getting the estimate wrong is cheap; treating the wrong estimate as
right is not.

## Phase 3, what the expert is asked

Phase 3 asks the expert what a doc cannot answer: *which files does this
spec touch, what are the traps, how would you cut it?* It writes no code, and
it writes no separate file — its answer lands as a section of `spec.md`, the
one artifact phase 2 and 3 share.

## Why each archetype gets the model and effort it gets

`helia:planner` and `helia:reviewer` run on opus at effort `high`: a wrong
plan is the most expensive failure this method has, and the review is the
net every task passes through — both are evaluative work where judgement
pays. `helia:explorer`, `helia:implementer`, `helia:probe` and
`helia:verifier` run on sonnet at effort `medium`: high-volume reading,
bounded production work, a finding, a command and its output — none needs
the ceiling. The values themselves live in `agents/*.md`, mirrored in
`helia:using-helia`.

Effort rises automatically on a mechanical signal; the model changes only
with the human partner. Two failed attempts stop the loop and hand the
decision to a human, rather than buying a bigger model or a higher effort.
The signals: `helia:executing-plans`' escalation table — this file does not
copy it.

## The task-review exemption

A per-task `helia:reviewer` dispatch is skipped when all three hold: the
task's tests pass, its diff is 30 lines of code or fewer, and it changes no
public API or shared contract. The orchestrator records the exemption on the
task's checklist line in `vault/<project>/<feature>/plan.md` and moves to
the next task without a reviewer round-trip.

This narrows the per-task gate, not the method's only gate: the final
whole-branch review is unconditional and sees every exempted task's diff for
the first time. When the line count or the contract boundary is unclear,
dispatch the reviewer — the exemption is for the unambiguous small case, not
a default to reach for. Full protocol: `helia:executing-plans`.

## The failure matrix, row by row

| What happened | Direction |
|---|---|
| Technical unknown | **down** — probe, bounded. The plan does not move. |
| Execution error (red test, broken build) | **down** — two attempts, bounded by the attempt budget. |
| Unplanned dependency, task too large | **up** — finish the independents, then re-plan. |
| Ambiguous or contradictory spec | **up** — stop now, back to phase 2 with the human. |
| A behaviour decision is missing, current work not wrong | **out** — a derived need (own directory, spec, cycle) if this cycle needs the answer to continue; otherwise a candidate, written as a note in `<project>/_notes/`. |

Going down is cheap, going up is expensive — but retrying a wrong plan is the
most expensive thing of all. Never spend an attempt budget on a failure that
belongs upward. **The attempt budget is two, counted in `vault/<project>/<feature>/plan.md`, not
in your head.** When it's spent, stop and ask a human partner for
arbitration — there is no third attempt and no automatic model escalation,
and an effort escalation does not buy one either.

Full protocol and cycle report: `helia:routing-failures`.
