---
name: understanding-need
description: Use at the start of every distinct need - once per need, not once per message - names the project, reads its registry file, classifies the request, and opens or resumes the feature directory before any other work
---

# Understanding the Need

Phase 1. The intake gate. Nothing else runs before it.

**Never delegate this phase to a subagent.** A subagent cannot ask a question,
so a delegated understanding is an invented one.

**Announce:** "Using understanding-need to route this request."

## Once per need, not once per message

This phase runs **once per distinct need**, not once per message. A need
already routed stays routed: answering a clarifying question you asked during
`aikit:designing-the-solution`, confirming the feature slug, narrowing the scope,
approving a spec — all of that is *inside* the same need. Do not re-invoke this
skill for it. Continue the phase you are in.

Re-invoke it when the subject changes: another project, another feature, a
request whose route would differ (a bug report arriving mid-spec), or a
resumption across a session boundary. When a message could be either, ask —
one question is cheaper than a wrong route, and cheaper than re-running this
phase on a need it already routed.

## 1. Name the project

Every request targets one project of a registered vault. The session context
lists them. If the request does not make the project obvious, ask — do not
guess from a filename.

**Read that project's registry file before anything else.** It carries the base
branch, the conventions to load, the domain agents, the completion criterion,
and the traps. Working on a project whose registry file you have not read is a
process error, not a shortcut.

**A registry file is how an expert is declared, not the only way one exists.**
Phase 3 is skipped only when no expert is *available* — never merely because no
file names one. When the need falls in a domain the workspace has a standing
expert for, dispatch it: for work on skills, agents, hooks and plugin
manifests, that expert is `claude-code-guide`. Say which expert you used, and
say so explicitly when you found none.

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
| Something is broken / behaves wrong | `aikit:diagnosing` — you are diagnosing, not building |
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
| Route | Dispatch a light implementer directly — no `spec.md`, no `plan.md` | `aikit:designing-the-solution` — continue to phase 2 |
| Proof | `git diff --name-only` matches the file(s) you named | `aikit:checking-plan-drift` per task |

State the estimate back to the human in the same breath as the route
confirmation (step 4) — "Fast-Path, touches `foo.py` and its test" — so a
wrong guess is visible before work starts, not after.

**When in doubt, Heavy-Path.** An estimate that turns out wrong mid-flight is
not a reason to keep going on the cheap route — stop, name what you found,
and re-route through `aikit:designing-the-solution`. A Fast-Path task whose diff grows
past what was named is exactly the drift the file declaration exists to catch;
treat it the same way whether the declaration came from a plan task or from
this estimate.

**Fast-Path dispatch:** the request itself, restated with the exact files
named, is the brief — there is no plan to extract it from. Dispatch
`aikit:implementer` (sonnet by default; see `aikit:using-aikit`'s model table
for when it escalates). When it reports done, run `git diff --name-only`
yourself against the files you named in step 4 — a match is the proof: no
reviewer, no ledger, no workspace. A mismatch is drift — route it with
`aikit:routing-failures` rather than accepting a "close enough" diff.

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

**A new directory gets its `status.md` before anything else is written.** Not a
placeholder — the three things a resumption needs and cannot re-derive:

```markdown
---
project: <project>
feature: <slug>
phase: understand
status: en-cours
cycle: 1
updated: <today>
---

# <feature> — cycle state

## Re-enter at
Phase 2 — nothing specified yet.
```

Phase 6 closes it; `aikit:routing-failures` writes to it when a cycle goes up.
Between those, it is the only thing a resumed session can trust.

**A need carrying `derived_from:` is a child cycle.** It runs the phases
normally, with one restriction: it may not derive again. Depth is capped at 1,
so any new behaviour decision discovered inside it goes **up**, to the parent
named in its frontmatter — never sideways into a grandchild
(`aikit:routing-failures`).

A resumed feature whose `status.md` names an unresolved blocker resumes at
`aikit:routing-failures`, not at execution.

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
