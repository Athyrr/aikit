---
name: understanding-need
description: Use at the start of every request - names the project, reads its registry file, classifies the request, and opens or resumes the feature directory before any other work
---

# Understanding the Need

Phase 1. The intake gate. Nothing else runs before it.

**Never delegate this phase to a subagent.** A subagent cannot ask a question,
so a delegated understanding is an invented one.

**Announce:** "Using understanding-need to route this request."

## 1. Name the project

Every request targets one project of the workspace. The session context lists
them. If the request does not make the project obvious, ask — do not guess from
a filename.

**Read that project's registry file before anything else.** It carries the base
branch, the conventions to load, the domain agents, the completion criterion,
and the traps. Working on a project whose registry file you have not read is a
process error, not a shortcut.

## 2. Classify the request

| The request is | Route |
|---|---|
| Something is broken / behaves wrong | `aikit:systematic-debugging` — you are diagnosing, not building |
| … and the diagnosis needs the live systems | the toolbelt's MCP servers must be connected. If they are not, say so and stop: the session has to be relaunched with `aikit/bin/ezy --tools`. Do not guess at production state from the code. |
| Build, add, change, remove behaviour | `aikit:brainstorming` — continue to phase 2 |
| "How does X work / where is Y" | Answer it. No feature directory, no spec. |
| Too unclear to classify | Ask one question. Do not open a directory on a guess. |

A debugging request that turns out to need a code change becomes a feature
request **after** the root cause is known — not before. A fix designed from a
symptom is a guess with a plan attached.

## 3. Open or resume the feature directory

```
work/<project>/<feature>/
```

at the workspace root. `<feature>` is a short kebab-case slug of what is being
built, stable for the whole life of the work.

**Look before you create.** If the directory already exists, this is a
resumption:

1. Read `status.md` first — it says where the last loop stopped and why.
2. Read `spec.md` and `plan.md` if they exist.
3. Re-enter at the phase the status file names. Do not restart from phase 1
   because restarting is easier than reading.

A resumed feature whose `status.md` names an unresolved blocker resumes at
`aikit:handling-blockers`, not at execution.

## 4. Confirm before moving on

State back, in three lines: the project, the feature slug, the route, and
whether this is new or a resumption. Get agreement, then continue.

## Red flags

| Thought | Reality |
|---|---|
| "The project is obvious, skip the registry" | The registry holds the completion criterion. You cannot finish without it. |
| "I'll create the directory and figure out the name later" | The slug is the identity of the work. Renaming it orphans the artifacts. |
| "There's a status.md but I know what to do" | The status file is the previous loop's report. It knows things you don't. |
| "Let me look at the code first" | Code tells you what is. The human tells you what should be. |
