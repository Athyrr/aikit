# Changelog

All notable changes to the aiKit method. Versions follow the plugin manifests
(`.claude-plugin/plugin.json`). Dates are the commit dates.

## 0.27.0
- **Trame et tranches.** Une `trame` (`VOCABULARY.md`, *The trame*) porte le
  decoupage d'un gros projet en tranches : `<projet>/trame.md`, `type: trame`,
  titres obligatoires But du projet, Graphe, Vue d'ensemble, Les tranches,
  Historique du decoupage. Une `tranche` est un need `feature` dont le slug
  figure dans la colonne `Dossier` de la trame. `aikit:understanding-need`
  verifie a l'ouverture d'une tranche que celles de sa cellule `Attend` sont
  archivees avec `status: done`, liste les prerequis hors tranche, s'arrete
  s'il en manque un, puis ecrit `en cours (ouverte le <date>)` ;
  `aikit:verifying-completion` ecrit `archivée le <date>` (ou
  `abandonnée le <date>`) a l'archivage et passe la trame en `status: done`
  apres sa derniere tranche. Nouvelle section « Levels of work » ; `slice`,
  `lot`, `jalon`, `milestone` et `chantier` retires.

## 0.26.0
- **Dependances amont.** Une `upstream dependency` (`VOCABULARY.md`) est tout ce
  dont le travail depend et qui peut changer sans nous. Section facultative
  `## Dépendances` des fiches projet et de `plan.md` (contrat dans
  `aikit:registering-a-project`). Nouvelle commande `bin/aikit-deps`, en lecture
  seule, bases `paquet` (nom exact, ou prefixe `*` a version unique), `surveille`,
  `release`, `branche`, `pr`, `fige`, `manuel`. `aikit:understanding-need` l'appelle
  a chaque besoin et reprise, l'orchestrateur avant le dispatch d'`aikit:planner`
  (`aikit:writing-plans`). Nouvelle skill `aikit:checking-dependencies` pour le tri
  d'un ecart sur un plan avec taches ; ligne dans `aikit:routing-failures`.
  Suite `scripts/test-deps`, porte 13 de `scripts/doctor` (treize portes).
  `scripts/test-plan-drift` repasse en `100755` : sans ce mode, la porte 11
  refusait tout commit.

## 0.25.0
- **Effort par archetype.** Chaque archetype declare `model:` et `effort:`
  dans son frontmatter : `aikit:planner` et `aikit:reviewer` sur opus a
  `high` ; `aikit:explorer`, `aikit:implementer`, `aikit:probe` et
  `aikit:verifier` sur sonnet a `medium`. La table de `aikit:using-aikit` en
  est le miroir, gardee par la nouvelle porte 12 de `scripts/doctor` (douze
  portes). Table d'escalade unique dans `aikit:executing-plans` (Model and
  Effort Selection) : l'effort monte automatiquement sur signal mecanique
  (revue finale, tache `Security: yes`, audit demande, re-revue apres le
  deuxieme fix, demande de l'humain pour le planner), le modele ne change
  jamais sans l'humain, un expert de domaine part toujours sur opus a `high`.
  Les gabarits de dispatch ne passent plus `model`/`effort` hors de cette
  table. Nouvelle propriete de tache `**Security:** yes` (`aikit:writing-plans`,
  `VOCABULARY.md`) ; champ `effort` dans l'evenement `dispatch`.
  `hooks/session-start` avertit quand `CLAUDE_CODE_EFFORT_LEVEL` ou
  `CLAUDE_CODE_SUBAGENT_MODEL_FORCE` est defini (`dispatch_override_warnings`
  dans `lib/vaults`) ; `aikit:understanding-need` suggere `/model opus` a un
  besoin Heavy-Path dans une session qui n'est pas sur opus.
- **`aikit:planner` passe a opus par defaut.** Un plan faux est l'echec le
  plus couteux de la methode, et le planner pese peu dans le cout d'un run.
  `agents/planner.md`, `using-aikit`, `reference.md` et le README sont alignes
  sur `executing-plans` (qui disait deja opus) ; le planner peut etre
  retrograde a sonnet a la main, jamais automatiquement. `using-aikit` :
  un expert de domaine (ligne `Agent:`) demarre toujours sur opus a `high`.
  Suppression du prompt orphelin
  `plan-document-reviewer-prompt.md` : la relecture du plan reste
  l'auto-relecture, le scan pre-flight et la validation humaine.

## 0.20.0
- **References.** Nouveau dossier `_reference/` par projet, ecosysteme et
  `_global` (documents de fond, un `INDEX.md` par dossier) ; nouvelle skill
  `aikit:capturing-reference` (creer ou mettre a jour, sur demande seulement) ;
  `reference` ajoute a `VOCABULARY.md` et `_reference/` aux prefixes reserves ;
  `aikit:registering-a-project` : la fiche ne pointe que vers l'INDEX, le
  squelette est inchange.

## 0.19.0
- **Ecosystemes et `part_of`.** `kind` connait trois valeurs (`depot`,
  `conception`, `ecosysteme`) et une valeur inconnue se voit au demarrage (`⚠`).
  Une fiche peut declarer `part_of: <ecosysteme>` ; le tableau de session
  regroupe l'ecosysteme et ses membres (`↳`), affiche `—` dans « ici » pour une
  `conception` ou un ecosysteme (plus de faux `absent d'ici`) et signale un
  `part_of` sans cible valide. `lib/vaults` : `resolve_projects` gagne une 6e
  colonne, `project_warnings` et `render_project_rows` sont nouvelles.
  `aikit:registering-a-project` : critere de partage et fiche d'ecosysteme (quatre
  sections) ; `aikit:understanding-need` : un membre mene a la fiche de son
  ecosysteme. Convention de citation `repo#ADR-0001` (`VOCABULARY.md`).

