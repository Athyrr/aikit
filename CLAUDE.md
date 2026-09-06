# Working on aiKit itself

This repository *is* the method. Editing it changes how every session in the
workspace behaves, including this one.

**aiKit is a plugin, not a harness.** The harness is Claude Code — it owns the
tools, the context window, the model calls, the permission system and the
hooks. aiKit ships prompts, agent definitions and one hook that the harness
consumes. Anything written here that assumes aiKit *is* the runtime is wrong.

## Rules

- **Skills are prompts, not documentation.** Every line costs context in a
  real session. Cut before you add.
- `skills/using-aikit/SKILL.md` is injected **in full** at every session start
  and after every compaction. Adding ten lines there taxes every session
  forever. Keep it under ~125 lines (it is at 120, and ~1,600 tokens).
- **Any agent carrying a `tools:` list must include `Skill` in it.** A `tools:`
  list is exhaustive: without `Skill` an agent cannot invoke a single skill —
  not even the one its own body tells it to follow. `agents/planner.md` said
  "Follow `aikit:writing-plans`" for eight versions while being unable to.
  `agents/implementer.md` carries **no** `tools:` list, deliberately — that is
  the only way to reach the toolbelt's MCP tools — so it inherits everything.
- Use `aikit:writing-skills` when creating or editing a skill.
- The completion criterion lives in `projects/aikit.md`, like every other
  project's. Run it; do not invent an equivalent.
- After touching `hooks/session-start`, verify it still emits valid JSON:
  ```bash
  CLAUDE_PLUGIN_ROOT=$PWD bash hooks/session-start | python3 -m json.tool > /dev/null
  ```
- **`claude plugin validate .` validates the marketplace manifest and nothing
  else** — it reads no `SKILL.md` and no `agents/*.md`. The real gates are three
  commands, not one:
  ```bash
  claude plugin validate .                  # manifest only
  claude plugin validate ./skills --strict
  claude plugin validate ./agents --strict
  ```
- **A plugin change only takes effect in a new session — but nothing else
  gates it.** `aikit-local` is a `directory` marketplace pointing at this
  working tree, so the harness loads the plugin from the tree itself, never
  from the copy under `~/.claude/plugins/cache/`. Uncommitted edits ship at the
  next session start. Measured 2026-09-06: that cache is frozen at `6556902`
  and carries no `skills/handling-secrets/`, yet the preamble injected into a
  session names `aikit:handling-secrets` — a line that exists only here, in an
  untracked file. `bin/deploy` bumps and commits; it does not decide what runs.

## It will not stay standalone

aiKit will be versioned and published. Do not write as though this repository
is private workspace tooling — two consequences, both binding now:

- **Keep workspace-specific facts out of the method surface.** Skills, agents
  and `README.md` describe the method. Paths, project names, MCP servers and
  infrastructure belong in `projects/*.md`, which is data.
- **`projects/*.md` is the unresolved boundary.** Those files describe private
  infrastructure — NATS streams, internal services, repository layouts — and
  they currently sit inside the repository that will be published. Settle how
  they are separated *before* the first public push, not after.

## Agents

- **The dispatch name carries the prefix**: `aikit:planner`, not `planner`.
  A bare name fails with "subagent_type does not exist". Project domain agents
  (`next_expert`, …) are not prefixed — they come from their own repository.
- **Renaming resolves differently for agents and skills.** For an agent, the
  frontmatter `name:` wins and the filename is cosmetic. For a skill, the model
  sees the *directory* name — but the old frontmatter `name:` keeps resolving as
  a typed command, so a directory-only rename leaves a working ghost that passes
  every grep. **Always move the directory (or file) and the frontmatter `name:`
  together.** All six agents currently agree.
- **A `tools:` list in an agent's frontmatter excludes MCP tools.** Measured:
  `aikit:explorer`, restricted to `Glob, Grep, Read, Bash, Write, TodoWrite,
  Skill`, sees no `mcp__*` tool at all, while `general-purpose` (tools `*`)
  inherits every connected server. So an agent that must reach the toolbelt
  cannot declare a `tools:` list — omit it and constrain by instruction.
- Subagents **inherit** the session's MCP connections. They never establish
  their own. What the launcher wired is what they get.
- The six archetypes are the *product* of this repository, not its experts.
  For questions about harness mechanics — name resolution, install scope, hook
  behaviour — the domain expert is `claude-code-guide`. See `projects/aikit.md`.

## Attribution

Derived from superpowers (MIT, Jesse Vincent). Keep the notice in `LICENSE`.
