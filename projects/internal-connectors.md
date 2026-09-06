---
name: internal-connectors
path: internal-connector-dm-cmdb-rfx-odoo-drive-api
summary: 34 scripts Python ETL autonomes synchronisant CMDB, Datamart, Odoo, ClickUp, Reflex, Shopify - sans tests, credentials en dur, la plupart ecrivent en production
---

# internal-connectors

## Identity

| | |
|---|---|
| Git root | `internal-connector-dm-cmdb-rfx-odoo-drive-api/` |
| Base branch | `master` |
| PR base | `master`, **par `recette`** — les 6 PR de l'historique viennent toutes de `recette`, jamais d'une branche de feature |
| Branches | `master`, `recette`, `dev` |
| Remote | `github.com/Ezytail/…` (org Ezytail) |
| Livraison | push `master` → GH Actions (runner self-hosted) → `ezytail/internal-connectors:latest` sur DockerHub → cron / K8s Job |

Le nom du répertoire est historique et ne décrit plus le périmètre : il n'y a
pas de connecteur « drive-api ». L'image Docker, elle, s'appelle
`internal-connectors` — d'où le nom de cette fiche.

## What it is

Dépôt de **34 scripts Python ETL autonomes** (~200 000 tokens de code), sans
package, sans framework, sans tests. Chaque script est un exécutable
point-à-point entre deux systèmes internes EzyGroupe, lancé par cron dans un
conteneur, un script par run.

