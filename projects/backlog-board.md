---
name: backlog-board
path: ezy-pilotage-prod
summary: « Ezy pilotage prod » - tableau de bord de production du flux sortant (Next.js) remplacant la notification Discord EZT011 - les 4 collecteurs tournent contre la production, l'ecran a ete refondu au cycle 3 - depot prive Ezytail/ezy-pilotage-prod
---

# backlog-board

> [!note] L'identifiant aiKit et le nom d'usage divergent, et c'est voulu
> aiKit connaît ce projet sous **`backlog-board`** — cette fiche, le dossier
> `work/backlog-board/`, tous les `[[wikilink]]` du vault. Les humains le
> connaissent sous **« Ezy pilotage prod »**. Aligner les deux orphelinerait
> chaque lien et chaque chemin absolu des huit documents du vault : écarté au
> cycle 3, sciemment.

## Identity

| | |
|---|---|
| Git root | `ezy-pilotage-prod/` |
| Base branch | `main` |
| Remote | `Ezytail/ezy-pilotage-prod` — **privé** |
| Dernier cycle | 3 — écran refondu, fusionné le 2026-09-06 (`0cbc4a9`) |

**Le dépôt existe depuis le 2026-09-06** (cycle 3). Il est privé, et doit le
rester : il porte des chiffres de production, des noms d'activités clientes et
des chemins d'infrastructure. `.env.local` et `data/*.json` sont hors dépôt ;
`.env.example` y est, sans aucune valeur.

## What it is

Tableau de bord de production journalier — charge et avancement du flux sortant.
Il remplace la notification Discord du flux `EZT011` par une page web
consultable. Quatre collecteurs CLI écrivent des `Snapshot<T>` JSON dans `data/`,
un front Next.js les lit et les rend.

Les sources sont **Ketra** (MongoDB atteint par WinRM 5986), **Reflex** (API REST
puis base Oracle en direct) et la **CMDB Directus** pour le périmètre. Le projet
est un portage des scripts Python de [[internal-connectors]] — `ketra_mongo_winrm`
notamment.

## Load before working

Les documents vivent dans le vault, **pas dans le dépôt** :
`work/backlog-board/mvp-localhost/`. Total des 8 documents : **~36 275 tokens**.
N'en ouvrir aucun en entier sans raison.

| The task touches | Read | ~tok |
|---|---|---|
| reprendre le projet, savoir où on en est | `status.md` | 2 461 |
| implémenter un collecteur | `collectors.md` | 3 385 |
| d'où vient un chiffre affiché, quelle requête, quels champs | `data-lineage.md` | 3 909 |
| quand un chiffre est faux et de combien | `collector-warnings.md` | 2 180 |
| l'enchaînement cron → pixel, **et le glossaire** | `flow.md` | 4 812 |
| secrets, authentification, CI/CD | `security.md` | 2 482 |
| ce qui attend une **décision métier**, plus affiché à l'écran | `questions-metier.md` | ~1 900 |
| la recette du probe WinRM, les façons connues d'échouer | `spike-winrm.md` | 2 742 |

`spec.md` fait **14 302 tokens** — jamais en entier. Ses sections :

| Section | Lignes | ~tok |
|---|---|---|
| Problème | 26-40 | 177 |
| Décisions actées, et pourquoi | 41-138 | 1 651 |
| Architecture | 139-362 | 2 660 |
| La stratégie de récupération est transposée, jamais repensée | 363-415 | 679 |
| Règles métier à préserver | 416-631 | 2 876 |
| Le tableau de bord | 632-721 | 1 076 |
| Cartographie Ketra, mesurée le 2026-09-03 | 722-870 | 1 806 |
| Limites assumées | 871-886 | 198 |
| Budget de performance — exigence, pas souhait | 887-953 | 799 |
| Fidélité des données : ce qui est garanti, et ce qui ne l'est pas | 954-1014 | 786 |
| Ce que le portage doit améliorer | 1015-1034 | 261 |
| Gestion d'erreurs | 1035-1048 | 195 |
| Interface | 1049-1063 | 182 |
| Authentification | 1064-1071 | 92 |
| Hors périmètre | 1072-1079 | 100 |
| Critère de complétion | 1080-1094 | 107 |
| Inconnues restantes | 1095-1131 | 438 |

## Domain agents

