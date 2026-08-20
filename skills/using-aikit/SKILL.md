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
reads its registry file, and routes. Nothing runs before it.

## The Phases

| # | Phase | Skill | Produces |
|---|---|---|---|
| 1 | Understand the need | `aikit:understanding-need` | the project, the route, the feature directory |
| 2 | Specify | `aikit:brainstorming` then `aikit:writing-specs` | `work/<project>/<feature>/spec.md` |
| 3 | Plan | `aikit:writing-plans` | `work/<project>/<feature>/plan.md` |
| 4 | Split into tasks | `aikit:writing-plans` | tasks, each declaring its files |
| 5 | Execute | `aikit:subagent-driven-development` | code, and `sdd/` next to the plan |
| 6 | Verify | `aikit:verification-before-completion` | the project's completion criterion, met |

Two cross-cutting skills: `aikit:checking-plan-drift` after every task, and
`aikit:handling-blockers` whenever something fails.

Phases 1 and 2 need the human. They are never delegated to a subagent — a
subagent cannot ask a question, so a delegated spec is an invented spec.

**The artifact is the memory, not the conversation.** Each phase ends with a
file. Never chain two phases in one context hoping to remember the first.

**aiKit writes nothing inside the project repositories** except the code
changes themselves. Specs, plans, ledgers and scratch all live under `work/`
at the workspace root.

## The archetypes, and their fixed models

| Role | Model |
|---|---|
| `aikit:planner`, `aikit:reviewer`, `aikit:spike` | **fable** — evaluative work: planning, judging a diff, answering a question |
| `aikit:implementer` | **opus** — production work, where wrong output costs most to undo |
| `aikit:explorer`, `aikit:verifier` | sonnet |

`aikit:reviewer` is always a fresh instance — never the one that wrote the code.

When a plan task names a project's domain expert (`Agent: next_expert`),
dispatch that one instead of the generic implementer — **and check its model,
overriding to opus for implementation work.** Domain experts carry their own
tier and it is not always the right one. Domain knowledge and model tier are
separate choices.

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
| "This is just a simple question" | Questions are tasks. Check for skills. |
| "I need more context first" | Skill check comes BEFORE clarifying questions. |
| "Let me explore the codebase first" | Skills tell you HOW to explore. Check first. |
| "This doesn't need a formal skill" | If a skill exists, use it. |
| "I remember this skill" | Skills evolve. Read the current version. |
| "I'll just do this one thing first" | Check BEFORE doing anything. |
| "The plan is basically right" | Basically right is a spec problem. Go up. |
| "I'll widen the scope slightly" | That is drift. Report it instead. |

## Precedence

User instructions (CLAUDE.md, AGENTS.md, direct requests) take precedence over
skills, which override default behavior. Only skip a skill workflow when your
human partner has explicitly told you to.
