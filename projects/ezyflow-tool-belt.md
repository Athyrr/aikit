---
name: ezyflow-tool-belt
path: ezyflow-tool-belt
summary: monorepo de serveurs MCP exposant la stack ezyflow (commandes, livraisons, logs, NATS) - la boite a outils de diagnostic
mcp: .mcp.json
allow: mcp__periscope mcp__gaia mcp__ref-match mcp__kube-vigie-recette mcp__kube-vigie-prod mcp__natseyes__detect_divergence mcp__natseyes__get_consumer mcp__natseyes__get_kv_value mcp__natseyes__get_message mcp__natseyes__get_pending_messages mcp__natseyes__get_pending_subjects mcp__natseyes__get_stream_info mcp__natseyes__list_consumers mcp__natseyes__list_kv_buckets mcp__natseyes__list_kv_keys mcp__natseyes__list_servers mcp__natseyes__list_streams mcp__natseyes__read_messages mcp__natseyes__search_subjects Read Grep Glob Skill
---

# ezyflow-tool-belt

## Identity

| | |
|---|---|
| Git root | `ezyflow-tool-belt/` |
| Base branch | `master` |
| Remote | `git@github.com:Ezytail/ezyflow-tool-belt.git` |

## What it is

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
refresh is picked up on the next launch.

**It holds no secrets.** Verified: the two `Authorization` headers are
`${VAR:-}` placeholders that the harness resolves at launch from the
environment; the only literal value is `NATSEYES_URL`. The toolbelt's own
`CLAUDE.md` explains why the default-syntax placeholder is deliberate — apm's
regex does not recognise it, so it never prompts and never freezes a token into
the file. The secrets live in the shell, exported by `toolbelt-env`:

```bash
eval "$(/home/adam_adhar/ezytail-workspace/ezyflow-tool-belt/toolbelt-env)"
```

That is what makes the configuration portable: the file can be referenced from
anywhere, and it only works where the environment is set. It gives
the tools but **not** the routing skills, so the session guesses at tool names
— measured. Prefer the scoped route whenever the answer matters.

`--add-dir` is not involved and not needed. The two flags are orthogonal:
`--add-dir` carries knowledge without tools, `--mcp-config` carries tools
without knowledge.

**It is opt-in, and the reason is cost**: 49 tool schemas across six servers,
roughly 5-6k tokens resident for the whole session (natseyes alone measures
2,115). Launch with `--tools` when the work needs diagnosis, not by reflex.

## The destructive surface

Four natseyes tools mutate **production**: `delete_message`, `delete_consumer`,
`delete_kv_key` (all annotated `destructiveHint=true`) and `replay_message`,
which fans a message out to every consumer of its subject.

**No blocklist stands in their way.** The gate is the ordinary permission
prompt: in an interactive session a call to any of them stops and asks you.
The other five servers are read-only by construction — kube-vigie bounds an
agent's verbs with `READ_VERBS`, and the rest are query surfaces.

The one place a prompt cannot protect you is an **unattended** process, which
approves nothing. That is why the `allow:` line above whitelists natseyes tool
by tool rather than the whole server: `bin/scoped` can read everything and
cannot delete anything, without a blocklist existing anywhere.

*(Worth a look one day: `ref-match` exposes `refmatch_run`, `refmatch_reconcile`
and `refmatch_refresh`. I have not established whether they mutate. They are
allowed at server level today.)*

The toolbelt **reads** the ecosystem and writes nothing back to it. A finding
that requires a code change becomes a task on the target project, never an
edit from here.

## Load before working

`CLAUDE.md` at the root, plus the per-tool `tools/<tool>/CLAUDE.md`.

## Domain agents

None. Its seven skills carry the routing instead — `ezyflow-tools` chooses the
server, `ezy-expert` holds the domain vocabulary, `ezy-context` fixes the terms.
They are only loaded where the toolbelt is installed.

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
