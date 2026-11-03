# Helia — Vocabulary

One word, one meaning. This file is the authority. `helia:writing-skills`
requires a consult before any skill is created or edited.

## The naming test — four questions, in order

1. Does the name carry its meaning without a gloss?
2. Is the word already taken, anywhere in this repository?
3. Is it on the right axis — the business of the thing, never its mechanism?
4. Does its scope match its home?

A name that fails question 2 is not a near miss. It is the failure this file
exists to prevent.

## The terms

| Term | Means | Never means | Discriminant |
|---|---|---|---|
| **need** | one distinct thing the human partner wants, with its own directory | a message, a task, a phase | it has a slug, a directory, and a `type:` in its frontmatter |
| **type** | the frontmatter field saying what a need produces: `feature`, `diagnostic`, `etude` or `refonte` — `note` on a note, `reference` on a reference, or `trame` on a trame | a phase, a status | written once, in the phase 1 stub |
| **feature** | a `need` whose type is "build or change behaviour" | a diagnostic, a refonte | `type: feature`; produces `spec.md` and `plan.md`, Fast-Path or Heavy-Path |
| **diagnostic** | a `need` whose type is "find out why something is wrong" | a feature that has not been specified yet | `type: diagnostic`; produces `diagnosis.md`, no `plan.md`; becomes a `feature` once the cause is known |
| **etude** | a `need` whose type is "answer a question the session cannot hold" | a probe, a "how does X work" answered in chat | `type: etude`; produces `etude.md` (question, constats, recommandation), no `plan.md` |
| **refonte** | a `need` whose type is "reshape without changing what must not change" | a feature, a cleanup | `type: refonte`; always Heavy-Path; its spec names the invariant |
| **trame** | the document that lays out a large project split into tranches: what each delivers, what it waits for, when it is done, where it stands | a need, a plan, a decisions log | `type: trame`, one per project, at `<project>/trame.md`, never inside a need's directory |
| **tranche** | a `feature` need listed in its project's trame: an increment delivered end to end, verifiable on its own | a task, a phase, a cycle | its slug is in the `Dossier` column of `<project>/trame.md`; it opens once the tranches its `Attend` cell names are archived with `status: done` |
| **note** | one idea or finding to study later, one file | a need, a task, a TODO line | `type: note`, in `<project>/_notes/<slug>.md` — or `_notes/<slug>.md` at the vault root when it would be another project |
| **reference** | a background document kept in the vault, in one place per level, found through an `INDEX.md` | a note (an idea to study), a heuristic (a trap, a Condition -> Action bullet written at phase 6), the documentation of a system | `type: reference` in `<project\|ecosystem\|_global>/_reference/`; test "if the system changes, does this document become false?" (yes = system documentation, `owner: depot` or `externe`, the vault keeps only a pointer; no = `owner: vault`); a heuristic may point to a reference, never the reverse |
| **upstream dependency** | anything the work relies on that can change without us: a source, a base, a last seen value, declared in `## Dépendances` of a registry file or of `plan.md` | a task's `Depends on`, a `reference` of the vault, `base_sha`, a base branch | something upstream moves on its own; `helia-deps` compares it, `helia:checking-dependencies` triages it |
| **archive** | where a finished or abandoned need goes: `<project>/_archive/<need>/` | a deletion, a `done` flag alone | reached only at phase 6, after human verification; `plan.md` moves to its `traces/` |
| **derived need** | a need opened *from* another because a behaviour decision was missing | a task, a candidate | it carries `derived_from:` and its parent carries `derived:` |
| **cycle** | one complete pass through the phases for one need | a loop, an iteration, a retry | it starts at phase 1 and ends at a verdict |
| **phase** | a numbered step of a cycle that ends in a written artifact | a step, a task | if it writes no file, it is not a phase |
| **task** | the smallest unit with its own test cycle and its own reviewer gate | a step, a phase | one brief, one file set, one commit |
| **security task** | a task whose header carries `**Security:** yes` because it meets at least one of the five criteria in `helia:writing-plans` (authentication or authorisation, secrets or credentials, cryptography, untrusted input, network exposure or permissions) | a security audit, an `Effort:` annotation | a property of the task, never a dispatch parameter; absent means `no`; the orchestrator translates it through `helia:executing-plans`' escalation table |
| **human task** | a task the human partner performs, titled `[humain]` (or `[human]`) in the plan | an approval stop, a review | at least one checkbox and no Files, Agent, Model or Security; the session dispatches nothing for it, and the monitor badges it while it is the first open task |
| **step** | one action inside a task, 2 to 5 minutes | a pass, a round, a phase | it fits in a checkbox |
| **pass** | one traversal of the question tree in `helia:grilling` | a round, a step | it belongs to phase 2 and nothing else |
| **attempt budget** | 2 — the cap on attempts at one task before stopping for human arbitration | "the budget" | counted per task, in `vault/<project>/<feature>/plan.md` |
| **orchestrator** | the session that holds the plan and dispatches | controller, coordinator, driver | it never implements |
| **human partner** | the person the orchestrator works with | user, client, operator | it is the only party that can approve a spec |
| **probe** | a bounded investigation answering one written question | spike, exploration, research | it returns a finding, never an implementation |
| **feasibility** | the class of request that asks whether a thing can be done | a probe, a spike | it is a request class, not an activity |
| **candidate** | a real problem that nothing is waiting on | parked, backlog, TODO | it is written as a `note` in `<project>/_notes/` and survives the need |
| **parked** | a review finding the human partner ruled acceptable to leave, after the attempt budget was spent | a candidate | it lives in `vault/<project>/<feature>/plan.md`, is not harvested, and stays only in the archived `traces/plan.md` |
| **heuristic** | one empirical rule or trap, written as Condition -> Action | a convention, a trap (registry §6) | it lives in `<project>/heuristics.md`, or `_global/heuristics.md` for the machine; one bullet, orchestrator-written, only at phase 6 |
| **ecosystem** | a registry file with `kind: ecosysteme`: projects that share a contract or a document, with a directory for what they share | a company, a data dependency, a project with code | members declare `part_of: <ecosystem>`; the session table lists them under it, prefixed `↳` |
| **kind** | the frontmatter field of a registry file saying what sort of project it is: `depot`, `conception` or `ecosysteme` | `type` (what a *need* produces) | `kind` lives in `<vault>/projects/<name>.md`; `type` lives in a need's `plan.md` |

