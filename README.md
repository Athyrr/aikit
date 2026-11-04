# Helia

An agentic method for Claude Code: a plugin that turns a request into a spec, a
plan, reviewed tasks and a verified result — without the orchestrating session
ever filling up.

**Helia is not a harness.** The harness is Claude Code — it owns the tools, the
context window, the model calls and the permission system. Helia ships prompts,
agent definitions and two hooks (`session-start`, `appearance`) that the harness
consumes.

Derived from [superpowers](https://github.com/obra/superpowers) (MIT, Jesse
Vincent) and detached from it: there is no merge path back, by design. Compare
and cherry-pick by hand when you feel like it.

## Why Helia

*Helia* comes from *Helianthus*, the sunflower — Greek *hēlios*, the sun, and
*anthos*, the flower. *Umbra* is its counterpart, the shadow facing that light.
The two themes the plugin ships come from that pair: **Helia**, a warm
dark-brown base with the sunflower's gold, amber and leaf green; **Umbra**, an
eclipse — cold black, a pale golden corona, bluish steel.

## What it does

A SessionStart hook injects, at every startup, `/clear` and **after every
compaction**, two things:

1. a compact stub — the cardinal rules (skill-check discipline, the phase-1
   gate, the Fast-Path/Heavy-Path split) — not the full method;
2. a table of every registered vault's projects, built from `<vault>/projects/*.md`
   for each vault declared on this machine — the router.

A second hook, in a block of its own, injects each declared vault's
`_global/consignes.md` — the instructions your human partner gave for every
project — at the same moments and on `resume` and `fork`, capped at 9,000 bytes for all
vaults together; a `SubagentStart` hook gives the same block to every
`helia:*` agent.

The full method (`skills/using-helia/SKILL.md`) loads on demand, like any
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
| 1 | Understand | `helia:understanding-need` | the feature directory, `vault/<project>/<feature>/plan.md` initialised |
| 2 | Design | `helia:designing-the-solution` → `helia:writing-specs` | `spec.md` |
| 3 | Assess | the project's domain expert, consultatively | an `## Impact` section appended to `spec.md` |
| 4 | Plan | `helia:writing-plans` | `vault/<project>/<feature>/plan.md`, split into tasks that each declare their files |
| 5 | Execute | `helia:executing-plans` | code, one task at a time, `vault/<project>/<feature>/plan.md`'s checkboxes ticked |
| 6 | Verify | `helia:verifying-completion` | the registry's completion criterion, met; `vault/<project>/<feature>/plan.md` closed |

Cross-cutting: `helia:budgeting-context` before any dispatch or large read,
`helia:checking-plan-drift` after every task, `helia:checking-dependencies` when
`helia-deps` prints a `!` line on a plan that has tasks, `helia:routing-failures` whenever
something fails, `helia:handling-secrets` before writing config or committing
anything that touches credentials, `helia:delegating-to-a-perimeter` when a question needs a
project's own tools, `helia:receiving-code-review` when incorporating feedback
from outside the method's own review loop.

## Model selection

Each archetype declares its model and effort in its own frontmatter
(`agents/*.md`); `helia:using-helia` mirrors them in one table, which
`scripts/doctor` keeps equal to the frontmatter. Two rules govern any change.
**Effort rises automatically** on a mechanical signal — a final whole-branch
review, a task marked `Security: yes`, a re-review after a second fix —
passed at dispatch and recorded in the event log. **The model never changes
without the human partner**: only a task's `Model: opus`, confirmed for that
task, does it. A project's domain expert is not a Helia archetype and is
always dispatched on opus at effort `high`. When a task's second attempt also
fails, `helia:routing-failures` stops the loop and asks the human partner for
arbitration instead of trying a third time, whatever the model or effort.

A **diagnostic** does not run these phases. `helia:diagnosing` opens
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
are confronted with `git diff --name-only`. This works *because* Helia writes no
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
K-of-N escalation, no automatic model bump; an effort escalation, the only
automatic one, never buys a third attempt. The orchestrator asks, states
what it tried, and waits.

