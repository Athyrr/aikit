---
name: ezyflow-tool-belt
path: ezyflow-tool-belt
summary: monorepo de serveurs MCP exposant la stack ezyflow (commandes, livraisons, logs, NATS) - DEBUG UNIQUEMENT
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

## Perimeter — read this before anything else

**The toolbelt is used from the toolbelt.** Its MCP servers only connect when
the session's own project root carries the `.mcp.json` — measured: a workspace
session with `--add-dir ezyflow-tool-belt` gets the seven skills but **none of
the MCP servers**. That is the intended boundary, and the harness enforces it
for free: knowledge crosses, tools do not.

So: **debug work happens in a session launched from this directory.** A
workspace session may read the toolbelt's skills for vocabulary and diagnosis;
it cannot and must not reach the MCP tools.

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
