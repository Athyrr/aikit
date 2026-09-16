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

Every request targets one project of a registered vault. The session context
lists them. If the request does not make the project obvious, ask — do not
guess from a filename.

**Read that project's registry file before anything else.** It carries the base
branch, the conventions to load, the domain agents, the completion criterion,
and the traps. Working on a project whose registry file you have not read is a
process error, not a shortcut.

A project the session table marks **absent d'ici** is registered but not cloned
on this machine. Read its registry file as usual — then say so before planning
anything that needs its code.

**Outside a vault that carries a registry**, the session context says so.
The method still applies in full — only the routing step has nothing to route
to. Establish the same four things from the project's own documentation before
planning, state that you did, and flag the completion criterion explicitly: it
is the one a registry would have given you, and phase 6 cannot run without it.

## 2. Classify the request

| The request is | Route |
|---|---|
| Something is broken / behaves wrong | `aikit:systematic-debugging` — you are diagnosing, not building |
| … and the diagnosis needs the live systems | `aikit:delegating-to-a-perimeter` — run it in its own process. Its tools are already in your session; what you are keeping out is its forty tool calls. Never guess at production state from the code. |
| Build, add, change, remove behaviour | Estimate size — see 2a below |
| "How does X work / where is Y" | Answer it. No feature directory, no spec. |
| Too unclear to classify | Ask one question. Do not open a directory on a guess. |

A debugging request that turns out to need a code change becomes a feature
request **after** the root cause is known — not before. A fix designed from a
symptom is a guess with a plan attached.

## 2a. Estimate size — Fast-Path or Heavy-Path

Before opening a feature directory, make the call:

| | Fast-Path | Heavy-Path |
|---|---|---|
| Criteria | ≤2 files, no API/contract break, no critical dependency | anything bigger, a new component, a refactor, or you are unsure |
| Route | Dispatch a light implementer directly — no `spec.md`, no `plan.md` | `aikit:brainstorming` — continue to phase 2 |
| Proof | `git diff --name-only` matches the file(s) you named | `aikit:checking-plan-drift` per task |

State the estimate back to the human in the same breath as the route
confirmation (step 4) — "Fast-Path, touches `foo.py` and its test" — so a
wrong guess is visible before work starts, not after.

**When in doubt, Heavy-Path.** An estimate that turns out wrong mid-flight is
not a reason to keep going on the cheap route — stop, name what you found,
and re-route through `aikit:brainstorming`. A Fast-Path task whose diff grows
past what was named is exactly the drift the file declaration exists to catch;
treat it the same way whether the declaration came from a plan task or from
this estimate.

**Fast-Path dispatch:** the request itself, restated with the exact files
named, is the brief — there is no plan to extract it from. Dispatch
`aikit:implementer` (sonnet by default; see `aikit:using-aikit`'s model table
for when it escalates). When it reports done, run `git diff --name-only`
yourself against the files you named in step 4 — a match is the proof: no
reviewer, no ledger, no workspace. A mismatch is drift — route it with
`aikit:handling-blockers` rather than accepting a "close enough" diff.

## 3. Open or resume the feature directory (Heavy-Path only)

Fast-Path work skips this: no feature directory, no artifacts, just the diff.

```
<vault>/<project>/<feature>/
```

in the vault that owns the project. `<feature>` is a short kebab-case slug of
what is being built, stable for the whole life of the work.

**Look before you create.** If the directory already exists, this is a
resumption:

1. Read `status.md` first — it says where the last loop stopped and why.
2. Read `spec.md` and `plan.md` if they exist.
3. Re-enter at the phase the status file names. Do not restart from phase 1
   because restarting is easier than reading.

A resumed feature whose `status.md` names an unresolved blocker resumes at
`aikit:handling-blockers`, not at execution.

## 4. Confirm before moving on

**Heavy-Path:** state back, in three lines: the project, the feature slug, the
route, and whether this is new or a resumption. Get agreement, then continue.

**Fast-Path:** state back, in one line: the project and the file(s) you expect
to touch. Get agreement, then continue.

## Red flags

| Thought | Reality |
|---|---|
| "The project is obvious, skip the registry" | The registry holds the completion criterion. You cannot finish without it. |
| "I'll create the directory and figure out the name later" | The slug is the identity of the work. Renaming it orphans the artifacts. |
| "There's a status.md but I know what to do" | The status file is the previous loop's report. It knows things you don't. |
| "Let me look at the code first" | Code tells you what is. The human tells you what should be. |