## 0.18.3
- **Ligne `Skill:` optionnelle dans l'en-tete de tache.** `aikit:writing-plans`
  nomme le skill a invoquer pour une tache d'UI (ex. `frontend-design`) ;
  l'implementeur le lit dans `plan.md` et l'invoque s'il est disponible,
  sinon l'ignore.

## 0.18.2
- **Evenement `need` dans le journal optionnel.** Apres confirmation de la
  route (`aikit:understanding-need` §4), l'orchestrateur ajoute une ligne
  `{"event":"need",...}` a `AIKIT_EVENTS_FILE` (type, projet, feature, session,
  ts), meme garde que le reste du journal ; re-emise a la reprise d'un need dans
  une nouvelle session. `aikit:executing-plans` precise que la ligne de rapport
  porte `"event":"report"`.

## 0.17.2
- **Signature aiKit dans toutes les sessions.** Le stub de demarrage porte la
  regle (trailer `Co-Authored-By: aiKit <noreply@aikit.dev>`, pied de PR
  `Generated with aiKit`) : elle n'existait que dans le `CLAUDE.md` de ce depot,
  donc une session sur un autre projet signait du nom du modele.

## 0.17.0
- **Forme fixe du vault.** Chaque projet porte `heuristics.md`, `_notes/` et
  `_archive/` des son inscription (`aikit:registering-a-project`, bloc
  `# skeleton:`) ; `_global/heuristics.md` et `_notes/` a la racine. Le
  prefixe `_` est reserve : aucun slug de need ne commence par `_`.
- **Un need fini part sous `<projet>/_archive/<need>/`** a la phase 6, apres
  verification humaine ; `plan.md` et les traces vont dans `traces/`, sauf si
  la cloture ecrit « rien a retenir » (`aikit:verifying-completion`).
  `bin/scoped` et `aikit:understanding-need` l'y retrouvent, apres
  `<projet>/<need>/`, et le signalent comme archive avant toute reprise.
- **`candidates.md` remplace par des notes** : un fichier par idee sous
  `<projet>/_notes/`, frontmatter `type: note`, corps *Observed / Where / Cost*.
- **Quatre types de need** : `feature`, `diagnostic`, `etude`, `refonte` ; le
  stub de phase 1 ecrit `type:` (`VOCABULARY.md`, `aikit:understanding-need`).
- `aikit:writing-specs` : lien mort `[[status]]` retire, `type:` et
  `abandonne` dans l'en-tete requis.

## 0.15.0
- **Dispatch orchestrateur -> implementer reduit a quatre choses** : la tache
  (pointeur vers `plan.md`), les fichiers cibles, la commande de test, au
  plus une contrainte. Plus d'extraction de brief separe, plus de fichier de
  rapport persistant — l'implementer lit `plan.md` lui-meme avec ses propres
  outils.
- **Retour sous-agent hermetique en trois lignes** (`STATUS`/`FILES`/`TEST`).
  Les quatre statuts `DONE`/`DONE_WITH_CONCERNS`/`BLOCKED`/`NEEDS_CONTEXT`
  fusionnes en `SUCCESS`/`FAILURE` ; `plan.md` devient le seul registre entre
  deux tentatives.
- **Budget de tentatives ramene a deux, sans escalade automatique de
  modele.** Au-dela, arbitrage humain plutot qu'une troisieme tentative ou
  un modele plus cher choisi seul. `run-workspace`, `task-brief` et
  `review-package` supprimes ; le diff part directement en dispatch
  (`BASE..HEAD`, jamais `HEAD~1`, qui tronque une tache multi-commits).
- **`aikit:reviewer` passe a sonnet par defaut** — opus reste une escalade
  manuelle : audit securite, ou deux echecs consecutifs de revue.
- **Decision de revue binaire `APPROVE`/`REJECT`.** `REJECT` reserve au bug
  fonctionnel, a la violation de spec ou a la regression de securite ; le
  reste (style, naming, micro-optimisation) devient une suggestion non
  bloquante sous `### Non-blocking suggestions`.
- **L'orchestrateur ne lit plus massivement le code pour specifier** —
  delegation a `aikit:explorer` sur une question fermee, synthese bridee a
  30-50 lignes, zero bloc de code brut.
- **`heuristics.md`** (`vault/<project>/` et `vault/_global/`) : regles
  empiriques capees a 50 lignes, lues en cascade avant le plan, ecrites
  uniquement par l'orchestrateur a la cloture (phase 6), une puce
  `Condition -> Action` par piege reellement rencontre.
- `scripts/doctor` : sortie des portes en echec plafonnee (25 lignes pour
  les portes 1-3, 30 pour le bloc deja filtre de la porte 9) au lieu d'un
  dump brut.
- L'etat d'avancement (plan + statut + cases a cocher) reste dans
  `vault/<project>/<feature>/plan.md` — jamais dans le depot du projet.

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
