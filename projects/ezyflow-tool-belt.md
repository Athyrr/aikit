---
name: ezyflow-tool-belt
path: ezyflow-tool-belt
perimeter: .
summary: monorepo de serveurs MCP exposant la stack ezyflow (commandes, livraisons, logs, NATS) - la boite a outils de diagnostic
allow: mcp__periscope mcp__gaia mcp__ref-match mcp__kube-vigie-recette mcp__kube-vigie-prod mcp__natseyes__detect_divergence mcp__natseyes__get_consumer mcp__natseyes__get_kv_value mcp__natseyes__get_message mcp__natseyes__get_pending_messages mcp__natseyes__get_pending_subjects mcp__natseyes__get_stream_info mcp__natseyes__list_consumers mcp__natseyes__list_kv_buckets mcp__natseyes__list_kv_keys mcp__natseyes__list_servers mcp__natseyes__list_streams mcp__natseyes__read_messages mcp__natseyes__search_subjects Read Grep Glob Skill
---

# ezyflow-tool-belt

## Identity

| | |
|---|---|
| Git root | `ezyflow-tool-belt/` — the **maintenance clone**, nothing is wired in it |
| Base branch | `master` |
| Remote | `git@github.com:Ezytail/ezyflow-tool-belt.git` |
| Installed as | apm package, **into the workspace root** |

The clone and the installation are two different things. You edit a tool in the
clone; you *use* the toolbelt from the installation. Wiring the clone as well —
which `apm install` run inside it does — produces two copies of the same
servers and a `.claude/skills/` that drifts from its own `.apm/skills/` source.
It was removed on 2026-08-21.

## What it is

Monorepo of servers exposing the ezyflow stack — orders, deliveries, logs,
product reference data, NATS/JetStream — to humans (web UI, CLI) and to agents
over MCP.

## How to reach it

**It is already there.** Installed once at the workspace root:

```bash
cd /home/adam_adhar/ezytail-workspace
apm install Ezytail/ezyflow-tool-belt --target claude
```

which writes `.mcp.json` (six HTTP servers) and `.claude/skills/` (seven
routing skills), and keeps the package under `apm_modules/ezytail/`.
`--target claude` is what keeps `.cursor/`, `.vscode/` and `.agents/` out — the
manifest declares three runtimes and would deploy to all of them.

What that installation reaches, measured 2026-08-21:

| From a session opened in | six MCP servers | seven skills |
|---|---|---|
| the workspace root | yes | yes |
| `ezylive/ezy_live` (sub-repo) | **yes** | no |
| `ezyflow-tool-belt/` (sub-repo) | **yes** | no |

`.mcp.json` lookup **walks up the directory tree**, so the tools follow you
into any project of the workspace with no flag and no `--add-dir`. Skills do
not travel: they are read from the session's own project root. A session that
has the tools but not `ezyflow-tools` guesses at server names — measured.

`.claude/settings.json` carries `enableAllProjectMcpServers: true`, otherwise
every one of the six sits at "pending approval" until an interactive session
blesses it, and an unattended one never gets them at all.

**No secret is in the file.** The three `Authorization` headers are
`${VAR:-}` placeholders the harness resolves at launch. Verified after this
install, with the three variables set in the environment: apm still wrote the
literal placeholder, because its own regex does not recognise the `:-` default
form. The secrets live in the shell:

```bash
eval "$(/home/adam_adhar/ezytail-workspace/ezyflow-tool-belt/toolbelt-env)"
```

**The perimeter is still worth opening** — `aikit:delegating-to-a-perimeter`,
`bin/scoped ezyflow-tool-belt <feature> "<question>"`. Not for the tools, which
you already have, but for two things: the seven routing skills, which only
exist at the workspace root (hence `perimeter: .` above), and the forty tool
calls of an investigation, which land in a context you throw away instead of
yours.

## The destructive surface

Four natseyes tools mutate **production**: `delete_message`, `delete_consumer`,
`delete_kv_key` (all annotated `destructiveHint=true`) and `replay_message`,
which fans a message out to every consumer of its subject.

**No blocklist stands in their way, by decision.** The gate is the ordinary
permission prompt: in an interactive session a call to any of them stops and
asks you. The other five servers are read-only by construction — kube-vigie
bounds an agent's verbs with `READ_VERBS`, and the rest are query surfaces.

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

Only when *maintaining* the toolbelt: `CLAUDE.md` at the root, plus the
per-tool `tools/<tool>/CLAUDE.md`. Using it needs none of that — the
`ezyflow-tools` skill routes, `ezy-expert` holds the domain vocabulary,
`ezy-context` fixes the terms.

## Domain agents

None. Its seven skills carry the routing instead.

## Completion criterion

Applies to maintenance work in the clone. Each tool's suite runs **from its own
directory** — running them together from the root breaks natseyes (its
`templates/`, `static/` and `CONFIG_PATH` are relative, and it yields 207
errors). The root `pytest.ini` bounds `testpaths` to the root scripts for
exactly that reason.

```bash
pytest                                          # root scripts only
cd tools/natseyes && venv-ubuntu/bin/pytest tests/
```

Other tools have no virtualenv in this clone: create one before claiming their
suite passes.

## Traps

- **The skill sources are `.apm/skills/`.** Any `.claude/skills/` or
  `.agents/skills/` inside the clone is an `apm install` artifact that drifts
  from source. Never edit them, and never run `apm install` in the clone
  without a reason — the install belongs to the workspace.
- **The installed package is pinned to a sha, the clone is not the same tree.**
  `apm.lock.yaml` at the workspace root names what is actually loaded. On
  2026-08-21 the clone was one commit behind that sha: a skill fixed in the
  clone changes nothing until `apm update` runs at the workspace root.
- **`.mcp.json` is gitignored because apm writes real token values into it** —
  true for a plain `${VAR}`, not for the `${VAR:-}` form this manifest uses.
  Check before assuming a given file is safe; never commit one either way.
- **Four natseyes MCP tools mutate production**: `delete_message`,
  `delete_consumer`, `delete_kv_key` and `replay_message`. Never call them to
  "test" anything.
- `READ_VERBS` bounds the verbs an *agent* may choose in kube-vigie — see the
  root `CLAUDE.md` section.
