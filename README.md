# aiKit

An agentic method for Claude Code: a plugin that turns a request into a spec, a
plan, reviewed tasks and a verified result — without the orchestrating session
ever filling up.

**aiKit is not a harness.** The harness is Claude Code — it owns the tools, the
context window, the model calls and the permission system. aiKit ships prompts,
agent definitions and one hook that the harness consumes.

Derived from [superpowers](https://github.com/obra/superpowers) (MIT, Jesse
Vincent) and detached from it: there is no merge path back, by design. Compare
and cherry-pick by hand when you feel like it.

## What it does

A SessionStart hook injects, at every startup, `/clear` and **after every
compaction**, two things:

1. a compact stub — the cardinal rules (skill-check discipline, the phase-1
   gate, the Fast-Path/Heavy-Path split) — not the full method;
2. a table of every registered vault's projects, built from `<vault>/projects/*.md`
   for each vault declared on this machine — the router.

The full method (`skills/using-aikit/SKILL.md`) loads on demand, like any
other skill, the first time a session actually needs it — the compaction
matcher on the stub is what survives context loss without re-paying for the
whole method every time. Outside a vault that carries a registry, only the
stub is injected — the routing step simply has nothing to route to.

## Fast-Path vs Heavy-Path

Phase 1 estimates size before anything else runs. **Fast-Path** — 2 files or
fewer, no API/contract break, no critical dependency — skips straight to a
direct implementer dispatch, validated with `git diff --name-only`: no spec,
no plan, no ledger. Everything bigger, or anything the estimate gets wrong
mid-flight, is **Heavy-Path**: the phase table below, in full.

## The phases (Heavy-Path)

| # | Phase | Skill | Produces |
|---|---|---|---|
| 1 | Understand the need | `aikit:understanding-need` | the project, the route, the feature directory |
| 2 | Specify | `aikit:brainstorming` → `aikit:writing-specs` | `spec.md` |
| 2.5 | Impact | the project's domain expert, consultatively | an `## Impact` section appended to `spec.md` |
| 3 | Plan | `aikit:writing-plans` | `plan.md` |
| 4 | Split into tasks | `aikit:writing-plans` | tasks, each declaring its files |
| 5 | Execute | `aikit:subagent-driven-development` | code, one task at a time |
| 6 | Verify | `aikit:verification-before-completion` | the registry's completion criterion, met |

Cross-cutting: `aikit:loading-policy` before any dispatch or large read,
`aikit:checking-plan-drift` after every task, `aikit:handling-blockers` whenever
something fails, `aikit:delegating-to-a-perimeter` when a question needs a
project's own tools, `aikit:receiving-code-review` when incorporating feedback
from outside the method's own review loop.

## Model cascading

`aikit:implementer` defaults to **sonnet**. It escalates to **opus** exactly
once per task — automatically, when `aikit:handling-blockers` finds two failed
fix-loop attempts recorded in the ledger. It is never a per-task judgement
call; see `aikit:handling-blockers` and `aikit:subagent-driven-development`.

A **diagnostic** does not run these phases. It produces a `diagnostic-N.md` and
stops; if it concludes that code must change, that finding becomes the input of
a phase 1 on the target project.

## The patterns

The phases are the shape. These are the load-bearing ideas — each one exists
because of a specific failure it prevents.

**The artifact is the memory, not the conversation.** Every phase ends with a
file. A conversation dies at compaction; `spec.md` does not. This is also why
the SessionStart hook matches `compact`: the method survives the loss it exists
to protect against.

**Big reads happen in contexts that get thrown away.** A subagent reads the
27,000-token document, returns a 300-token finding, and dies. The orchestrator
holds the plan and the state, around 10k. Pass paths, not contents; the
registry routes each task to the *section* it needs, never the whole file.

**The nature of a failure decides the direction.**

| What happened | Direction |
|---|---|
| Technical unknown | **down** — a bounded spike; the plan does not move |
| Execution error | **down** — fix loop, bounded by a budget |
| Unplanned dependency, task too large | **up** — finish the independents, re-plan |
| Ambiguous or contradictory spec | **up** — stop, back to phase 2 with the human |

Going down is cheap, going up is expensive, and retrying a wrong plan is the
most expensive of all. **The retry budget is counted in the ledger, not in
context** — a conversation that compacts forgets it is on its fourth attempt.

**Drift is checked mechanically.** After each task, the files the task declared
are confronted with `git diff --name-only`. This works *because* aiKit writes no
artifact inside a repository: every file in a diff is production code by
construction.

**The reviewer is always a fresh instance** — never the agent that wrote the
code. An author re-reads their intention, not their text.

