# Working on aiKit itself

This repository *is* the method. Editing it changes how every session in the
workspace behaves, including this one.

## Rules

- **Skills are prompts, not documentation.** Every line costs context in a
  real session. Cut before you add.
- `skills/using-aikit/SKILL.md` is injected **in full** at every session start
  and after every compaction. Adding ten lines there taxes every session
  forever. Keep it under ~100 lines.
- Use `aikit:writing-skills` when creating or editing a skill.
- After touching `hooks/session-start`, verify it still emits valid JSON:
  ```bash
  CLAUDE_PLUGIN_ROOT=$PWD bash hooks/session-start | python3 -m json.tool > /dev/null
  ```
- After touching any manifest: `claude plugin validate .`
- A plugin change only takes effect in a **new** session.

## Agents

- **The dispatch name carries the prefix**: `aikit:planner`, not `planner`.
  A bare name fails with "subagent_type does not exist". Project domain agents
  (`next_expert`, …) are not prefixed — they come from their own repository.
- **A `tools:` list in an agent's frontmatter excludes MCP tools.** Measured:
  `aikit:explorer`, restricted to `Glob, Grep, Read, Bash, Write, TodoWrite`,
  sees no `mcp__*` tool at all, while `general-purpose` (tools `*`) inherits
  every connected server. So an agent that must reach the toolbelt cannot
  declare a `tools:` list — omit it and constrain by instruction instead.
- Subagents **inherit** the session's MCP connections. They never establish
  their own. What the launcher wired is what they get.

## Attribution

Derived from superpowers (MIT, Jesse Vincent). Keep the notice in `LICENSE`.
There is no upstream remote and no merge path — that is deliberate.
