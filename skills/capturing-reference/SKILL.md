---
name: capturing-reference
description: Use only when the human partner explicitly asks to keep a document as a reference in the vault, or to update one already kept - "garde ce document en référence", "mets à jour la référence", "keep this as a reference", "refresh that reference". Never on your own initiative, never for an idea to study later (that is a note)
---

# Capturing Reference

A **reference** is a document kept in the vault so that later sessions can read
it: a design brief, a paper, an outside source (`VOCABULARY.md`, row
*reference*). It is not a note and not a heuristic. This skill only writes;
reading goes through the project file's "Load before working" table, then the
`INDEX.md`.

Triggered by an explicit request only. If you merely think something is worth
keeping, say so in one line and let the human partner decide. Never use the
tool's own auto-memory for it, and never write to `heuristics.md`.

## Is it a reference?

Ask: **if the system changes, does this document become false?**

- Yes: it documents the system. The vault keeps a pointer only (`owner: depot`
  or `externe`).
- No (intent, decision, the outside world): a reference, `owner: vault`.

## Create

1. Propose to the human partner the level and the `owner`; do not decide alone.
   Level: `<project>/_reference/`, `<ecosystem>/_reference/` or
   `_global/_reference/`. One folder per level, never inside a feature folder.
2. Create `_reference/` if missing, then write the document with the
   frontmatter below.
3. Add its row to `INDEX.md` in the same operation (create `INDEX.md` with its
   header if missing).
4. `owner: vault`: store the document as is, no digest. `externe` or `depot`:
   a digest of 1 to 2 k tokens; an optional raw copy goes in `raw/`, read on
   demand by a subagent (`aikit:budgeting-context`). Only when the source is
   public or supplied as text.

```yaml
---
type: reference
project: <project | ecosystem | global>
owner: vault | externe | depot
scope: project | feature:<slug>
summary: <one line: what it is>
status: actif | obsolete
created: <date>
updated: <date>
sources: <URL or path in the repo>    # externe and depot only
read: <when to read this document>     # required for externe and depot
checked: <date last checked against the source>   # externe and depot only
---
```

## Update

- `owner: vault`: edit in place (git keeps the history), refresh `updated:`;
  refresh `summary:` and the INDEX row if the nature of the content changed.
- `owner: externe` or `depot`: re-read the source, rewrite the whole digest,
  set `checked:` to today.
- Wholesale replacement: delete the old reference and create a new one, or set
  `status: obsolete`.

## INDEX.md

No frontmatter (so it stays out of the `project:` views). Maintained by this
skill alone. Template, verbatim:

```markdown
# <level> — references

Une session qui voit une reference contredire le reel le signale a l'humain et ne la corrige pas seule ; la correction passe par `aikit:capturing-reference`, sur demande.

| Document | owner | A lire quand | ~tok | Note |
|---|---|---|---|---|
| `<file>.md` | vault \| externe \| depot | <reading condition> | <measured> | feature archivee |
```

- `~tok` is measured, never estimated:
  `python3 -c "import os;print(os.path.getsize('FILE')//4)"`.
- Note "feature archivee" only for a `scope: feature:<slug>` whose feature is
  archived; the reference stays in place.

## Before handing back

The file sits in the right `_reference/`, its INDEX row exists, `type` is
`reference` (never `note`). Do not commit the vault. Confirm the path in one
line.
