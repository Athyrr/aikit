# aiKit

Agentic method for the Ezytail workspace. Derived from
[superpowers](https://github.com/obra/superpowers) (MIT, Jesse Vincent),
detached from it: no upstream remote, no merges. Compare and cherry-pick by
hand when you feel like it.

## What it does

A SessionStart hook injects, at every startup, `/clear` and **after every
compaction**, two things:

1. the full text of `skills/using-aikit/SKILL.md` — the method itself;
2. a table of the workspace projects, built from `projects/*.md` — the router.

The compaction matcher is the point: the method survives context loss, which
is the failure it exists to prevent.

## The phases

| # | Phase | Skill |
|---|---|---|
| 1 | Understand the need | `aikit:understanding-need` |
| 2 | Specify | `aikit:brainstorming` → `aikit:writing-specs` |
| 3 | Plan | `aikit:writing-plans` |
| 4 | Split into tasks | `aikit:writing-plans` |
| 5 | Execute | `aikit:subagent-driven-development` |
| 6 | Verify | `aikit:verification-before-completion` |

Cross-cutting: `aikit:checking-plan-drift` after every task,
`aikit:handling-blockers` whenever something fails.

## The archetypes

`aikit:explorer`, `aikit:planner`, `aikit:implementer`, `aikit:reviewer`, `aikit:verifier`, `aikit:spike` — roles in
the process, each with a fixed model: **fable** for `aikit:planner`, `aikit:reviewer` and
`aikit:spike` (evaluative work), **opus** for `aikit:implementer` (production work), sonnet
for `aikit:explorer` and `aikit:verifier`. Implementation therefore runs at the ceiling,
which removes "retry on a stronger model" from the fix loop — see
`aikit:handling-blockers`. A project's domain agents (`next_expert`, …) are the other axis:
when a plan task names one, it is dispatched instead of the generic
`aikit:implementer`. The registry says which exist.

Phases 1 and 2 are never delegated — a subagent cannot ask the human a
question.

## Where things live

```
ezytail-workspace/
├── aikit/            this repository — the method, versioned
│   ├── hooks/        the SessionStart injection
│   ├── skills/       the skills
│   └── projects/     the registry: one file per project
├── work/             the artifacts — local, never versioned, Obsidian vault
│   └── <project>/<feature>/{spec,plan,tasks,status}.md
└── <the project repositories>
```

**aiKit writes nothing inside the project repositories** except the code
changes themselves. No specs, no plans, no ledger, no scratch. `git status`
in a project never shows a method artifact.

## Install

```bash
claude plugin marketplace add ~/ezytail-workspace/aikit --scope user
claude plugin install aikit@aikit-local --scope user
```

`--scope user` makes the method available from any repository on the machine —
it is a way of working, not workspace data. What stays workspace-bound is the
**registry**: the hook looks for `aikit/projects/*.md` by walking up from the
session's directory, so outside the workspace only the method is injected.

Install in one scope only. Two scopes means two entries, and `bin/deploy`
updates one of them while the other keeps running.

## Launch

`claude` on its own is enough for most work. The launcher exists for one
thing only — surfacing the **agents and skills of the other repositories**,
which nothing carries on its own:

```bash
aikit/bin/ezy                 # + agents and skills of every registered project
aikit/bin/ezy --only ezylive  # one project
aikit/bin/ezy --dry-run       # print the command instead of running it
```

`--add-dir` carries agents and skills; it does **not** carry that project's
`.mcp.json` or its `CLAUDE.md`. All measured. Cost is the descriptions only,
~1.4k tokens for the whole workspace.

**The toolbelt needs no flag.** It is installed at the workspace root
(`apm install Ezytail/ezyflow-tool-belt --target claude`) and `.mcp.json`
lookup walks *up* the tree, so its six MCP servers are present in any session
opened anywhere in the workspace — including inside a sub-repository like
`ezylive/ezy_live`. Its seven routing skills do not travel that way: they are
read from the session's own project root, which is why a toolbelt
investigation is worth running through `bin/scoped` (perimeter `.`).

Nothing is denied in `ezytail-workspace/.claude/settings.json`. The four
destructive natseyes tools are held back by the ordinary permission prompt in
an interactive session, and by the registry's `allow:` list in an unattended
one — see `projects/ezyflow-tool-belt.md`.

## Iterating on the method

Installing a plugin **copies** it into
`~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/`, and
`claude plugin update` compares *versions*, not content — without a bump it
reports "already at the latest version" and the stale copy keeps running.

```bash
aikit/bin/deploy [patch|minor|major] "message"
```

bumps both manifests, checks the hook still emits valid JSON, validates,
commits, resyncs and updates. **Takes effect in a new session.**

One exception: `projects/*.md` is read from the source tree by the hook, so a
registry edit is live in the next session with no deploy.

## Differences from superpowers

- renamed throughout (`aikit:` prefix), single harness (Claude Code only)
- artifacts moved out of the repositories into `work/`
- per-project registry injected at session start
- failure-direction rule (down = spike/fix loop, up = re-plan/re-spec)
- retry budget written to the ledger, not held in context
- multi-harness ports, CI, upstream docs and the remote brand image removed
