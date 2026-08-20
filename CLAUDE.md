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

## Attribution

Derived from superpowers (MIT, Jesse Vincent). Keep the notice in `LICENSE`.
There is no upstream remote and no merge path — that is deliberate.