**A perimeter is a process, not a subagent.** A subagent runs inside its
caller's session: it cannot re-scope to another directory or load a project's
skills. Only a process launched with the right working directory can — and what
that buys is not the tools but the forty tool calls landing in a context that
gets discarded.

**Phases 1 and 2 are never delegated.** A subagent cannot ask a question, and a
delegated spec is an invented spec.

## The archetypes

`aikit:explorer`, `aikit:planner`, `aikit:implementer`, `aikit:reviewer`, `aikit:verifier`, `aikit:spike` — roles in
the process, each with a fixed model: **opus** for `aikit:planner` and
`aikit:reviewer` (evaluative work where a wrong judgement is the most
expensive kind of failure), **sonnet** for `aikit:implementer` (production
work), `aikit:explorer`, `aikit:verifier` and `aikit:spike` (a spike is
bounded and low-stakes — closer to explorer's shape of work than to planning
or review). Implementation escalates to opus exactly once per
task — automatically, when the ledger shows two failed fix-loop attempts — never
a per-task judgement call; see `aikit:handling-blockers`. Opus is therefore not
escalation-only: it also stands as planner and reviewer's default on a machine
with no cheaper evaluative-tier model (this method originally ran that tier on
**fable**). A project's domain agents are the other axis: when a plan task
names one, it is dispatched instead of the generic `aikit:implementer`. The
registry says which exist.

## Where things live

The method is versioned and installed once per machine. The artifacts and the
registry are not — they belong to the vault that uses the method, never to
the method itself.

```
<anywhere>/aikit/             a clone of the method — the SOURCE, not the plugin
  hooks/                      the SessionStart injection
  skills/                     the skills
  agents/                     the six archetypes
  scripts/                    doctor and deploy — NOT on the Bash tool's PATH
  bin/                        ezy and scoped — these ARE on it

~/.config/aikit/vaults         which vaults this machine knows, and where
<vault>/                      a clone of a vault (an Obsidian vault)
  projects/                   the registry: one file per project
  <project>/<feature>/{spec,plan,ledger,diagnostic-N}.md   spec carries its own Impact section
<anywhere>/                   the project repositories, found by scanning

~/.claude/plugins/cache/aikit-marketplace/aikit/<version>/   ← WHAT ACTUALLY RUNS after task 17
```

The hook resolves projects from the vaults declared in
`~/.config/aikit/vaults` and a scan of their declared roots. So the method
travels everywhere while a registry stays bound to one vault — and the two
never live in the same repository.

**aiKit writes nothing inside the project repositories** except the code
changes themselves. No specs, no plans, no ledger, no scratch. `git status`
in a project never shows a method artifact.

## Install

On any machine, once, at user scope:

```bash
claude plugin marketplace add git@github.com:Athyrr/aikit.git   # this remote: task 15
claude plugin install aikit@aikit-marketplace --scope user      # this marketplace: task 17
```

Register the marketplace over **SSH**, and install in **one scope only** — two
scopes means two copies, and an update touches one while the other keeps
running. `--scope user` makes the method available from any repository on the
machine: it is a way of working, not vault data. **You do not clone this
repository to use the method.**

To make a vault routable, give it a registry — `<vault>/projects/<name>.md`
per project, and declare the vault in `~/.config/aikit/vaults`.
`aikit:registering-a-project` carries the frontmatter contract the
tooling parses and the six sections a registry file must hold.

**[`SETUP.md`](SETUP.md) is the full procedure** — prerequisites (Git for
Windows is a hard one), why SSH rather than HTTPS, registering a vault,
and the development loop below. That text lives there, once.

## Iterating on the method

Installing a plugin **copies** it into
`~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/`, and
`claude plugin update` compares *versions*, not content — without a bump it
reports "already at the latest version" and the stale copy keeps running.

```bash
scripts/deploy [patch|minor|major] "message"
```

from the clone bumps both manifests, runs the nine gates, commits, pushes,
refreshes the marketplace and updates the install. **Takes effect in a new
session.** Run `scripts/doctor` any time for the gates without deploying, and
`claude --plugin-dir .` to load the working tree into one session only.
[`SETUP.md`](SETUP.md) §4 has the whole loop, from clone to merge.

The registry is exempt: `<vault>/projects/*.md` lives in the vault
and is read from disk by the hook, so a registry edit is live in the next
session with no deploy.

## Differences from superpowers

- renamed throughout (`aikit:` prefix), single harness (Claude Code only)
- artifacts moved out of the repositories into `vault/`
- per-vault registry injected at session start, from declared vaults
- failure-direction rule (down = spike/fix loop, up = re-plan/re-spec)
- retry budget written to the ledger, not held in context
- multi-harness ports, CI, upstream docs and the remote brand image removed
