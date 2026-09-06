---
name: aikit
path: aikit
summary: la methode elle-meme - plugin Claude Code (skills, archetypes, hook, fiches projet) qui gouverne toutes les sessions du workspace
---

# aikit

## 1. Identity

| | |
|---|---|
| Git root | `aikit/` |
| Base branch | `master` |
| Remote | aucun **pour l'instant** — publication prevue (voir Traps) |
| Version | lue dans `.claude-plugin/plugin.json` |
| Installation | scope **user**, marketplace `aikit-local`, une seule entree |

## 2. What it is

Le plugin qui porte la methode : les skills (des prompts, pas de la doc), les
six archetypes d'agents avec leur modele fixe, le hook de demarrage qui injecte
la skill d'entree en entier, et les fiches projet que ce document illustre.

Editer ce depot change le comportement de **toutes** les sessions du workspace,
y compris celle en cours. C'est le seul projet du workspace dont le produit est
le processus.

## 3. Load before working

| The task touches | Read | ~tok |
|---|---|---|
| travailler sur ce depot | `CLAUDE.md` | 511 |
| le contrat de la methode, les phases | `skills/using-aikit/SKILL.md` | 1 387 |
| creer ou editer une skill | `skills/writing-skills/SKILL.md` | 6 520 |
| la boucle d'execution, le ledger, les rounds | `skills/subagent-driven-development/SKILL.md` | 8 516 |
| ajouter ou editer une fiche projet | `skills/registering-a-project/SKILL.md` | 1 139 |
| les contrats de dispatch des archetypes | `agents/*.md` (les 6) | 3 001 |
| la face publique de la methode | `README.md` | 2 126 |

**Ne jamais lire l'ensemble des skills** : `skills/*/SKILL.md` totalise
**43 791** tokens. La ligne que vous cherchez est dans une seule d'entre elles.

## 4. Domain agents

| Agent | Owns | Does not own | Model |
|---|---|---|---|
| `claude-code-guide` | la mecanique du harness : skills, agents, hooks, MCP, manifestes de plugin, resolution des noms de dispatch, contraintes d'installation | le contenu de la methode, ses phases, ses regles | son propre tier — passer `opus` explicitement |

Il n'est **pas** declare dans ce depot : il vient du harness. Une fiche est la
facon de declarer un expert, pas la seule facon qu'il en existe un — la phase 3
ne se saute que quand aucun expert n'est disponible.

Les six archetypes (`aikit:explorer`, `implementer`, `planner`, `reviewer`,
`spike`, `verifier`) sont le **produit** de ce depot, pas ses experts. Ne pas
les dispatcher pour juger la mecanique du harness.

## 5. Completion criterion

Pas de suite de tests. Deux portes mecaniques, depuis la racine du depot :

```bash
cd /home/adam_adhar/ezytail-workspace/aikit
claude plugin validate .                      # le manifeste de marketplace SEULEMENT
claude plugin validate ./skills --strict      # les skills
claude plugin validate ./agents --strict      # les archetypes
CLAUDE_PLUGIN_ROOT=$PWD bash hooks/session-start | python3 -m json.tool > /dev/null
```

Les quatre doivent sortir en 0. Mesure du 2026-08-21 : les quatre passent.

**`claude plugin validate .` ne valide QUE le manifeste de marketplace.** Il ne
lit aucun `SKILL.md` ni `agents/*.md` — verifie, il l'annonce lui-meme dans sa
sortie. Les deux invocations `--strict` sont indispensables et separees ; s'en
passer, c'est n'avoir aucune garde sur les fichiers qu'on edite le plus.

**Une troisieme verification ne s'automatise pas** : un changement de plugin ne
prend effet qu'en session neuve, donc seule une nouvelle session prouve que la
methode charge sans erreur. Tant qu'elle n'a pas eu lieu, le verdict honnete est
`GATES_PASS — human verification required`. Jamais `PASS`.

## 6. Traps

- **Une liste `tools:` est exhaustive** : elle exclut les `mcp__*` **et**
  `Skill`. Un agent qui doit atteindre la toolbelt n'en declare aucune.
  `implementer` n'en porte pas — c'est delibere.
- **Le nom de dispatch porte le prefixe** : `aikit:planner`, jamais `planner`.
  Les agents de domaine d'un projet ne le portent pas.
- Pour un agent, le `name:` du frontmatter et le nom de fichier doivent
  **rester egaux**. Les 6 le sont aujourd'hui.
- **Un changement de plugin ne prend effet qu'en session neuve — mais rien
  d'autre ne le filtre.** `aikit-local` est un marketplace `directory` pointant
  sur cet arbre : le harness charge le plugin depuis l'arbre, jamais depuis la
  copie de `~/.claude/plugins/cache/`. Une edition non commitee part en
  production a la session suivante. Mesure du 2026-09-06 : ce cache est fige
  sur `6556902` et n'a pas `skills/handling-secrets/`, alors que le preambule
  injecte cite `aikit:handling-secrets` — present seulement ici, non suivi.
- `skills/using-aikit/SKILL.md` est injecte **en entier** a chaque session et
  apres chaque compaction. Le plafond de ~125 lignes n'est pas cosmetique.
- **Scope user uniquement.** Deux scopes = deux copies dont une seule se met a
  jour. `bin/deploy [patch|minor|major] "msg"` bump, valide, commit, reinstalle.
- **Pas de chemin de merge vers superpowers** — deliberé. Derive de superpowers
  (MIT, Jesse Vincent) ; garder la mention dans `LICENSE`.
- **aiKit sera versionne et publie.** Ne pas ecrire ce depot comme de l'outillage
  prive. Les faits propres au workspace vont dans `projects/*.md`, pas dans les
  skills ni le README. Et `projects/*.md` est lui-meme la frontiere non tranchee :
  ces fiches decrivent de l'infrastructure privee depuis l'interieur du depot qui
  sera publie. A regler avant le premier push public.
- Les artefacts de la methode n'entrent jamais dans les depots des projets :
  ils vivent sous `work/`, qui est un vault Obsidian.
