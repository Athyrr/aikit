# aiKit — Vocabulary

One word, one meaning. This file is the authority. `aikit:writing-skills`
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
| **need** | one distinct thing the human partner wants, with its own directory | a message, a task, a phase | it has a feature slug and a directory |
| **feature** | a `need` whose type is "build or change behaviour" | a diagnostic | `type: feature` in the need's frontmatter |
| **diagnostic** | a `need` whose type is "find out why something is wrong" | a feature that has not been specified yet | `type: diagnostic` |
| **derived need** | a need opened *from* another because a behaviour decision was missing | a task, a candidate | it carries `derived_from:` and its parent carries `derived:` |
| **cycle** | one complete pass through the phases for one need | a loop, an iteration, a retry | it starts at phase 1 and ends at a verdict |
| **phase** | a numbered step of a cycle that ends in a written artifact | a step, a task | if it writes no file, it is not a phase |
| **task** | the smallest unit with its own test cycle and its own reviewer gate | a step, a phase | one brief, one file set, one commit |
| **step** | one action inside a task, 2 to 5 minutes | a pass, a round, a phase | it fits in a checkbox |
| **pass** | one traversal of the question tree in `aikit:grilling` | a round, a step | it belongs to phase 2 and nothing else |
| **attempt budget** | 3 — the cap on attempts at one task | "the budget" | counted per task, in the ledger |
| **round budget** | 5 — the cap on fix rounds after a review | "the budget" | counted per review cycle, in the ledger |
| **round** | one fix cycle after a review finding | a pass, an attempt, a `grilling` traversal | bounded by the round budget |
| **the round cap** | the point where the round budget is spent and findings are ruled | "the breaker" | it produces rulings, not retries |
| **orchestrator** | the session that holds the plan and dispatches | controller, coordinator, driver | it never implements |
| **human partner** | the person the orchestrator works with | user, client, operator | it is the only party that can approve a spec |
| **probe** | a bounded investigation answering one written question | spike, exploration, research | it returns a finding, never an implementation |
| **feasibility** | the class of request that asks whether a thing can be done | a probe, a spike | it is a request class, not an activity |
| **candidate** | a real problem that nothing is waiting on | parked, backlog, TODO | it lives at project level and survives the need |
| **parked** | a review finding ruled at the round cap | a candidate | it lives in the ledger and dies with the plan |

The word **budget**, unqualified, is retired. Write `attempt budget` or
`round budget`. Never `loop` as a technical term; write `cycle`, `round`,
or `pass` and say which.

## Two rules that follow from the terms

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