The word **budget**, unqualified, is retired. Write `attempt budget`. Never
`loop` as a technical term; write `cycle` or `attempt` and say which.

Write `tranche`. Never `slice` — already taken (a part of a document read in
`helia:budgeting-context`) — nor `lot`, `jalon` or `milestone`. Never
`chantier` for a project.

**A document in another repository is cited `repo#ADR-0001`** — the repository name, `#`, the identifier of the ADR, spec or ticket (`some-repo#ADR-0001`, `some-org/some-repo#105`). One form for all three.

## Levels of work, largest first

| Level | Term | Contains | Never write |
|---|---|---|---|
| 1 | project | its needs; its trame, if it is split into tranches | chantier |
| 2 | need | its cycles; a `feature` listed in the trame is a **tranche** | slice, lot, jalon, milestone (for a tranche) |
| 3 | cycle | its phases, phase 1 to a verdict | loop, iteration |
| 4 | phase | one artifact; at phase 5, the tasks | stage, step |
| 5 | task | its steps, one test cycle, one review, one commit | ticket, job |
| 6 | step | one 2–5 minute action, one checkbox | — |

The **trame** is not a level: it is the document that orders the tranches
of one project. **pass** (phase 2) and **attempt** (one try at a task) are
counts, not levels.

## The trame

`<project>/trame.md`. Frontmatter: `project`, `type: trame`, `status`
(`active`, or `done` once no tranche is left to open or in progress), `updated`.

Required headings, in this order:
- **But du projet**: what the tranches deliver together.
- **Graphe**: a Mermaid graph; `A --> B` means B opens only once A is
  archived. It is for the reader: the `Attend` column decides.
- **Vue d'ensemble**: one row per tranche,
  `Tranche | En une phrase | Attend | Parallèle possible avec | État | Dossier`.
  `Attend` names prerequisite tranches and prerequisites outside tranches
  (a decision, a measure); `Dossier` is the tranche's need slug.
- **Les tranches**: one sub-heading per tranche, with Objectif, Contenu,
  Attend, Terminée quand, État.
- **Historique du découpage**: one dated row per change to the list or the
  order, with the decision behind it.

Optional: Règles du découpage, Ordre retenu, Pistes parallèles, Après la
dernière tranche, Points ouverts.

`État` reads `à ouvrir`, `en cours (ouverte le <date>)`,
`archivée le <date>` or `abandonnée le <date>`. It summarises the tranche's
`plan.md`, which wins when the two disagree. An `abandonnée` tranche does not
meet a prerequisite. Two tranches that can both open may run in parallel.

## Three rules that follow from the terms

**A skill invoked by another skill or by an archetype must never carry
`disable-model-invocation`.** That flag removes the skill's description from
context entirely, so the invoking skill's instruction resolves to nothing and
the graft dies silently, with no error on the method's side. The flag is
reserved for entry points a human types.

**A rename moves two things, never one.** For an **agent**, the frontmatter
`name:` is authoritative and the filename is cosmetic. For a **skill**, the
model sees the **directory** name — but the old frontmatter `name:` keeps
resolving as a typed command. A directory-only rename leaves a working ghost
alias that passes every grep. Gate 10 of `scripts/doctor` is what catches it.

**The `_` prefix is reserved.** `_archive/`, `_notes/`, `_reference/` and `_global/` are the
vault's own directories. No need slug may start with `_`; `bin/scoped` refuses
one.
