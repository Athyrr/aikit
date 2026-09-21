---
name: writing-specs
description: Use after designing the solution to write the spec artifact - the contract for where it lives, what metadata it carries, and what it must contain before a plan can be argued from it
---

# Writing Specs

Phase 2. Turns the understanding reached with the human into the artifact every
later phase argues from.

**Announce:** "Using writing-specs to record the design."

`aikit:designing-the-solution` produces the design through dialogue. This skill governs
what gets written down. Run `aikit:designing-the-solution` first; do not write a spec from a
request you have not explored.

## Where it goes

```
vault/<project>/<feature>/spec.md
```

Never inside a project repository. Artifacts are local and unversioned by
design — the code repositories stay clean of method output.

## Required header

Because the artifact is **not** versioned alongside the code, it cannot rely on
travelling with the branch. It has to say what it describes:

```markdown
---
project: example-service
feature: carrier-filter
phase: spec
status: draft | agreed | done | superseded
branch: feature/carrier-filter
base_sha: 0171b92
created: 2026-08-20
updated: 2026-08-20
---

# [Feature] — Spec

Plan: [[plan]] · Status: [[status]]

> **Asked for:** "<the human's request, quoted verbatim>"
```

The verbatim quote is not decoration. Between here and a task brief the request
is reformulated four times — spec, plan, brief, dispatch — and each
reformulation is a chance to drift. Quoted once, it can be checked at every
level. Carry it into each task brief too.

`status` moves in one direction: `draft` until the human agrees, `agreed`
while the work runs, `done` once phase 6 has produced a passing verdict,
`superseded` when a later spec replaces this one. Nothing else marks a feature
as finished — `vault/` is an Obsidian vault and `aikit.base` reads exactly this
field, so a spec left at `agreed` reads as still in flight forever.

`base_sha` is the commit the spec was written against. Months later it is the
only way to tell whether the spec describes the code you are looking at. On a
project with no repository, write `base_sha: n/a (no repository)` — say it
explicitly rather than leaving it blank.

The wikilinks cost nothing and make the artifact directory navigable as a vault.

## Required content

- **Problem** — what is wrong or missing today, in your human partner's terms.
- **Goal** — one sentence. What is true when this is done.
- **Scope** — what is in. Then **explicitly** what is out. The out-list is what
  stops phase 3 from planning work nobody asked for.
- **Behaviour** — what the system does, observably. Not how.
- **Constraints** — versions, naming rules, exact strings, platform limits.
  Copy exact values verbatim; a plan that paraphrases a constraint loses it.
- **Verification** — how a human confirms this works, in this project's terms.
  Start from the registry's completion criterion. On a project with no test
  suite, this section is the only gate that exists — write it as steps someone
  can follow, not as "test manually".
- **Open questions** — anything unresolved, each with who decides.

## Before handing off

A spec with an open question is not agreed. Either resolve it with the human or
mark the affected scope out-of-scope for this pass.

Set `status: agreed`, then run **phase 2.5** before planning: dispatch the
project's domain expert consultatively — *"here is the spec: which files does
it touch, what are the traps, how would you cut it? Write no code."* — and have
it append the answer as an `## Impact` section at the end of the same
`spec.md`, not a separate file. `impact.md` does not exist: the expert's
judgement on *this* spec is inseparable from the spec it judges, and a plan
that disagrees with it is disagreeing with a section of its own spec, not a
detached document that can drift out of sync.

The expert answers what a document cannot: a judgement on **this** spec. Skip
2.5 only where the registry lists no domain expert. Then continue to
`aikit:writing-plans`.

## Red flags

| Thought | Reality |
|---|---|
| "The design is in the conversation, that's enough" | The conversation dies at compaction. The artifact is the memory. |
| "I'll fill in verification later" | Verification is what tells you the work is done. Without it there is no end. |
| "Scope is obvious, no need for an out-list" | Everything is in scope until something says it isn't. |
| "I'll paraphrase the constraint" | Exact values are exact. Copy them. |
