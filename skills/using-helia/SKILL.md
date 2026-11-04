---
name: using-helia
description: Use when starting any conversation - establishes the Helia method, its phases, where artifacts live, and how to react when a task fails
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

**Every distinct need starts with `helia:understanding-need`** — once per need,
not once per message: answering clarifying questions inside a need already
routed does not re-invoke it. It names the project, reads its registry, routes.

## Execution routing — mandatory

- **Fast-Path** (Session principale directe, sans `vault/<project>/<feature>/plan.md`, sans
  sous-agent dédié) — strict condition: **≤2 files changed, and no public
  API / data-schema change.** Action: implement and run a targeted test
  immediately. Generating any intermediate documentation is forbidden.
- **Heavy-Path** (Subagent-Driven Development) — condition: multi-file
  refactors, new features, or an API/contract break. Action: the minimal
  spec, broken into micro-tasks, delegated to disposable subagents — the
  full phase table below.

`helia:understanding-need` makes this call and proves it: Fast-Path with
`git diff --name-only` against the files it named; Heavy-Path drift-checked
per task. Tables: `reference.md`.

## The Phases (Heavy-Path)

> **A step is a phase only if it ends in a written artifact.**

| # | Phase | Skill | Artifact |
|---|---|---|---|
| 1 | Understand | `helia:understanding-need` | the need's directory, `vault/<project>/<feature>/plan.md` initialised |
| 2 | Design | `helia:designing-the-solution` then `helia:writing-specs` | `spec.md` |
| 3 | Assess | the domain expert, consultatively | the `## Impact` section of `spec.md` |
| 4 | Plan | `helia:writing-plans` | `vault/<project>/<feature>/plan.md` filled in with tasks |
| 5 | Execute | `helia:executing-plans` | the code, and `vault/<project>/<feature>/plan.md`'s checkboxes |
| 6 | Verify | `helia:verifying-completion` | the verdict, `vault/<project>/<feature>/plan.md` closed, candidates harvested |

Cross-cutting: `helia:budgeting-context` before any dispatch or large read,
`helia:checking-plan-drift` after every task, `helia:checking-dependencies` when
`helia-deps`, run in phase 1 and by the orchestrator before dispatching
`helia:planner` (`helia:writing-plans`), reports a change on a plan with tasks,
`helia:routing-failures` on any failure, `helia:handling-secrets` before writing config or committing anything
that touches credentials, `helia:delegating-to-a-perimeter` when a question needs
a project's own MCP servers, `helia:receiving-code-review` when incorporating
feedback from outside the method's own review loop. `helia:writing-notes` only when the human partner explicitly asks to
set an idea aside, `helia:capturing-reference` only when they explicitly ask to keep a document as a reference.

**Heuristics** — two vault files, both capped at 50 lines, read in cascade
(`_global/` then `<project>/`) while writing the plan:
`vault/_global/heuristics.md` (technical traps that do not depend on the
project: machine, OS, shell, or a cross-project pattern) and
`vault/<project>/heuristics.md` (this project's empirical traps). A **trap**
is written only by the orchestrator at phase 6, and only when a cycle hit a
real, unexpected one.

## Where an instruction lives

An instruction your human partner gives is written to the vault the moment it
is given — at any phase, not only phase 6 — never only to the harness memory:

| The instruction concerns | It goes to |
|---|---|
| one project | `<project>/heuristics.md` |
| every project | `_global/consignes.md` |
| the method itself | a note |
| a personal taste of your human partner | the harness memory |

The harness memory keeps only personal tastes and pointers to the vault. With
no vault reachable it is a buffer that names the destination: move the entry
as soon as the vault is back.

**Precedence: the most specific wins** — the repository's `CLAUDE.md` >
`<project>/heuristics.md` > `_global/consignes.md`.

**The two `_global/` files split by origin.** `_global/heuristics.md` holds
technical traps the method learned; `_global/consignes.md` holds instructions
your human partner gave for every project, injected at every session start
(capped at 4,000 bytes per vault). An instruction Helia has absorbed, or that
became specific to one project, leaves `consignes.md`; the review happens at
the Memory step of phase 6 (`helia:verifying-completion`).

## What loads where

> **Big reads happen in contexts that get thrown away.**

A subagent reads the 27,000-token doc, returns a 300-token finding, and dies.
**You hold the plan and the state — nothing else**, around 10k.

Never open a large doc whole: the registry routes each kind of task to the
section it needs. Pass **paths, not contents**. Full table:
`helia:budgeting-context`.

Phases 1 and 2 need the human, so they are never delegated: a subagent cannot
ask a question, and a delegated spec is an invented spec.

**The artifact is the memory, not the conversation.** Each phase ends with a
file. Never chain two phases in one context hoping to remember the first.

**Helia writes nothing inside the project repositories** except the code
changes themselves. The plan, its live status, and checkbox task tracking
all live in one per-feature file, `vault/<project>/<feature>/plan.md`,
removed when the feature closes. Specs, probe findings and scratch live
alongside it — the vault that owns the project is an **Obsidian vault**:
invoke `obsidian:obsidian-markdown` before writing its syntax,
`obsidian:obsidian-bases` before editing `helia.base`.

## The archetypes: model and effort

This table is a **mirror** of the `model:` and `effort:` frontmatter of
`agents/*.md`, and only those files run: `scripts/doctor` (gate 12) fails
when the two differ. Rationale per role: `reference.md`, next to this skill.

| Archetype | Model | Effort |
|---|---|---|
| `helia:explorer` | `sonnet` | `medium` |
| `helia:implementer` | `sonnet` | `medium` |
| `helia:probe` | `sonnet` | `medium` |
| `helia:verifier` | `sonnet` | `medium` |
| `helia:planner` | `opus` | `high` |
| `helia:reviewer` | `opus` | `high` |

**Effort rises automatically on a mechanical signal; the model never changes
without your human partner.** The signals, and what each passes at dispatch,
live in one table only: `helia:executing-plans`, Model and Effort Selection.
Two failed attempts still stop the loop (`helia:routing-failures`).

`helia:reviewer` is always a fresh instance — never the one that wrote the code.

**Task-review exemption:** a task whose tests pass, whose diff is ≤30 lines,
and that changes no public API or shared contract skips the per-task
`helia:reviewer` dispatch — the orchestrator moves straight to the next
task. The final whole-branch review still covers it; the exemption removes
only the per-task gate, never the branch-level one. See
`helia:executing-plans`.

When a task names a domain expert (`Agent: api-expert`), dispatch it instead
of the generic implementer. Its frontmatter belongs to its project, not to
Helia: how it is dispatched is a row of the same escalation table.

**Your own model.** A Heavy-Path need confirmed in a session that does not
run on opus gets one line suggesting `/model opus` (`helia:understanding-need`,
step 4). The choice stays your human partner's; nothing blocks.

## When something fails, the nature of the failure decides the direction

**Down**, plan unmoved: technical unknown → bounded probe; red test or broken
build → two attempts, bounded by the attempt budget. **Up**: unplanned dependency, a file
outside the task's declared set, or a task too large → finish the independents,
then re-plan; spec ambiguous or contradictory → stop, back to phase 2 with the
human. **Out**: a missing behaviour decision, current work not wrong → a derived
need (own directory, spec, cycle) if this cycle needs the answer to continue, else a candidate. Going up is expensive, but
retrying a wrong plan is the most expensive of all. **The attempt budget is two,
and it lives in `vault/<project>/<feature>/plan.md`.** Protocol: `helia:routing-failures`.

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
