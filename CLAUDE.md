# Working on aiKit itself

This repository *is* the method. Editing it changes how every aiKit session
behaves, including this one.

**aiKit is a plugin, not a harness.** The harness is Claude Code — it owns the
tools, the context window, the model calls, the permission system and the
hooks. aiKit ships prompts, agent definitions and one hook that the harness
consumes. Anything written here that assumes aiKit *is* the runtime is wrong.

## Règle d'aiguillage d'exécution obligatoire

- **FAST-PATH** (session principale directe, sans `vault/<project>/<feature>/plan.md`, sans
  sous-agent) — condition stricte : **≤ 2 fichiers modifiés ET aucun
  changement d'API publique/schéma de données**. Action : exécution
  immédiate du code + test ciblé. Interdiction de générer de la
  documentation intermédiaire.
- **HEAVY-PATH** (Subagent-Driven Development) — condition : refactorings
  multi-fichiers, nouvelles fonctionnalités, rupture d'API. Action :
  spécification minimale, découpage en micro-tâches, délégation à des
  sous-agents jetables — la table des phases dans `skills/using-aikit/SKILL.md`.

`aikit:understanding-need` fait cet arbitrage à la phase 1 et le prouve :
`git diff --name-only` pour le Fast-Path, `aikit:checking-plan-drift` par
tâche pour le Heavy-Path. C'est la même règle que documentent
`skills/using-aikit/SKILL.md` et `skills/using-aikit/reference.md` — ce
bloc en est la formulation impérative, pas une règle séparée.

## Rules

- **Skills are prompts, not documentation.** Every line costs context in a
  real session. Cut before you add.
- `hooks/session-start` injects a **compact stub** at every session start and
  after every compaction — cardinal rules and the vault registry table, not
  the method's full text. `skills/using-aikit/SKILL.md` itself loads on
  demand, like any other skill, the first time a session invokes it. Keep the
  stub (the `preamble` string in `hooks/session-start`) small enough to read
  in one screen — it pays on every session and every compaction, unconditionally.
  The full skill body pays once, in whichever context invokes it, so it can
  afford to be larger, but it is still read by a live context: cut before you add.
- **Any agent carrying a `tools:` list must include `Skill` in it.** A `tools:`
  list is exhaustive: without `Skill` an agent cannot invoke a single skill —
  not even the one its own body tells it to follow. `agents/implementer.md`
  carries **no** `tools:` list, deliberately — that is the only way to reach a
  project's MCP tools — so it inherits everything.
- Use `aikit:writing-skills` when creating or editing a skill.
- **Heavy-Path execution tracks its plan and status in one file,
  `vault/<project>/<feature>/plan.md`** — no separate ledger, no
  `status.md`, no review-package file. aiKit writes nothing inside the
  project repository itself. Task state is standard GitHub checkboxes. A
  task whose second fix attempt still fails review stops the loop and asks
  the human partner for arbitration, instead of escalating rounds or models
  automatically.
- **`VOCABULARY.md` is the authority on every term the method uses.** Read it
  before naming anything — a skill, an agent, a phase, an artifact. It is a
  file, not a skill: read it by path, do not invoke it.
- The completion criterion is `scripts/doctor`, in this repository — see
  **Completion criterion** below. Run it; do not invent an equivalent.
- After touching `hooks/session-start`, verify it still emits valid JSON:
  ```bash
  CLAUDE_PLUGIN_ROOT=$PWD bash hooks/session-start | python3 -m json.tool > /dev/null
  ```
- **`claude plugin validate .` validates the marketplace manifest and nothing
  else** — it reads no `SKILL.md` and no `agents/*.md`. That is why gates 1-3
  are three separate commands, not one:
  ```bash
  claude plugin validate .                  # manifest only
  claude plugin validate ./skills --strict
  claude plugin validate ./agents --strict
  ```

- **A plugin change only takes effect in a new session, and only after it is
  published**. `aikit-marketplace` is a `github` marketplace: the harness runs
  a copy pinned to a version under `~/.claude/plugins/cache/`, never this
  working tree. Editing a file here changes nothing until `scripts/deploy` has
  bumped, committed and pushed.
  `/reload-plugins` reloads skills, agents and hooks without restarting; only
  the SessionStart preamble needs `startup|clear|compact` — and `/clear` is one
  of those. `claude --plugin-dir .` loads this tree for one session, taking
  precedence over the installed copy.