Systèmes reliés : **CMDB** (Directus, REST + GraphQL) · **Datamart**
(PostgreSQL `datamart.ezytail.com:5555`) · **Odoo** (odoorpc / JSONRPC+SSL) ·
**ClickUp** (API v2) · **Reflex** (API HCO + accès DB) · **Ketra** (Mongo via
WinRM) · **Shopify** · **Google Sheets** · **Discord** (webhooks d'alerte).

Les répertoires sont nommés par **destination**, pas par source.

## Load before working

Le code total pèse **199 737 tokens**. Ne jamais ouvrir un répertoire entier :
un script est autonome, il se lit seul.

| The task touches | Read | ~tok |
|---|---|---|
| n'importe quoi ici | `CLAUDE.md` + `README.md` | 2 454 |
| un flux précis | **le seul script concerné** | 500–20 000 |
| build / image / CI | `Dockerfile` + `docker-compose.yml` + `.github/workflows/production.yml` | 850 |
| dépendances | `requirements.txt` | 44 |

Coût d'un répertoire entier, si la tentation vient — c'est la mesure qui dit
non :

| Répertoire | Flux | Scripts | ~tok |
|---|---|---|---|
| `dm_connector_python/` | → Datamart | 11 | 76 137 |
| `cmdb_connector_python/` | → CMDB | 11 | 54 476 |
| `odoo_connector_python/` | ClickUp → Odoo (temps passés) | 2 | 38 963 |
| `discord_connector_python/` | alerting backlog Ketra/Reflex | 4 | 10 794 |
| `tech_scripts/` | vacuum / purge PostgreSQL | 3 | 9 369 |
| `shopify_connector_python/` | Odoo → Shopify | 1 | 5 398 |
| `reflex_to_ezyconnect_connector_pyhton/` | emplacements GEI *(typo dans le nom)* | 1 | 2 439 |
| `reflex_hco_connector_python/` | reset password via GSheet | 1 | 2 159 |

Les cinq plus gros scripts, à dispatcher plutôt qu'à lire :
`clickup_to_odoov18_temps_passes_cmdb_enhanced.py` (19 941),
`clickup_to_odoo_temps_passes_cmdb_enhanced.py` (19 021),
`ezyconnect_to_dm_synchronisation_cmdb_enhanced.py` (12 553),
`odoov18_to_dm_accountmoveline.py` (11 022),
`clickup_to_cmdb_projects.py` (10 954).

**Deux générations de code cohabitent**, et l'anatomie change du tout au tout :

- **« enhanced »** (`clickup_to_cmdb_projects`, `clickup_to_odoo*_temps_passes`,
  `run_purge`, `vacuum_full_*`) — docstring d'en-tête avec changelog versionné,
  blocs `# ===`, une classe client par système (`ClickUpClient`, `CMDBClient`,
  `OdooClient`), une classe orchestratrice (`ProjectSynchronizer`, `Connector`),
  `main()`, logging, alertes Discord, `--dry-run`. C'est le modèle à suivre.
- **procédurale** (`odoo_dm_*`, `odoov18_to_dm_*`, `odoo_cmdb_*`,
  `odoov18_to_cmdb_*`, `rfx_dm_*`) — script linéaire, pas de fonctions,
  connexions ouvertes au niveau module (voir Traps).

## Domain agents

**Aucun.** Pas d'expert domaine enregistré pour ce projet.

Le domaine métier (comptabilité Odoo, référentiel article Reflex, modèle de
données CMDB) n'est documenté nulle part dans le dépôt : il est implicite dans
les mappings de champs de chaque script. Une question métier se répond en
dispatchant un `aikit:explorer` sur **le script concerné**, jamais de mémoire.

## Completion criterion

**Il n'y a pas de suite de tests.** Aucun `test_*`, aucun `pytest.ini`,
`pyproject.toml`, `.flake8`, `tox.ini` ni `.pre-commit-config.yaml` — vérifié
2026-09-02. La phase 6 n'a donc pas de commande qui rende un verdict complet.

Le seul portail automatisable, exécuté depuis la racine du dépôt :

```bash
find . -path ./.git -prune -o -name '*.py' -print0 | xargs -0 python3 -m py_compile
```

Mesuré le 2026-09-02 : **34/34 compilent, exit 0**. Nettoyer derrière soi —
`.gitignore` ne couvre pas `__pycache__` (voir Traps) :

```bash
find . -name '__pycache__' -not -path './.git/*' -exec rm -rf {} + 2>/dev/null
```

Le second portail est le `--dry-run`, mais il ne couvre que **10 scripts sur
34** et il **exige le réseau et des credentials de production** :

```
clickup_to_cmdb_projects · ketra_to_cmdb_conf_activity_exist
reflex_to_cmdb_type_support · ezyconnect_to_dm_synchronisation_cmdb_enhanced
clickup_to_odoo_temps_passes_cmdb_enhanced
clickup_to_odoov18_temps_passes_cmdb_enhanced
reflex_to_ezyconnect_gei_emplacement · purge_crapro_ligne_orpheline
run_purge · vacuum_full_all_db_sync_cmdb
```

**Le verdict honnête sur ce projet est `GATES_PASS — human verification
required`.** Jamais `PASS`. La compilation ne dit rien du comportement, et les
24 autres scripts n'ont aucun mode d'essai : les faire tourner, c'est écrire
dans un système réel.

## Traps

- **`--env=recette` est ignoré par 25 scripts sur 34.** `CLAUDE.md` annonce
  « recette par défaut pour la sécurité » : c'est vrai pour 9 scripts
  seulement. Les 25 autres codent `https://cmdb.ezytail.cloud` **en dur** —
  la production. Passer `--env=recette` à l'un d'eux ne protège de rien et
  donne l'illusion inverse. Vérifier avant chaque exécution :
  `grep -n "cmdb\.ezytail\.cloud\|CMDB_ENV" <script>`.

- **Le préfixe `odoov18_` ne dit pas quel Odoo est visé.** Les 4 scripts
  `odoov18_*` pointent tous sur `erp.ezytail.com` / db `app` — le host que
  `CLAUDE.md` présente comme la v15. Le host v18
  (`ezytail.gestion18.mind-and-go.net`) est commenté partout. Le commit
  `fec274e` (2026-08-04) a renommé `ezytail_prod` → `app` **uniformément sur
  les variantes v15 et v18**, ce qui laisse penser que la migration a ramené la
  v18 sur le host historique et que la table de `CLAUDE.md` est périmée — non
  confirmé auprès de l'équipe. Dans les deux lectures, la conséquence est la
  même : **lire la ligne `url =` active du script**, ne jamais déduire la cible
  de son nom.

- **8 scripts se connectent au niveau module, avant tout parsing d'arguments.**
  `odoo.login()` et `psycopg2.connect()` s'exécutent à l'import : lancer le
  script *avec n'importe quel argument, y compris pour voir son aide*
  authentifie contre l'Odoo et le PostgreSQL de production. Concernés :
  `odoo_dm_accountmoveline` (l.21), `odoo_dm_productproduct` (l.19),
  `odoov18_to_dm_accountmoveline` (l.31), `odoov18_to_dm_productproduct`
  (l.25), `odoo_cmdb_hr_employee` (l.38), `odoo_cmdb_res_partner` (l.71),
  `odoov18_to_cmdb_hr_employee` (l.46), `odoov18_to_cmdb_res_partner` (l.116).
  Les lire, jamais les exécuter pour « voir ».

- **Tous les secrets sont en dur dans le code versionné.** Zéro `os.environ`.
  Clé API ClickUp, mots de passe CMDB, identifiants Odoo, credentials WinRM,
  webhooks Discord — en clair, et dans tout l'historique git. S'y ajoute un
  compte de service Google committé :
  `reflex_hco_connector_python/reflexconnector-ee499e8c84be.json`. Ne jamais
  recopier une de ces valeurs dans un rapport, un artifact ou un message : elle
  sortirait du dépôt. Une rotation ne nettoie pas l'historique.

- **3 répertoires sur 8 ne sont pas dans l'image Docker.** Le `Dockerfile` ne
  `COPY` que `cmdb_`, `dm_`, `odoo_`, `reflex_hco_` et `tech_scripts`.
  `discord_connector_python/`, `shopify_connector_python/` et
  `reflex_to_ezyconnect_connector_pyhton/` n'existent pas dans le conteneur :
  leurs scripts ne tournent qu'en local, et le README qui promet le contraire
  se trompe.

- **Les paires v15/v18 sont des copier-coller, pas des variantes.**
  `odoo_cmdb_hr_employee.py` (638 l.) et `odoov18_to_cmdb_hr_employee.py`
  (647 l.) divergent de **15 lignes**. Idem productproduct (35),
  accountmoveline (77), temps_passes (76). `ketra_mongo_winrm.py` est dupliqué
  à l'identique (même md5) dans `cmdb_connector_python/` et
  `discord_connector_python/`. Toute correction métier doit être appliquée dans
  les deux fichiers — le vérifier explicitement à chaque tâche.

- **`.gitignore` ne contient que `.DS_Store`.** Deux `.pyc` sont trackés
  (`cmdb_connector_python/__pycache__/*.cpython-313.pyc`). Tout `python3 -m
  py_compile` en crée d'autres, non ignorés, qui polluent le `git status` et
  faussent `aikit:checking-plan-drift`. Nettoyer après le portail de
  compilation.

- **La docstring peut contredire le code qu'elle documente.** Vérifié sur
  `discord_connector_python/ketra_reflex_orders_backlog.py` : l'en-tête annonce
  une jointure `$lookup` sur `IDORDER` (l. 35) et des lots de 10 activités
  (l. 42) ; le code joint sur `ProductionKeys.Order` (l. 216) et fait des lots de
  8 (l. 261). Les deux divergences sont silencieuses — `IDORDER` porte la même
  valeur, mais n'est indexé nulle part : le résultat est juste, le flux est ~60×
  plus lent. Dans un dépôt sans tests, la docstring n'est vérifiée par rien.
  **La lire pour l'intention, jamais pour un fait technique.**

- **Le changelog vit dans la docstring du script**, pas dans le message de
  commit — avec numéro de version, et parfois la mention d'une version buguée à
  ne pas utiliser (`clickup_to_cmdb_projects.py`, v1.3.2). Modifier un script
  « enhanced » sans incrémenter sa version et ajouter son entrée casse la seule
  traçabilité qui existe.

- **Aucun `allow:` dans la frontmatter, volontairement.** Un `bin/scoped` non
  supervisé n'approuve rien ici, et c'est le comportement voulu : la quasi
  totalité des scripts écrit en production sans garde-fou.

> [!danger] Secrets : mesuré le 2026-09-03 — **25 fichiers Python sur 34**
> Aucun `.env`, aucun gestionnaire, rien dans `.gitignore`. Exposés en clair et
> dans tout l'historique git : le **compte CMDB nominatif**
> (`ketra_reflex_orders_backlog.py`, constante `CMDB`), le **compte
> `Administrator` du serveur Ketra** (`ketra_mongo_winrm.py`), les **deux
> webhooks Discord**, les identifiants **Reflex API** et **Oracle**.
>
> Invoquer `aikit:handling-secrets` avant toute intervention touchant ces
> fichiers. Et se souvenir que l'URL CMDB est câblée sur la **production** :
> `--env=recette` ne la change pas.
