# Changelog

All notable changes to the aiKit method. Versions follow the plugin manifests
(`.claude-plugin/plugin.json`). Dates are the commit dates.

## 0.11.0
- **Fast-Path vs Heavy-Path.** `aikit:understanding-need` estime la taille en
  phase 1 : deux fichiers ou moins, pas de rupture de contrat, pas de
  dependance critique passe en Fast-Path — dispatch direct de
  `aikit:implementer`, validation par `git diff --name-only`, aucun artifact.
  Tout le reste reste Heavy-Path, les six phases completes.
- **Cascade de modeles.** `aikit:implementer` demarre sur sonnet (etait opus).
  L'escalade vers opus n'est plus une decision au cas par cas : elle se
  declenche automatiquement dans `aikit:handling-blockers`, apres deux echecs
  de la boucle de correction consignes dans le ledger.
- **Hook SessionStart allege.** Le stub injecte les regles cardinales et le
  registre des vaults ; il ne cat plus le corps complet de
  `using-aikit/SKILL.md`, qui se charge a la demande, comme n'importe quel
  autre skill, la premiere fois qu'une session l'invoque.
- **`spec.md` porte son propre Impact.** La phase 2.5 (expert du domaine)
  n'ecrit plus `impact.md` : sa reponse devient une section `## Impact`
  ajoutee a `spec.md`. `plan.md` et `ledger.md` restent des fichiers a part.
- `scripts/doctor` et `scripts/test-vaults` valident le JSON en preferant
  node a python3, meme ordre que `scripts/deploy` : un `python3` present sur
  le PATH mais non fonctionnel (stub Windows "App Execution Alias") ne fait
  plus tomber les gates 4 et 9.

## 0.10.0
- **Le vault remplace le workspace comme unite d'association.** La remontee
  d'ancetres (`find_workspace()`, dupliquee dans trois fichiers) disparait :
  les vaults sont declares par machine dans `~/.config/aikit/vaults`, les
  projets localises par scan et apparies par remote git normalise. `repo:`
  remplace `path:` dans le contrat de la fiche registre ; `dir:` dit ou est
  le repertoire de travail, oriente au depot localise quand `repo:` est
  present.
- **Un projet sans depot est un projet.** `kind: conception` declare
  explicitement l'absence de depot, de branche et de critere de completion
  mecanique, et son `summary:` est injecte comme n'importe quel autre.
- **Neuvieme porte** : `scripts/doctor` lance desormais `scripts/test-vaults`,
  la suite qui garde `lib/vaults` — code execute a chaque demarrage de
  session.
- Le piege « ne pas ouvrir de session aiKit dans le vault » (`SETUP.md`) est
  supprime : il n'existe plus de remontee d'ancetres a tromper.
- Contrat de frontmatter a deux niveaux dans le vault : `status.md`,
  obligatoire, seule source de l'etat d'un chantier (`phase` orthogonal a
  `status`) ; tout autre fichier de chantier porte un contrat minimal
  (`project`, `feature`, `title`).

## 0.9.1
- `scripts/deploy` pousse vers la ref amont explicite
  (`git push "$remote" HEAD:"$ref"`) au lieu d'un `git push` nu : `push.default`
  vaut `simple` partout ou personne ne l'a configure, et `simple` refuse un push
  dont le nom de la branche amont differe du nom de la branche locale. Le
  deploiement mourait donc apres le bump et apres le commit, sans chemin de
  rejeu — la version commitee mais jamais publiee, et un second run bumpant a
  partir d'un numero deja consomme.
- Le remote et la branche sortent de `branch.<x>.remote` et `branch.<x>.merge`,
  jamais d'un decoupage de `@{upstream}` : git accepte un remote dont le nom
  porte un `/`, et `merge` porte deja le ref complet cote distant.
- L'amont est resolu avant la premiere ecriture : un amont absent, ou une HEAD
  detachee, fait echouer deploy sans consommer de version ni laisser de commit
  orphelin.

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
