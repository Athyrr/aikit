# Changelog

All notable changes to the aiKit method. Versions follow the plugin manifests
(`.claude-plugin/plugin.json`). Dates are the commit dates.

## 0.13.0
- **Les renommages : un mot, un sens.** 8 repertoires et 8 `name:` alignes sur
  `VOCABULARY.md` en un seul commit — `spike` -> `probe`, `sdd` -> `runs`, et
  les unifications de `loading-policy` (-> `budgeting-context`) et
  `systematic-debugging` (-> `diagnosing`). README, hooks/session-start et
  toutes les references croisees suivent dans le meme commit : un renommage
  qui laisse une reference perimee derriere lui recree l'alias fantome que la
  porte 10 existe pour detecter.
- Deux phrases cassees par la substitution mecanique du renommage, corrigees
  dans `routing-failures` et `executing-plans` — la casse ou la substitution
  d'un mot dans une phrase peut en briser la grammaire meme quand le mot est
  juste.
- `executing-plans` absorbe l'ancien contenu de son propre nom en section
  unique : une seule skill porte l'execution, plus deux qui se recouvraient
  partiellement.

## 0.12.0
- **`VOCABULARY.md` : un mot, un sens.** Nouveau fichier a la racine du depot,
  lu obligatoirement (regle ajoutee dans `writing-skills/SKILL.md` et
  `CLAUDE.md`) avant de nommer quoi que ce soit — skill, agent, phase,
  artefact. Il se lit a la racine d'aiKit, jamais du projet en cours.
- **Dixieme porte.** `scripts/doctor` verifie que le repertoire de chaque
  skill egale son `name:` en frontmatter. Renommer seulement le repertoire
  laisse l'ancien `name:` resoudre comme une commande fantome — un alias qui
  passe tous les greps et que seule cette porte detecte. Mesure en phase 3 de
  refonte-methode. `SETUP.md` et `scripts/deploy` rattrapent la comptabilite
  neuf -> dix portes.
- **`plan-drift` refuse une entree `Files:` illisible au lieu de la deviner.**
  Deux chemins sur une ligne, ou de la prose apres le chemin, etaient avales
  entiers : le chemin impossible revenait INCHANGE pendant que le vrai fichier
  revenait EXTRA, route comme une dependance non planifiee — un echec vers le
  haut. Quatre faux verdicts DRIFT sur un cycle mesure ; le parseur leve
  desormais plutot que de deviner.
- `grilling` vendore (MIT, mattpocock/skills) : son unite est la passe, pas le
  round.
- `dispatching-parallel-agents` supprimee : un plan sequence par fichiers
  partages, pas par domaine.

## 0.11.3
- **`lib/vaults` resout la fiche du vault lui-meme hors scan/probe.** Le vault
  n'est jamais range sous une racine de projets — ce n'est pas un oubli de
  config mais sa nature — donc sa propre fiche ne pouvait jamais matcher par
  le scan/probe generique : un faux negatif garanti (« absent » alors que
  c'est la fiche en cours de lecture), pas une information vraie comme pour un
  projet qui peut reellement manquer. `resolve_projects()` compare desormais
  le `repo:` de la fiche au remote git du vault lui-meme avant de retomber sur
  le scan.

## 0.11.2
- **`using-aikit/SKILL.md` repasse sous son plafond.** 153 -> 124 lignes ; la
  forme longue (tables de routage completes, rationale des modeles, matrice
  d'echec ligne a ligne) part dans un companion `reference.md`, charge a la
  demande comme n'importe quel autre companion du depot.
- **Regle cardinale n1 clarifiee.** « Every request starts with... » devient
  « once per distinct need, not once per message » : repondre a une question
  de clarification a l'interieur d'un besoin deja route ne re-invoque plus
  `aikit:understanding-need`. Regle explicite ajoutee dans
  `understanding-need/SKILL.md`.
- **Bug de hook corrige.** `.githooks/pre-commit` neutralise desormais
  `GIT_DIR`/`GIT_WORK_TREE`/`GIT_INDEX_FILE` avant `scripts/doctor` : ces
  variables, posees par git pour le process qui execute un hook, ecrasaient
  la decouverte du depot que `git -C` fait normalement, et la porte 9
  (`scripts/test-vaults`) echouait silencieusement sous `git commit` — jamais
  en invocation directe de `scripts/doctor`.

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
