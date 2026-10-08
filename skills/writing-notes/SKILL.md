---
name: writing-notes
description: Use only when the human partner explicitly asks to set an idea aside - "mets ça de côté", "note pour plus tard", "put this aside", "note that for later". Never on your own initiative
---

# Writing Notes

A **note** is one idea to study later, one file. It is not a need, a task or a
TODO line (`VOCABULARY.md`, rows *note* and *candidate*). Writing one opens no
feature directory, no spec, no plan, and does not change the work in progress.

Triggered by an explicit request only. If you merely think something is worth
keeping, say so in one line and let the human partner decide. A document to keep (a decision, a design account, a source digest), not an idea to study, is a reference: use `aikit:capturing-reference`.

## Steps

1. **Search first.** Grep the keywords in `<project>/_notes/` and in `_notes/`
   at the vault root.
   - Same idea: complete that note, create nothing.
   - Neighbouring idea: create the new note and link both ways with `[[slug]]`.
   - Unsure which: ask, one line.
2. **Place it.** `<vault>/<project>/_notes/<slug>.md`, or `<vault>/_notes/<slug>.md`
   when the idea would be another project. Slug in kebab-case.
3. **Write it** in the human partner's language:

   ```markdown
   ---
   type: note
   project: <project>
   origin: <the need slug, or "session du <date>">
   created: <today>
   ---

   # <title>

   - **Observed:** what was seen, with the facts that would be lost.
   - **Where:** the files or surfaces it touches.
   - **Cost:** what handling it would take.
   - **Related:** [[other-note]] (optional)
   ```
4. **Stop.** Do not commit the vault. Confirm the written path in one line.
