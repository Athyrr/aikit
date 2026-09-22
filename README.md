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
fewer, no public API or data-schema change — skips straight to a direct
implementer dispatch in the main session, validated with
`git diff --name-only`: no spec, no `vault/<project>/<feature>/plan.md`, no intermediate
documentation of any kind. Everything bigger, or anything the estimate gets
wrong mid-flight, is **Heavy-Path**: subagent-driven, the phase table below,
in full.

## The phases (Heavy-Path)

| # | Phase | Skill | Produces |
|---|---|---|---|
| 1 | Understand | `aikit:understanding-need` | the feature directory, `vault/<project>/<feature>/plan.md` initialised |
| 2 | Design | `aikit:designing-the-solution` → `aikit:writing-specs` | `spec.md` |
| 3 | Assess | the project's domain expert, consultatively | an `## Impact` section appended to `spec.md` |
| 4 | Plan | `aikit:writing-plans` | `vault/<project>/<feature>/plan.md`, split into tasks that each declare their files |
| 5 | Execute | `aikit:executing-plans` | code, one task at a time, `vault/<project>/<feature>/plan.md`'s checkboxes ticked |
| 6 | Verify | `aikit:verifying-completion` | the registry's completion criterion, met; `vault/<project>/<feature>/plan.md` closed |

Cross-cutting: `aikit:budgeting-context` before any dispatch or large read,
`aikit:checking-plan-drift` after every task, `aikit:routing-failures` whenever
something fails, `aikit:delegating-to-a-perimeter` when a question needs a
project's own tools, `aikit:receiving-code-review` when incorporating feedback
from outside the method's own review loop.

## Model selection

Every role runs on sonnet by default, and none escalates automatically.
`aikit:implementer`, `aikit:probe`, `aikit:explorer` and `aikit:verifier` are
fixed there with no escalation at all. `aikit:planner` and `aikit:reviewer`
also default to sonnet, each with its own manual-only escalation to opus: the
planner for a complex distributed-architecture redesign at the human
partner's explicit request, the reviewer for a security-sensitive audit or
two consecutive fix attempts that still fail review. When a task's second
attempt also fails, `aikit:routing-failures` stops the loop and asks the
human partner for arbitration instead of trying a third time or reaching for
a bigger model.

A **diagnostic** does not run these phases. `aikit:diagnosing` opens
`diagnosis.md` before investigating and updates it after every hypothesis —
what's ruled out, what's still open, the root cause once known — so
eliminations survive a reset instead of being re-tested. It becomes a feature
need, entering phase 1, only once that root cause is known: a fix designed
from a symptom is a guess with a plan attached.

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
| Technical unknown | **down** — a bounded probe; the plan does not move |
| Execution error | **down** — two attempts, bounded by the attempt budget |
| Unplanned dependency, task too large | **up** — finish the independents, re-plan |
| Ambiguous or contradictory spec | **up** — stop, back to phase 2 with the human |
| A behaviour decision is missing, current work isn't wrong | **out** — a derived need, its own directory and cycle |

Going down is cheap, going up is expensive, and going out costs a full cycle;
retrying a wrong plan is still the most expensive of all. **The attempt budget
is two, counted in `vault/<project>/<feature>/plan.md`, not in context** — a conversation that
compacts forgets it is on its second attempt. When it's spent, the method
stops and asks the human partner to rule, rather than escalating rounds or
models on its own.

**Drift is checked mechanically.** After each task, the files the task declared
are confronted with `git diff --name-only`. This works *because* aiKit writes no
artifact inside a repository: every file in a diff is production code by
construction.

**The reviewer is always a fresh instance** — never the agent that wrote the
code. An author re-reads their intention, not their text. Its decision is
binary: `REJECT` only for a functional bug, a spec/task violation, or a
security regression — everything else (style, naming, structure) becomes a
non-blocking suggestion, and still `APPROVE`s.

**Two redundant reviews were removed.** The plan no longer faces a dedicated
plan-reviewer subagent before handoff — its self-review stays, but validation
is the human partner's, directly. And a task whose tests pass, whose diff is
≤30 lines, and that touches no public API or shared contract skips its
per-task reviewer dispatch entirely; the final whole-branch review is what
still catches it, unconditionally.

**Dispatch in, report out — both hermetic.** The orchestrator's dispatch
prompt carries four things and nothing else: the task's identity in
`plan.md`, its file paths, its test command, at most one constraint. No
pasted code, no session history. The implementer's reply back is three
lines — `STATUS` / `FILES` / `TEST` — nothing else re-enters the
orchestrator's context. There is no separate brief or report file; `plan.md`
itself is the record.

**Two attempts, then a human, never a bigger model automatically.** A task
that still fails review after its second attempt stops the loop — no
K-of-N escalation, no automatic model bump. The orchestrator asks, states
what it tried, and waits.

