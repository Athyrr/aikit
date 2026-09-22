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

**Every distinct need starts with `aikit:understanding-need`** — once per need,
not once per message: answering clarifying questions inside a need already
routed does not re-invoke it. It names the project, reads its registry, routes.

Route first: `aikit:understanding-need` splits every request into **Fast-Path**
(≤2 files, no contract break — no spec, no plan, one dispatch, proven by
`git diff --name-only`) or **Heavy-Path** (everything else, or a wrong
estimate — the phases below, drift-checked per task). Tables: `reference.md`.

## The Phases (Heavy-Path)

> **A step is a phase only if it ends in a written artifact.**

| # | Phase | Skill | Artifact |
|---|---|---|---|
| 1 | Understand | `aikit:understanding-need` | the need's directory, `status.md` initialised |
| 2 | Design | `aikit:designing-the-solution` then `aikit:writing-specs` | `spec.md` |
| 3 | Assess | the domain expert, consultatively | the `## Impact` section of `spec.md` |
| 4 | Plan | `aikit:writing-plans` | `plan.md` |
| 5 | Execute | `aikit:executing-plans` | the code, and `runs/<plan>/` |
| 6 | Verify | `aikit:verifying-completion` | the verdict, `status.md` closed, candidates harvested |

Cross-cutting: `aikit:budgeting-context` before any dispatch or large read,
`aikit:checking-plan-drift` after every task, `aikit:routing-failures` on any
failure, `aikit:handling-secrets` before writing config or committing anything
that touches credentials, `aikit:delegating-to-a-perimeter` when a question needs
a project's own MCP servers, `aikit:receiving-code-review` when incorporating
feedback from outside the method's own review loop.

## What loads where

> **Big reads happen in contexts that get thrown away.**

A subagent reads the 27,000-token doc, returns a 300-token finding, and dies.
**You hold the plan and the state — nothing else**, around 10k.

Never open a large doc whole: the registry routes each kind of task to the
section it needs. Pass **paths, not contents**. Full table:
`aikit:budgeting-context`.

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
| `aikit:planner`, `aikit:reviewer` | **opus** — evaluative work where a wrong judgement is the most expensive kind of failure: planning, judging a diff |
| `aikit:probe` | **sonnet**, escalating to **opus** after 2 recorded attempt failures — bounded, low-stakes investigation; closer to explorer's shape of work than to planning or review |
| `aikit:implementer` | **sonnet**, escalating to **opus** only when the ledger shows 2 failed fix-round attempts (`aikit:routing-failures`) — never a per-task choice |
| `aikit:explorer`, `aikit:verifier` | sonnet |

Rationale per role: `reference.md`, next to this skill.

`aikit:reviewer` is always a fresh instance — never the one that wrote the code.

When a task names a domain expert (`Agent: api-expert`), dispatch it instead
of the generic implementer, **overriding its model to opus** for implementation
work: experts carry their own tier and it is often lower.

## When something fails, the nature of the failure decides the direction

**Down**, plan unmoved: technical unknown → bounded probe; red test or broken
build → fix rounds, bounded by the attempt budget. **Up**: unplanned dependency, a file
outside the task's declared set, or a task too large → finish the independents,
then re-plan; spec ambiguous or contradictory → stop, back to phase 2 with the
human. **Out**: a missing behaviour decision, current work not wrong → a derived
need (own directory, spec, cycle) if this cycle needs the answer to continue, else a candidate. Going up is expensive, but
retrying a wrong plan is the most expensive of all. **The attempt budget lives
in the ledger.** Protocol: `aikit:routing-failures`.

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