**Aucun agent propre à ce projet.** Les archétypes génériques s'appliquent.

Deux ressources voisines portent le domaine, et ce ne sont pas des agents :

- La skill **`ezy-expert`** couvre Ketra, Reflex, les statuts et la CMDB Directus
  — le vocabulaire métier de ce projet. À charger pour comprendre une donnée
  source, jamais pour décider ce que le tableau affiche : ça, c'est `spec.md`.
- Le projet [[internal-connectors]] contient les scripts Python d'origine. Le
  portage transpose leur stratégie de récupération **à l'identique** — voir
  `spec.md` § *La stratégie de récupération est transposée, jamais repensée*.

## Completion criterion

Depuis `backlog-board/`, avec Node **18.19.1** :

```bash
npm run typecheck && npm test && npm run lint
```

Vérifié le 2026-09-06, après la fusion du cycle 3 : typecheck propre,
**81/81 tests**, lint sortie 0, `npm run build` 4 pages.

> [!warning] Il n'y a toujours aucun navigateur utilisable sur cette machine
> Vérifié le 2026-09-06 : le binaire Chromium de Playwright se télécharge, mais
> `libnss3`, `libnspr4` et `libasound2` manquent au système et aucun navigateur
> n'est installé — le lancement échoue. **Un rapport de sous-agent affirmant
> avoir mesuré un rendu est donc à vérifier avant d'être cru** ; c'est arrivé au
> cycle 3.
>
> Se débloque par `sudo apt-get install -y libnss3 libnspr4 libasound2t64`,
> après quoi une capture Playwright depuis un dossier hors dépôt fonctionne
> sans toucher aux dépendances du projet.

> [!warning] Ce critère ne couvre pas le rendu visuel
> Aucun navigateur sur la machine : le DOM, les chiffres et le build sont
> vérifiés, **l'apparence ne l'a jamais été**. Un verdict honnête sur une tâche
> qui touche à l'écran est `GATES_PASS — vérification humaine requise`, jamais
> `PASS`.

## Traps

**Le vault est derrière un lien symbolique vers `/mnt/c/`.** Un subagent à qui on
donne `work/backlog-board/...` ne le franchit pas et conclut que les documents
n'existent pas. **Toujours passer le chemin résolu** :
`/home/adam_adhar/ezytail-workspace/work/backlog-board/mvp-localhost/`.

**Node 18.19.1 est épinglé, ce n'est pas du conservatisme.** `winrm-client@0.0.12`
est un paquet pré-1.0, déjà pris en défaut deux fois, validé sur cette version
seulement.

**La jointure vers `ShippingHistory` est `(ActivityId, OrderName)`.** `OrderName`
seul collisionne entre activités — 3 cas sur 40 commandes courantes, mesuré. Sans
`ActivityId`, la moyenne de traversée rend **−2 822 heures**.

**Le `$lookup` vers `<TRI>.CMDCLI` se fait sur `ProductionKeys.Order`, jamais sur
`IDORDER`.** Même valeur, mais `IDORDER` n'est indexé nulle part : ~60× plus lent,
76 s contre 1,3 s.

**WinRM 5986, jamais 5985.** Toutes les mesures du 2026-09-03 sont passées par
`pywinrm` sur le 5985 — l'outil, pas la cible. Elles disent ce que Ketra
*contient* ; elles ne disent **rien** du chemin Node en production, qui n'a
jamais été essayé.

**Lecture seule sur Ketra et Reflex, sans exception.** Le canal WinRM tourne sous
`Administrator` : rien côté serveur ne restreint les écritures. Interdits, y
compris les discrets : `insert` · `update` · `remove` · `save` · `drop` ·
`createIndex` · `$out` · `$merge` · `renameCollection` · `mapReduce`.

**`data/ketra.json` et `data/scope.json` sont amorcés à la main**, pas collectés.
Ils portent depuis le 2026-09-04 un avertissement en tête de `warnings[]` qui le
dit. Ne pas le retirer tant qu'aucun collecteur ne les produit.

**Les fixtures ne sont jamais un repli automatique** (`USE_FIXTURES=true` pour les
demander), et **`null` ne s'affiche jamais `0`** — une dégradation n'est pas un
zéro.

**`npm run lint` appelle `eslint .` directement**, pas `next lint`, déprécié
depuis Next 15.3. La configuration est en flat config (`eslint.config.mjs`).