**A trap earns a line only once it's real.** `heuristics.md` (per-project
and machine-global, capped at 50 lines each) is read in cascade before
planning and written by the orchestrator alone, only at close, only for a
trap this cycle actually hit — never a restatement of what was already
known.

**A perimeter is a process, not a subagent.** A subagent runs inside its
caller's session: it cannot re-scope to another directory or load a project's
skills. Only a process launched with the right working directory can — and what
that buys is not the tools but the forty tool calls landing in a context that
gets discarded.

**Phases 1 and 2 are never delegated.** A subagent cannot ask a question, and a
delegated spec is an invented spec.

**Phase 2's interview works one frontier at a time.** `aikit:grilling` asks
every question whose prerequisites are already settled, recommends an answer
to each, and waits — a question that depends on one still open belongs to a
later pass, not this one.

**A brief can be wrong about a fact; it can't be wrong about a requirement.**
When an implementer's brief contradicts what the code actually does, the code
wins — but that yields only to a stated fact, never to scope creep, and it
must be reported explicitly, not corrected silently.

## The archetypes

`aikit:explorer`, `aikit:planner`, `aikit:implementer`, `aikit:reviewer`,
`aikit:verifier`, `aikit:probe` — roles in the process, each on **sonnet** by
default and no automatic escalation between models. `aikit:planner` and
`aikit:reviewer` are the two a human partner can escalate to opus by hand:
the planner for a complex distributed-architecture redesign at their
explicit request, the reviewer for a security-sensitive audit or after two
consecutive fix attempts still fail review. Two failed attempts at a task
stop the loop and hand the decision to the human partner instead of buying a
bigger model; see `aikit:routing-failures`. A project's domain agents are the
other axis: when a plan task names one, it is dispatched instead of the
generic `aikit:implementer`. The registry says which exist.

## Where things live

The method is versioned and installed once per machine. The artifacts and the
registry are not — they belong to the vault that uses the method, never to
the method itself.

```
<anywhere>/aikit/             a clone of the method — the SOURCE, not the plugin
  hooks/                      the SessionStart injection
  skills/                     the skills
  agents/                     the six archetypes
  VOCABULARY.md               one word, one meaning — read before naming anything
  scripts/                    doctor and deploy — NOT on the Bash tool's PATH
  bin/                        ezy and scoped — these ARE on it

~/.config/aikit/vaults         which vaults this machine knows, and where
<vault>/                      a clone of a vault (an Obsidian vault)
  projects/                   the registry: one file per project
  <project>/candidates.md     real problems nothing is waiting on — survives any one need
  <project>/heuristics.md     empirical rules and traps for this project — orchestrator-written, at /finish only
  <project>/<feature>/        spec.md, plan.md, diagnosis.md, probe findings — spec carries its own Impact section
  _global/heuristics.md       cross-project environment, OS and shell rules
<anywhere>/                   the project repositories, found by scanning

~/.claude/plugins/cache/aikit-marketplace/aikit/<version>/   ← WHAT ACTUALLY RUNS
```

The hook resolves projects from the vaults declared in
`~/.config/aikit/vaults` and a scan of their declared roots. So the method
travels everywhere while a registry stays bound to one vault — and the two
never live in the same repository.

**aiKit writes nothing inside the project repositories** except the code
changes themselves. No specs, no plan, no ledger, no scratch. `git status`
in a project never shows a method artifact.

## Install

On any machine, once, at user scope:

```bash
claude plugin marketplace add git@github.com:Athyrr/aikit.git
claude plugin install aikit@aikit-marketplace --scope user
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

from the clone bumps both manifests, runs the ten gates, commits, pushes,
refreshes the marketplace and updates the install. **Takes effect in a new
session.** Run `scripts/doctor` any time for the gates without deploying, and
`claude --plugin-dir .` to load the working tree into one session only.
[`SETUP.md`](SETUP.md) §4 has the whole loop, from clone to merge.

The registry is exempt: `<vault>/projects/*.md` lives in the vault
and is read from disk by the hook, so a registry edit is live in the next
session with no deploy.

## Differences from superpowers

- renamed throughout (`aikit:` prefix), single harness (Claude Code only)
- artifacts moved out of the repositories into `vault/`; plan, status and
  ledger consolidated into one file, `vault/<project>/<feature>/plan.md`,
  instead of three
- per-vault registry injected at session start, from declared vaults
- failure-direction rule (down = probe/two attempts, up = re-plan/re-spec, out = derived need)
- attempt budget (two) written to `vault/<project>/<feature>/plan.md`, not held in context —
  spent, it stops and asks a human partner rather than escalating on its own
- multi-harness ports, CI, upstream docs and the remote brand image removed
