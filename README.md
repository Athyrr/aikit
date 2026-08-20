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
| 1 | Understand the need | `aikit:brainstorming` |
| 2 | Specify | `aikit:brainstorming` |
| 3 | Plan | `aikit:writing-plans` |
| 4 | Split into tasks | `aikit:writing-plans` |
| 5 | Execute | `aikit:subagent-driven-development` |
| 6 | Verify | `aikit:verification-before-completion` |

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
claude plugin marketplace add /home/adam_adhar/ezytail-workspace/aikit
claude plugin install aikit@aikit-local
```

## Differences from superpowers

- renamed throughout (`aikit:` prefix), single harness (Claude Code only)
- artifacts moved out of the repositories into `work/`
- per-project registry injected at session start
- failure-direction rule (down = spike/fix loop, up = re-plan/re-spec)
- retry budget written to the ledger, not held in context
- multi-harness ports, CI, upstream docs and the remote brand image removed