## Completion criterion

No test suite. One command, from the repository root, carrying **ten gates**:

```bash
scripts/doctor
```

It prints `GATES_PASS` and exits 0 when all ten pass; it exits 1 otherwise,
naming each gate that fell.

| # | Gate |
|---|---|
| 1 | `claude plugin validate .` — the marketplace manifest ONLY |
| 2 | `claude plugin validate ./skills --strict` — the skills |
| 3 | `claude plugin validate ./agents --strict` — the archetypes |
| 4 | `hooks/session-start` emits valid JSON |
| 5 | no ancestor-walk resolution left — a tripwire against its return, not a debt probe |
| 6 | no probe left pointing at the plugin's former bundled registry |
| 7 | no workspace fact left in the tree — five hard-coded brand strings |
| 8 | `core.hooksPath` is `.githooks` |
| 9 | `scripts/test-vaults` — the vault-resolution suite. `lib/vaults` runs at every session start, so its tests belong to the completion criterion rather than beside it |
| 10 | every skill's directory name equals its frontmatter `name:` — the only detector of a directory-only rename, which leaves a working ghost alias |

Gates 1-4 and 10 check form. Gates 5-7 check the target rather than the shape: they
catch what no manifest validation can see. Gate 8 is **local** git config — it
does not clone, push or inherit, so a fresh clone must arm it once:

```bash
git config core.hooksPath .githooks
```

That is the whole point of gate 8, making the arming visible instead of assumed.
Once armed, `.githooks/pre-commit` runs all ten before every commit, and a red
gate refuses it.

**What it does not prove, and says so itself** — `A NEW session is still
required to prove the method loads.` A plugin change only takes effect in a new
session, so until a fresh one has loaded the method without error, the honest
verdict is `GATES_PASS`, never `PASS`.

## It will not stay standalone

aiKit will be versioned and published. Do not write as though this repository
is private workspace tooling.

**Keep workspace-specific facts out of the method surface.** Skills, agents and
`README.md` describe the method. Paths, project names, MCP servers and
infrastructure belong in the registry — `<vault>/projects/*.md`, which
is data and lives in a vault, never in this repository.

Gate 7 of `scripts/doctor` catches part of this, and only part: it greps the
tree for **five hard-coded brand strings**, nothing else. A private
infrastructure fact that avoids those five words passes green. It is a tripwire
for the obvious cases, never proof that the surface is clean — that judgement
stays yours.

## Agents

- **The dispatch name carries the prefix**: `aikit:planner`, not `planner`.
  A bare name fails with "subagent_type does not exist". A project's own domain
  agents are not prefixed — they come from its repository.
- **Renaming resolves differently for agents and skills.** For an agent, the
  frontmatter `name:` wins and the filename is cosmetic. For a skill, the model
  sees the *directory* name — but the old frontmatter `name:` keeps resolving as
  a typed command, so a directory-only rename leaves a working ghost that passes
  every grep. **Always move the directory (or file) and the frontmatter `name:`
  together.** Gate 10 of `scripts/doctor` is what makes that mechanical.
- **A `tools:` list in an agent's frontmatter excludes MCP tools.** Measured:
  an agent restricted to `Glob, Grep, Read, Bash, Write, TodoWrite, Skill` sees
  no `mcp__*` tool at all, while one with tools `*` inherits every connected
  server. So an agent that must reach a project's MCP servers cannot declare a
  `tools:` list — omit it and constrain by instruction.
- Subagents **inherit** the session's MCP connections. They never establish
  their own. What the launcher wired is what they get.
- The six archetypes are the *product* of this repository, not its experts.
  For questions about harness mechanics — name resolution, install scope, hook
  behaviour — the domain expert is `claude-code-guide`. See the
  registry file `<vault>/projects/aikit.md`, outside this repository.

## Attribution

Derived from superpowers (MIT, Jesse Vincent). Keep the notice in `LICENSE`.