**A trap earns a line only once it's real.** `heuristics.md` (per-project
and machine-global, capped at 50 lines each) is read in cascade before
planning and written by the orchestrator alone, only at close, only for a
trap this cycle actually hit — never a restatement of what was already
known. An instruction your human partner gives is not a trap: it goes to the vault the moment it is given — `<project>/heuristics.md` for one project, `_global/consignes.md` for all of them — never only into the harness memory.

**A perimeter is a process, not a subagent.** A subagent runs inside its
caller's session: it cannot re-scope to another directory or load a project's
skills. Only a process launched with the right working directory can — and what
that buys is not the tools but the forty tool calls landing in a context that
gets discarded.

**Phases 1 and 2 are never delegated.** A subagent cannot ask a question, and a
delegated spec is an invented spec.

**Phase 2's interview works one frontier at a time.** `helia:grilling` asks
every question whose prerequisites are already settled, recommends an answer
to each, and waits — a question that depends on one still open belongs to a
later pass, not this one.

**A brief can be wrong about a fact; it can't be wrong about a requirement.**
When an implementer's brief contradicts what the code actually does, the code
wins — but that yields only to a stated fact, never to scope creep, and it
must be reported explicitly, not corrected silently.

## The archetypes

`helia:explorer`, `helia:planner`, `helia:implementer`, `helia:reviewer`,
`helia:verifier`, `helia:probe` — roles in the process, each with its own
model and effort (see Model selection above). Effort rises automatically on
a mechanical signal; the model changes only by the human partner's hand. Two
failed attempts at a task stop the loop and hand the decision to the human
partner instead of buying a bigger model; see `helia:routing-failures`. A
project's domain agents are the other axis: when a plan task names one, it is
dispatched instead of the generic `helia:implementer`. The registry says which
exist.

## Where things live

The method is versioned and installed once per machine. The artifacts and the
registry are not — they belong to the vault that uses the method, never to
the method itself.

```
<anywhere>/helia/             a clone of the method — the SOURCE, not the plugin
  hooks/                      the SessionStart and SubagentStart injections
  palettes/                   the colours of the two themes, by role — their source of truth
  themes/                     Helia and Umbra, generated from palettes/ by scripts/build-themes
  skills/                     the skills
  agents/                     the six archetypes
  VOCABULARY.md               one word, one meaning — read before naming anything
  scripts/                    doctor, deploy and their test suites — NOT on the Bash tool's PATH
  bin/                        ezy, scoped and helia-deps — these ARE on it

~/.config/helia/vaults         which vaults this machine knows, and where
<vault>/                      a clone of a vault (an Obsidian vault)
  projects/                   the registry: one file per project
  _global/heuristics.md       technical traps that hold for every project — machine, OS, shell
  _global/consignes.md        the human partner's instructions for every project — injected at session start
  _notes/                     notes whose idea would be another project
  <project>/heuristics.md     this project's traps (phase 6) and the human partner's instructions for it (when given)
  <project>/trame.md          how the project splits into tranches, if it does — VOCABULARY.md, The trame
  <project>/_notes/           one note per file — real problems nothing is waiting on, ideas to study later
  <project>/_archive/<need>/  finished or abandoned needs: spec.md, traces/
  <project>/_reference/       reference documents kept on request (helia:capturing-reference), routed by an INDEX.md
  _global/_reference/         references common to several projects, no ecosystem
  <project>/<need>/           the need in flight: spec.md, plan.md, diagnosis.md, etude.md — spec carries its own Impact section
<anywhere>/                   the project repositories, found by scanning

~/.claude/plugins/cache/athyrr-marketplace/helia/<version>/   ← WHAT ACTUALLY RUNS
```

The hook resolves projects from the vaults declared in
`~/.config/helia/vaults` and a scan of their declared roots. So the method
travels everywhere while a registry stays bound to one vault — and the two
never live in the same repository.

**Helia writes nothing inside the project repositories** except the code
changes themselves. No specs, no plan, no ledger, no scratch. `git status`
in a project never shows a method artifact.

## Install

On any machine, once, at user scope:

```bash
claude plugin marketplace add Athyrr/helia
claude plugin install helia@athyrr-marketplace --scope user
```

