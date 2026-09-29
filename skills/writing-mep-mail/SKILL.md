---
name: writing-mep-mail
description: Use when the human partner asks for a release mail, a MEP mail, or a mise en recette / mise en production follow-up for a project
---

# Writing a MEP Mail

One mail per release. The rules below are shared by every project; the layout
and the source of the changes belong to the project's own template.

## Steps

1. Name the project. Read `<vault>/<project>/templates/mep-mail.md`. No
   template: ask the human partner for the sections, write the mail, then
   offer to save it as that template.
2. Recette or production: if the human partner did not say, ask. Never guess.
3. Collect the changes with the source the template names, since the last
   MEP. Unknown starting point: ask.
4. Fill the template's layout. Return the mail alone.

## Shared rules

- **Recette or prod.** Recette: subject `[MEP recette] - <Project>`, intro
  `Mise en recette de <Project> : voici les changements apportés.`
  Production: subject `[MEP] - <Project>`, intro `Mise en prod de <Project> :
  voici les changements apportés.` The word "recette" appears only in a
  recette. A template that carries its own intro sentence overrides this one.
- **Readers are functional and sales people, not developers.** One line per
  visible effect, in their words. No jargon, no setting values ("agrandi", not
  "+25 %"). A technical fix becomes its effect: "les données remontent de
  façon plus fiable".
- **Bold** the section titles and the keywords the template marks. Nothing
  else: no emojis, no section outside the template.
- End with `Cordialement`. No name: the mailbox adds the signature.
- Drop merges, docs, CI, chores and refactors with no visible effect. Merge
  commits that touch the same behaviour into one line.
- Never repeat a change already announced in a previous MEP.
- Statuses (deployed, in progress, upcoming) come from the human partner,
  never from you.
- A version number appears only if the project's template asks for one.

## What a project template holds

`<vault>/<project>/templates/mep-mail.md`, three parts only:

1. **Layout**: the sections specific to this project.
2. **Source of the changes**: where to read them (commit range, base branch,
   version bump) and whether the mail carries a version.
3. **A validated example.**

Shared rules do not go there; a rule written in two places drifts.
