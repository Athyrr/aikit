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

## First: name the project

Every request targets one project of the workspace. Identify it, then read its
registry file before anything else — it holds the base branch, the domain
agents to dispatch, the completion criterion, and the convention files to load.
**Never work on a project whose registry file you have not read.**

## The Phases

| # | Phase | Skill | Produces |
|---|---|---|---|
| 1 | Understand the need | `aikit:brainstorming` | shared understanding (dialogue with the human) |
| 2 | Specify | `aikit:brainstorming` | `work/<project>/<feature>/spec.md` |
| 3 | Plan | `aikit:writing-plans` | `work/<project>/<feature>/plan.md` |
| 4 | Split into tasks | `aikit:writing-plans` | tasks inside the plan, each declaring its files |
| 5 | Execute | `aikit:subagent-driven-development` | code, and `sdd/` next to the plan |
| 6 | Verify | `aikit:verification-before-completion` | the project's completion criterion, met |

Phases 1 and 2 need the human. They are never delegated to a subagent — a
subagent cannot ask you a question, so a delegated spec is an invented spec.

**The artifact is the memory, not the conversation.** Each phase ends with a
file. Never chain two phases in one context hoping to remember the first.

**aiKit writes nothing inside the project repositories** except the code
changes themselves. Specs, plans, ledgers and scratch all live under
`work/` at the workspace root.

## When something fails, the nature of the failure decides the direction

| What happened | Direction | What to do |
|---|---|---|
| Technical unknown ("I don't know how X works") | **down** | Bounded investigation, minimal sub-context. The plan does not move. |
| Execution error (red test, broken build) | **down** | Fix loop, bounded by a retry budget. |
| Unplanned dependency, task too large | **up** | Finish the independent tasks, then re-split or re-plan. |
| Ambiguous or contradictory spec | **up** | Stop immediately. Back to phase 2, with the human. |

Going down is cheap, going up is expensive — but retrying a wrong plan is the
most expensive thing of all. Never spend a retry budget on a failure that
belongs upward.

**The retry budget is counted in the ledger, not in your head.** An agent that
retries does not reliably remember it is on attempt three; compaction erases
that first. A budget that is not written down is not a budget.

## Red Flags

These thoughts mean STOP — you're rationalizing:

| Thought | Reality |
|---------|---------|
| "This is just a simple question" | Questions are tasks. Check for skills. |
| "I need more context first" | Skill check comes BEFORE clarifying questions. |
| "Let me explore the codebase first" | Skills tell you HOW to explore. Check first. |
| "Let me gather information first" | Skills tell you HOW to gather information. |
| "This doesn't need a formal skill" | If a skill exists, use it. |
| "I remember this skill" | Skills evolve. Read the current version. |
| "The skill is overkill" | Simple things become complex. Use it. |
| "I'll just do this one thing first" | Check BEFORE doing anything. |
| "I know what that means" | Knowing the concept ≠ using the skill. Invoke it. |
| "The plan is basically right" | Basically right is a spec problem. Go up. |

## Precedence

User instructions (CLAUDE.md, AGENTS.md, direct requests) take precedence over
skills, which override default behavior. Only skip a skill workflow when your
human partner has explicitly told you to.