Install in **one scope only** — two
scopes means two copies, and an update touches one while the other keeps
running. `--scope user` makes the method available from any repository on the
machine: it is a way of working, not vault data. **You do not clone this
repository to use the method.**

To make a vault routable, give it a registry — `<vault>/projects/<name>.md`
per project, and declare the vault in `~/.config/helia/vaults`.
`helia:registering-a-project` carries the frontmatter contract the
tooling parses and the six sections a registry file must hold (an ecosystem carries four).

**[`SETUP.md`](SETUP.md) is the full procedure** — prerequisites (Git for
Windows is a hard one), registering a vault,
and the development loop below. That text lives there, once.

### Migrating from aiKit

Up to 0.27.0 the plugin was `aikit@aikit-marketplace`. To move an existing install:

```bash
claude plugin marketplace remove aikit-marketplace
claude plugin marketplace add Athyrr/helia
claude plugin install helia@athyrr-marketplace --scope user
```

Then `/reload-plugins`. Old `AIKIT_*` variables and `~/.config/aikit/` are still read for now.

## Appearance

The plugin ships two dark themes, **Helia** and **Umbra** (`themes/`, generated
from `palettes/` by `scripts/build-themes` — edit the palette, never the theme;
gate 14 refuses a theme that no longer matches its palette). `/theme` lists them.

At startup, a `SessionStart` hook (`hooks/appearance`) sets `theme` in
`~/.claude/settings.json` to `custom:helia:helia`, unless it already names
Helia or Umbra: any other theme is replaced. Claude Code reads the theme once
per session, so the hook says `thème Helia à la prochaine session` and the next
session opens in Helia. A `theme` set in a project's or local settings wins
over the user's.

The status line (`bin/helia-statusline`) draws two lines in the exact colours of
the active theme's palette — Umbra under `custom:helia:umbra`, Helia under
anything else:

```
✺ helia  │  <model>  │  <project> ⎇ <branch>
ctx ━━━━──────── 30 %    5h ━━━━━━━───── 60 % ↺ 2h14    hebdo ━━━━──────── 32 % ↺ 3j
```

After a `/theme`, it changes colour at the next message. At startup the hook
also shows the HELIA logotype, with the plugin version, the theme's state and
the first declared vault that has a `projects/` folder.

Opt-outs, one file each; the hook then leaves that part alone:

- `~/.config/helia/no-statusline` — the status line
- `~/.config/helia/no-theme` — the theme
- `~/.config/helia/no-banner` — the startup banner

Uninstalling the plugin leaves `theme` pointing at a theme that no longer
exists, and `statusLine` at a script that no longer exists: pick another theme
with `/theme`, and delete `statusLine` from `~/.claude/settings.json`.

## Iterating on the method

Installing a plugin **copies** it into
`~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/`, and
`claude plugin update` compares *versions*, not content — without a bump it
reports "already at the latest version" and the stale copy keeps running.

```bash
scripts/deploy [patch|minor|major] "message"
```

from the clone bumps both manifests, runs every gate of `scripts/doctor`, commits, pushes,
refreshes the marketplace and updates the install. **Takes effect in a new
session.** Run `scripts/doctor` any time for the gates without deploying, and
`claude --plugin-dir .` to load the working tree into one session only.
[`SETUP.md`](SETUP.md) §4 has the whole loop, from clone to merge.

The registry is exempt: `<vault>/projects/*.md` lives in the vault
and is read from disk by the hook, so a registry edit is live in the next
session with no deploy.

## Differences from superpowers

- renamed throughout (`helia:` prefix), single harness (Claude Code only)
- artifacts moved out of the repositories into `vault/`; plan, status and
  ledger consolidated into one file, `vault/<project>/<feature>/plan.md`,
  instead of three
- per-vault registry injected at session start, from declared vaults
- the human partner's instructions for every project (`_global/consignes.md`) injected at session start and into every `helia:*` agent
- failure-direction rule (down = probe/two attempts, up = re-plan/re-spec, out = derived need)
- attempt budget (two) written to `vault/<project>/<feature>/plan.md`, not held in context —
  spent, it stops and asks a human partner rather than escalating on its own
- multi-harness ports, CI, upstream docs and the remote brand image removed
