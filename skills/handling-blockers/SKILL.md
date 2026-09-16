---
name: handling-blockers
description: Use when a task fails, stalls, or surprises you - decides whether to go down (spike, fix loop) or up (re-plan, re-spec), enforces the retry budget, and defines what an escalation must contain
---

# Handling Blockers

**Announce:** "Using handling-blockers to route this failure."

## The rule

> **The nature of the failure decides the direction.**

| What happened | Direction | Mechanism |
|---|---|---|
| Technical unknown — "I don't know how X works" | **down** | Spike. The plan does not move. |
| Execution error — red test, broken build, wrong output | **down** | Fix loop, bounded by budget. |
| Unplanned dependency; files outside the task's declared set; task too large | **up** | Finish the independent tasks, then re-split or re-plan. |
| Spec ambiguous, contradictory, or silent on the case at hand | **up** | Stop now. Back to phase 2, with the human. |

Going down is cheap. Going up is expensive. **Retrying a wrong plan is the most
expensive thing of all** — it burns budget, fills context, and produces code
that will be thrown away.

Never spend a retry on a failure that belongs upward. Two questions settle it:

1. **Would a different attempt at the same task plausibly succeed?**
   No → the failure is upward.
2. **Does fixing this require changing what the task is supposed to do?**
   Yes → the failure is upward.

## Down — the spike

`aikit:spike` is an **agent**, dispatched with the Agent tool as
`subagent_type: "aikit:spike"` — never invoked as a skill. Same for the other
archetypes. The `aikit:` prefix is part of the name: a dispatch on bare `aikit:spike`
fails with "subagent_type does not exist".

Project domain agents are **not** prefixed: `api-expert`, `schema-expert`,
`ui-expert`, `docs-expert`.

A spike answers one question. It is not a smaller version of the task.

- **One question, written down before starting.** "Does the upload endpoint
  emit a correlation id on partial batches?" — not "look into uploads".
- **Bounded**: a stated budget of attempts or tool calls, agreed before dispatch.
- **Produces a finding, not code.** Write it to
  `vault/<project>/<feature>/sdd/<plan>/spike-<slug>.md`: the question, what was
  tried, what was observed (with file paths, line numbers, command output), the
  answer, and what it implies for the task.
- Any code written during a spike is thrown away. If the spike's code looks
  worth keeping, that is a signal the plan was wrong — go up.
- **The task resumes with the finding in its brief.** The spike does not
  complete the task.

If the spike exhausts its budget without an answer, that is not a failed
spike — it is an unplanned dependency. **Go up.**

## Down — the fix loop

- The budget is **three attempts** unless the plan says otherwise.
- Each attempt must change something real: more context in the brief, a
  narrower target, or a spike first so the attempt stops guessing.
  Re-dispatching the same brief to the same tier is not an attempt, it is a
  coin flip charged to your budget.
- `aikit:implementer` defaults to **sonnet**. **Attempts 1-2 stay on sonnet.**
  If both fail, attempt 3 escalates to **opus** — the one tier-escalation this
  method allows, and it fires only here: two recorded failures in the ledger,
  never a per-task judgement call made in the moment. Escalating the model
  does not excuse you from also changing the brief — a bigger model on the
  same brief is still a coin flip.
- After the budget: escalate. Do not extend it silently.

## The budget lives in the ledger, not in your head

An agent that retries does not reliably remember it is on attempt three;
compaction erases that first. **A budget that is not written down is not a
budget.**

Write, in the plan's ledger, before each attempt:

```
Task 4: attempt 2/3 — reviewer flagged the consumer name; re-dispatching with the naming table in the brief
```

Before starting any attempt, read the ledger's lines for this task. If you find
attempt 3 already recorded, the budget is spent — escalate, even if you have no
memory of the earlier attempts.

## Up — the loop report

Going up does not mean stopping everything.

1. **Finish the tasks that do not depend on the broken assumption.** Half a
   plan delivered is better than a plan abandoned mid-flight, and the finished
   work narrows what the next plan has to cover.
2. Do not start any task that touches the broken assumption.
3. Write the loop report to `vault/<project>/<feature>/status.md`:

```markdown
## Loop 2 — stopped at execution, going up to planning

**Done:** Tasks 1, 2, 5 (commits abc1234, def5678, 9012abc)
**Not started:** Tasks 3, 4 — both depend on the consumer naming decision
**What broke:** Task 3 needs a consumer name the plan derives by rule; the
rule does not hold for `Billing.PriceSink` (registry trap, example-service).
**Re-enter at:** planning — the naming step needs its own task, before 3 and 4.
**Open question for the human:** none / <the question>
```

4. Re-enter at the named phase. Carry the report; do not re-derive it.

## Escalating to the human

Escalate when the budget is spent, when the spec is ambiguous, or when the
project has no automatic verdict (see the registry — on such projects the human
is the default verifier, not the last resort).

**"I'm blocked" is not an escalation.** It makes the human pay the whole cost of
reconstructing the context. An escalation contains:

- the question or decision, in one sentence;
- what was tried, and what was observed — concretely;
- the options you see, with the trade-off of each;
- **your recommendation**, and what you will do if the human says nothing.

## Red flags

| Thought | Reality |
|---|---|
| "One more try and it'll work" | Attempt four is the doom loop. Read the ledger. |
| "I'll just adjust the plan a bit as I go" | Silent re-planning is the drift the method exists to prevent. Go up, in writing. |
| "The spec doesn't say, I'll pick something sensible" | An unspecified case is an upward failure. Ask. |
| "A spike will sort this out" | Spikes answer questions. They do not fix wrong plans. |
| "I'll note the blocker in my todo" | Todos die with the context. The ledger and status file don't. |
| "Escalating looks like failure" | Escalating late is the failure. |
