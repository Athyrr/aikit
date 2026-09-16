---
name: using-aikit
description: Use when starting any conversation - establishes the aiKit method, its phases, where artifacts live, and how to react when a task fails
---

<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, ignore this skill.
Do the task you were given, report, stop.
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
If you think there is even a 1% chance a skill might apply to what you are doing, you ABSOLUTELY MUST invoke the skill.

IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.

This is not negotiable. You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>

## The Rule

**Invoke relevant skills BEFORE any response or action** — including clarifying
questions, exploring the codebase, or reading files. If a skill turns out wrong
for the situation, you don't have to use it.

Then announce "Using [skill] to [purpose]" and follow it exactly. If it has a
checklist, create a todo per item.

**Every request starts with `aikit:understanding-need`.** It names the project,
reads its registry file, and routes. Nothing runs before it. Where no registry
exists, the method is unchanged — its facts come from the project's own docs.

## Fast-Path vs Heavy-Path

`aikit:understanding-need` classifies every build/change/remove request into
one of two routes before phase 2 starts:

| | Fast-Path | Heavy-Path |
|---|---|---|
| When | ≤2 files, no API/contract break, no critical dependency | new component, refactor, contract change, or the estimate is wrong |
| Artifacts | none — no `spec.md`, no `plan.md` | the full chain below |
| Execution | one light dispatch, direct | phases 2 through 5 |
| Validation | `git diff --name-only` against the stated file set | `aikit:checking-plan-drift` per task |

Fast-Path is a bet, not a discount on rigor: if the diff exceeds the stated
files, or a second file turns out to need a change the first didn't predict,
that is drift — stop and re-route to Heavy-Path rather than absorbing it
silently. Getting the estimate wrong is cheap; treating the wrong estimate as
right is not.

## The Phases (Heavy-Path)

| # | Phase | Skill | Produces |
|---|---|---|---|
| 1 | Understand the need | `aikit:understanding-need` | the project, the route, the feature directory |
| 2 | Specify | `aikit:brainstorming` then `aikit:writing-specs` | `vault/<project>/<feature>/spec.md` |
| 2.5 | Impact | dispatch the project's domain expert, consultatively | an `## Impact` section appended to `spec.md` |
| 3 | Plan | `aikit:writing-plans` | `vault/<project>/<feature>/plan.md` |
| 4 | Split into tasks | `aikit:writing-plans` | tasks, each declaring its files |
| 5 | Execute | `aikit:subagent-driven-development` | code, and `sdd/` next to the plan |
| 6 | Verify | `aikit:verification-before-completion` | the project's completion criterion, met |

Phase 2.5 asks the expert what a doc cannot answer: *which files does this
spec touch, what are the traps, how would you cut it?* It writes no code, and
it writes no separate file — its answer lands as a section of `spec.md`, the
one artifact phase 2 and 2.5 share.

Cross-cutting: `aikit:loading-policy` before any dispatch or large read,
`aikit:checking-plan-drift` after every task, `aikit:handling-blockers` on any
failure, `aikit:handling-secrets` before writing config or committing anything
that touches credentials, `aikit:delegating-to-a-perimeter` when a question needs
a project's own MCP servers.

## What loads where

> **Big reads happen in contexts that get thrown away.**

A subagent reads the 27,000-token doc, returns a 300-token finding, and dies.
**You hold the plan and the state — nothing else**, around 10k.

Never open a large doc whole: the registry routes each kind of task to the
section it needs. Pass **paths, not contents**. Full table:
`aikit:loading-policy`.

Phases 1 and 2 need the human, so they are never delegated: a subagent cannot
ask a question, and a delegated spec is an invented spec.

**The artifact is the memory, not the conversation.** Each phase ends with a
file. Never chain two phases in one context hoping to remember the first.

**aiKit writes nothing inside the project repositories** except the code
changes themselves. Specs, plans, ledgers and scratch all live under the vault
that owns the project — an **Obsidian vault**: invoke `obsidian:obsidian-markdown`
before writing its syntax, `obsidian:obsidian-bases` before editing `aikit.base`.

## The archetypes, and their fixed models

| Role | Model |
|---|---|
| `aikit:planner`, `aikit:reviewer`, `aikit:spike` | **fable** — evaluative work: planning, judging a diff, answering a question |
| `aikit:implementer` | **sonnet**, escalating to **opus** only when the ledger shows 2 failed fix-loop attempts (`aikit:handling-blockers`) — never a per-task choice |
| `aikit:explorer`, `aikit:verifier` | sonnet |

`aikit:reviewer` is always a fresh instance — never the one that wrote the code.

When a task names a domain expert (`Agent: api-expert`), dispatch it instead
of the generic implementer, **overriding its model to opus** for implementation
work: experts carry their own tier and it is often lower.

## When something fails, the nature of the failure decides the direction

| What happened | Direction |
|---|---|
| Technical unknown | **down** — spike, bounded. The plan does not move. |
| Execution error (red test, broken build) | **down** — fix loop, bounded by budget. |
| Unplanned dependency, task too large | **up** — finish the independents, then re-plan. |
| Ambiguous or contradictory spec | **up** — stop now, back to phase 2 with the human. |

Going down is cheap, going up is expensive — but retrying a wrong plan is the
most expensive thing of all. Never spend a retry budget on a failure that
belongs upward. **The budget is counted in the ledger, not in your head.**

Full protocol, escalation contract and loop report: `aikit:handling-blockers`.

## Red Flags

These thoughts mean STOP — you're rationalizing:

| Thought | Reality |
|---------|---------|
| "I need more context first" | Skill check comes BEFORE clarifying questions. |
| "This doesn't need a formal skill" | If a skill exists, use it. |
| "I remember this skill" | Skills evolve. Read the current version. |
| "I'll read the file myself, it's quicker" | Quicker now, resident forever. Dispatch. |
| "The plan is basically right" | Basically right is a spec problem. Go up. |
| "I'll widen the scope slightly" | That is drift. Report it instead. |

## Precedence

User instructions (CLAUDE.md, AGENTS.md, direct requests) take precedence over
skills, which override default behavior. Only skip a skill workflow when your
human partner has explicitly told you to.
