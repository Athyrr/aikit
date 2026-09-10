# Working on aiKit itself

This repository *is* the method. Editing it changes how every aiKit session
behaves, including this one.

**aiKit is a plugin, not a harness.** The harness is Claude Code — it owns the
tools, the context window, the model calls, the permission system and the
hooks. aiKit ships prompts, agent definitions and one hook that the harness
consumes. Anything written here that assumes aiKit *is* the runtime is wrong.

## Rules

- **Skills are prompts, not documentation.** Every line costs context in a
  real session. Cut before you add.
- `skills/using-aikit/SKILL.md` is injected **in full** at every session start
  and after every compaction. Adding ten lines there taxes every session
  forever. Keep it under ~125 lines (~1,600 tokens).
- **Any agent carrying a `tools:` list must include `Skill` in it.** A `tools:`
  list is exhaustive: without `Skill` an agent cannot invoke a single skill —
  not even the one its own body tells it to follow. `agents/implementer.md`
  carries **no** `tools:` list, deliberately — that is the only way to reach a
  project's MCP tools — so it inherits everything.
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

> [!warning] The next bullet is not true yet — read this one first
> It describes the **target** montage. Until **task 17 of the `distribution`
> chantier** switches it, the montage measured in
> `~/.claude/plugins/known_marketplaces.json` is the opposite:
>
> ```json
> "aikit-local": {
>   "source": { "source": "directory", "path": "<the absolute path of THIS repository>" },
>   "installLocation": "<the same path again>"
> }
> ```
>
> A `directory` marketplace whose install location **is this working tree**. So
> today the harness loads the plugin from these very files, and nothing —
> no bump, no commit, no push — stands between an editor and production:
> **every saved edit is live for every session in the workspace at the next
> `startup`, `/clear` or compaction.** Edit `skills/using-aikit/SKILL.md`,
> `hooks/session-start` or any `agents/*.md` as if it were already deployed,
> because it is. Delete this callout in task 17, with the montage it describes.

- **A plugin change only takes effect in a new session, and only after it is
  published.** `aikit-marketplace` is a `github` marketplace: the harness runs a
  copy pinned to a version under `~/.claude/plugins/cache/`, never this working
  tree. Editing a file here changes nothing until `scripts/deploy` has bumped,
  committed and pushed. `/reload-plugins` reloads skills, agents and hooks
  without restarting; only the SessionStart preamble needs `startup|clear|compact`
  — and `/clear` is one of those. `claude --plugin-dir .` loads this tree for one
  session, taking precedence over the installed copy.

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
  A bare name fails with "subagent_type does not exist". A project's own domain
  agents are not prefixed — they come from its repository.
- **Renaming resolves differently for agents and skills.** For an agent, the
  frontmatter `name:` wins and the filename is cosmetic. For a skill, the model
  sees the *directory* name — but the old frontmatter `name:` keeps resolving as
  a typed command, so a directory-only rename leaves a working ghost that passes
  every grep. **Always move the directory (or file) and the frontmatter `name:`
  together.**
- **A `tools:` list in an agent's frontmatter excludes MCP tools.** Measured:
  an agent restricted to `Glob, Grep, Read, Bash, Write, TodoWrite, Skill` sees
  no `mcp__*` tool at all, while one with tools `*` inherits every connected
  server. So an agent that must reach a project's MCP servers cannot declare a
  `tools:` list — omit it and constrain by instruction.
- Subagents **inherit** the session's MCP connections. They never establish
  their own. What the launcher wired is what they get.
- The six archetypes are the *product* of this repository, not its experts.
  For questions about harness mechanics — name resolution, install scope, hook
  behaviour — the domain expert is `claude-code-guide`. See `projects/aikit.md`.

## Attribution

Derived from superpowers (MIT, Jesse Vincent). Keep the notice in `LICENSE`.
