---
name: ezyflow-tool-belt
path: ezyflow-tool-belt
summary: monorepo de serveurs MCP exposant la stack ezyflow (commandes, livraisons, logs, NATS) - la boite a outils de diagnostic
mcp: .mcp.json
allow: mcp__natseyes mcp__periscope mcp__gaia mcp__ref-match mcp__kube-vigie-recette mcp__kube-vigie-prod Read Grep Glob Skill
deny: mcp__natseyes__delete_message mcp__natseyes__delete_consumer mcp__natseyes__delete_kv_key mcp__natseyes__replay_message
---

# ezyflow-tool-belt

| | |
|---|---|
| Git root | `ezyflow-tool-belt/` |
| Base branch | `master` |
| Remote | `git@github.com:Ezytail/ezyflow-tool-belt.git` |

Monorepo of servers exposing the ezyflow stack — orders, deliveries, logs,
product reference data, NATS/JetStream — to humans (web UI, CLI) and to agents
over MCP. Distributed as an apm package.

## How to reach it

The toolbelt is a tool. aiKit uses it the way a developer does — through the
MCP servers, from wherever the work is.

**The default route is a scoped process**, so the orchestrator never carries
the toolbelt:

```bash
aikit/bin/scoped ezyflow-tool-belt <feature> "<question>"
```

See `aikit:delegating-to-a-perimeter`. The process runs with `cwd` here, so it
loads the six MCP servers **and** the seven routing skills natively, and its
own subagents inherit them. Permissions come from the `allow:` / `deny:` lines
of this file's frontmatter.

The ad-hoc alternative, for a one-off question with no plan around it:

```bash
aikit/bin/ezy --tools          # workspace session + the toolbelt's MCP servers
```

It wires `--mcp-config ezyflow-tool-belt/.mcp.json --strict-mcp-config`. That
file is the one apm maintains: no copy, no duplicate, and an `apm install`
refresh is picked up on the next launch. **Never copy it anywhere** — it holds
real `Authorization: Bearer` tokens, which is why it is gitignored. It gives
the tools but **not** the routing skills, so the session guesses at tool names
— measured. Prefer the scoped route whenever the answer matters.

`--add-dir` is not involved and not needed. The two flags are orthogonal:
`--add-dir` carries knowledge without tools, `--mcp-config` carries tools
without knowledge.

**It is opt-in, and the reason is cost**: 49 tool schemas across six servers,
roughly 5-6k tokens resident for the whole session (natseyes alone measures
2,115). Launch with `--tools` when the work needs diagnosis, not by reflex.

## What is blocked from the workspace, and why

Four natseyes tools mutate **production**: `delete_message`, `delete_consumer`,
`delete_kv_key` (all annotated `destructiveHint`) and `replay_message`, which
fans a message out to every consumer of its subject.

They are denied in `ezytail-workspace/.claude/settings.json`. A denied MCP tool
is not refused at call time — it is **removed from the schema**, so it is never
offered and never attempted.

The perimeter is therefore no longer "which directory you launched from" but
something finer and more useful: **read everything from the workspace, mutate
only from the toolbelt.** A session launched inside `ezyflow-tool-belt/` uses
that project's settings, not the workspace's, so the deliberate destructive
work still has its place — just not the same place as writing code.

The toolbelt **reads** the ecosystem and writes nothing back to it. A finding
that requires a code change becomes a task on the target project, never an
edit from here.

## Load before working

`CLAUDE.md` at the root, plus the per-tool `tools/<tool>/CLAUDE.md`.

## Completion criterion

Each tool's suite runs **from its own directory** — running them together from
the root breaks natseyes (its `templates/`, `static/` and `CONFIG_PATH` are
relative, and it yields 207 errors). The root `pytest.ini` bounds `testpaths`
to the root scripts for exactly that reason.

```bash
pytest                                      # root scripts only
cd tools/natseyes   && venv-debian/bin/pytest tests/
cd tools/kube-vigie && venv-debian/bin/pytest
```

## Traps

- **The skill sources are `.apm/skills/`.** `.claude/skills/` and
  `.agents/skills/` are apm *install artifacts* and have already drifted from
  source. Never edit them — the edit is lost on the next install and the source
  keeps the old content.
- **`.mcp.json` is gitignored because apm writes real token values into it.**
  Never commit it, never copy it into a versioned location, never move it into
  the workspace.
- **Four natseyes MCP tools mutate production**: `delete_message`,
  `delete_consumer`, `delete_kv_key` (all marked DESTRUCTIVE in their
  docstrings) and `replay_message`, which fans a message out to every consumer
  of its subject. Never call them to "test" anything.
- `READ_VERBS` bounds the verbs an *agent* may choose in kube-vigie — see the
  root `CLAUDE.md` section.
