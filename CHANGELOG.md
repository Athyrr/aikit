# Changelog

All notable changes to the aiKit method. Versions follow the plugin manifests
(`.claude-plugin/plugin.json`). Dates are the commit dates.

## 0.9.0
- Plugin distribue depuis `git@github.com:Athyrr/aikit.git` via un marketplace
  `github` prive en SSH : editer l'arbre ne publie plus rien, `scripts/deploy`
  publie.
- Le registre (`projects/*.md`) sort du plugin et vit dans le vault de l'espace,
  sous `<espace>/vault/projects/`. La sonde du hook suit.
- Les artefacts s'appellent `vault/`, plus `work/`.
- `bin/` n'accueille que des commandes d'execution ; `deploy` et `doctor`
  passent dans `scripts/`, hors du `PATH`.
- `scripts/doctor` porte huit portes, dont trois greps de cible et la
  verification de `core.hooksPath` ; `.githooks/pre-commit` les execute.
- Windows : `"shell": "bash"` retire, l'echec « aucun bash » devient bruyant.

## 0.8.7
- Registry extracted from the method repository: `projects/` now ships a
  skeleton and one example only; a workspace carries its own registry.
- Manifests, README and CLAUDE.md made workspace-agnostic; install documented
  from a git clone at user scope.
- `bin/doctor` added — the four validation gates without deploying.

## 0.8.6
- README: the method's load-bearing patterns; phases aligned on `using-aikit`.

## 0.8.5
- Archetypes may invoke skills; `work/` is an Obsidian vault; write its syntax
  through `obsidian:obsidian-markdown`.

## 0.8.4
- Diagnostic phase recorded in the traces' frontmatter.

## 0.8.3
- `work/` is an Obsidian vault: a `done` status closes the cycle at phase 6.

## 0.8.2
- Toolbelt installed in the workspace: launcher without MCP, `perimeter:` for
  the scoped process; fiche and README updated.

## 0.8.1
- `bin/deploy` pushes to user scope, not project scope.

## 0.8.0
- Registry specified and normalised; `deny` removed, `allow` as an allow-list.

## 0.7.0
- Method generic everywhere; registry found only by ancestry.

## 0.6.1
- Registry found by walking up from any cwd; `.mcp.json` carries no secrets.

## 0.6.0
- Loading policy + phase 2.5 (impact) + the routing tables.

## 0.5.0
- Perimeter scope: `bin/scoped` + the `delegating-to-a-perimeter` skill.

## 0.4.x
- Direct access to a project's MCP tools; destructive tools refused; archetypes
  dispatched with the `aikit:` prefix.

## 0.3.x
- Per-phase model constraint: fable for plan/review/spike, opus for
  implementation.

## 0.2.x
- Four new skills + the six archetypes + wiring; spike is an agent, not a skill.

## 0.1.x
- Detached fork from superpowers; project registry + hook reading the live
  source; `bin/deploy`; the `ezy` launcher deriving paths from the registry.
