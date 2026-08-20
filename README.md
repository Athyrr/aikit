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

`explorer`, `planner`, `implementer`, `reviewer`, `verifier`, `spike` — roles in
the process, each with a fixed model: **fable** for `planner`, `reviewer` and
`spike` (evaluative work), **opus** for `implementer` (production work), sonnet
for `explorer` and `verifier`. Implementation therefore runs at the ceiling,
which removes "retry on a stronger model" from the fix loop — see
`aikit:handling-blockers`. A project's domain agents (`next_expert`, …) are the other axis:
when a plan task names one, it is dispatched instead of the generic
`implementer`. The registry says which exist.

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
claude plugin marketplace add ~/ezytail-workspace/aikit --scope project
claude plugin install aikit@aikit-local --scope project
```

`--scope project` writes to `ezytail-workspace/.claude/settings.json`: aiKit is
active in the workspace and nowhere else on the machine.

## Launch

```bash
aikit/bin/ezy                    # agents + skills of every registered project
aikit/bin/ezy --tools            # ... plus the MCP servers projects declare
aikit/bin/ezy --no-dirs --tools  # MCP only
aikit/bin/ezy --only ezylive     # one project
```

**Two orthogonal switches**, because the harness treats them independently:

| | carries | does not carry | cost |
|---|---|---|---|
| `--add-dir` | agents, skills | `.mcp.json`, `CLAUDE.md` | ~1.4k tok, whole workspace |
| `--mcp-config` | MCP servers | skills, agents | ~5-6k tok for the toolbelt's 49 tools |

Both measured. `--mcp-config` points at the toolbelt's own `.mcp.json` — the
file apm maintains — so there is no copy to keep in sync, and none of its
`Bearer` tokens are duplicated anywhere.

Tools are opt-in because of that cost, not because of a boundary. The boundary
is enforced elsewhere and more precisely: the four destructive natseyes tools
are denied in `ezytail-workspace/.claude/settings.json`, which **removes them
from the schema** rather than refusing them at call time. Read everything from
the workspace; mutate only from a session launched inside the toolbelt.

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
