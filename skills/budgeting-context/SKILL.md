---
name: budgeting-context
description: Use when dispatching anything or about to read a large file - decides what gets loaded, by whom, and in which context, so the orchestrator never carries what a disposable context could have read
---

# Budgeting Context

**The rule everything else follows:**

> **Big reads happen in contexts that get thrown away.**

A subagent reads 27,000 tokens, distills them into a 300-token finding, and
dies. The orchestrator keeps the 300. That asymmetry is the entire context-budget
strategy — everything below is its application.

## Three tiers

| Tier | What | Paid |
|---|---|---|
| **Resident** | the method preamble, the project registry table, every skill and agent *description* | every session, ~4k |
| **On invocation** | a skill's *body* | once, in the context that invoked it |
| **On demand, disposable** | project docs, domain-agent bodies, MCP tool schemas, source code | in a subagent that dies with it |

The harness already works this way — a skill costs ~40 tokens resident and
~3,400 when invoked; MCP tools expose names and defer schemas. **Do not fight
it by pre-loading "just in case".**

## Who loads what

| Phase | Context | Loads | Must NOT load |
|---|---|---|---|
| 1 Understand | orchestrator | the project's registry file (~500) | code, project docs |
| 2 Specify | orchestrator + `aikit:explorer` | explorer reads code and docs, returns a finding | orchestrator reads nothing itself |
| 2.5 Impact | the project's domain expert | its own prompt + the docs of its area | orchestrator receives the `## Impact` section appended to `spec.md`, nothing else |
| 3-4 Plan | `aikit:planner` | spec (with its Impact section), the doc **slices** the registry routes to | the whole doc; the orchestrator does not re-read |
| 5 Execute | one `aikit:implementer` per task | its brief, and only its brief | the plan, other tasks, session history |
| 5b Review | `aikit:reviewer` | the diff and the brief it must satisfy | the conversation |
| 6 Verify | `aikit:verifier` | the registry's command and its output | anything else |
| Diagnostic | a scoped process | its MCP servers and routing skills | the orchestrator loads none of it |

## The orchestrator's context budget

At any moment it should hold roughly:

```
resident (preamble + registry)   ~4k
the plan                         ~2-4k
the ledger / status               ~1k
────────────────────────────────────
                                 ~10k
```

**If the orchestrator is above that, something was read in the wrong place.**
The usual culprits: a project doc opened "to check something", a subagent's
full report pasted instead of its path, a diff read inline.

## Reading a large document

Never open a large doc whole. The project's registry file carries a routing
table: *this kind of task → that section*. Read the slice.

`example-service/EVENT_FLOWS.md` is 27,400 tokens, but its sections run from 165
to 3,981. A task on inbound orders needs `Subject Key Convention` plus
`1. Inbound Order Flows` — about 4,600. Six times less, for strictly
more relevance.

If the section you need is not in the routing table, read the table of contents
first, then the section. Never the file.

The registry's **Load before working** table routes to the project's docs *and*
to the vault's own artifacts — a finished chantier's `status.md` is often the
cheapest answer to "how does this work and why is it like that".

## Handing work over

- Pass **paths, not contents**. A report's path costs 20 tokens; its body costs
  thousands and stays resident for the rest of the session.
- A dispatch prompt describes one task, never the session's history.
- What comes back is a conclusion plus the evidence that carries it — the full
  trace goes to a file.

## Red flags

| Thought | Reality |
|---|---|
| "I'll read the CLAUDE.md to be safe" | 8,900 tokens for context you may not need. Read the slice the registry names. |
| "Let me paste the report so I have it" | You have the path. That is having it. |
| "I'll load everything up front, it's simpler" | It is simpler for one turn and worse for every turn after. |
| "The subagent should see the whole plan" | It needs its task. The plan is the orchestrator's job. |
| "I'll check the code myself, it's quicker" | Quicker now, resident forever. Dispatch `aikit:explorer`. |
