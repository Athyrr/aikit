---
name: routing-failures
description: Use when a task fails, stalls, or surprises you - decides whether to go down (probe, fix rounds), up (re-plan, re-spec), or out (a derived need), enforces the attempt budget, and defines what an escalation must contain
---

# Routing Failures

**Announce:** "Using routing-failures to route this failure."

## The rule

> **The nature of the failure decides the direction.**

| What happened | Direction | Mechanism |
|---|---|---|
| Technical unknown — "I don't know how X works" | **down** | Probe. The plan does not move. |
| Execution error — red test, broken build, wrong output | **down** | The fix rounds, bounded by the attempt budget. |
| Unplanned dependency; files outside the task's declared set; task too large | **up** | Finish the independent tasks, then re-split or re-plan. |
| Spec ambiguous, contradictory, or silent on the case at hand | **up** | Stop now. Back to phase 2, with the human. |
| **A behaviour decision is missing, and the current work is not wrong** | **out** | **A derived need.** |

Going down is cheap. Going up is expensive. **Retrying a wrong plan is the most
expensive thing of all** — it burns the attempt budget, fills context, and
produces code that will be thrown away.

Never spend a retry on a failure that belongs upward. Two questions settle it:

1. **Would a different attempt at the same task plausibly succeed?**
   No → the failure is upward.
2. **Does fixing this require changing what the task is supposed to do?**
   Yes → the failure is upward.

## Down — the probe

`aikit:probe` is an **agent**, dispatched with the Agent tool as
`subagent_type: "aikit:probe"` — never invoked as a skill. Same for the other
archetypes. The `aikit:` prefix is part of the name: a dispatch on bare `probe`
fails with "subagent_type does not exist".

Project domain agents are **not** prefixed: `api-expert`, `schema-expert`,
`ui-expert`, `docs-expert`.

A probe answers one question. It is not a smaller version of the task.

- **One question, written down before starting.** "Does the upload endpoint
  emit a correlation id on partial batches?" — not "look into uploads".
- **Bounded**: a stated bound on attempts or tool calls, agreed before dispatch.
- **Produces a finding, not code.** Write it to
  `vault/<project>/<feature>/runs/<plan>/probe-<slug>.md`: the question, what was
  tried, what was observed (with file paths, line numbers, command output), the
  answer, and what it implies for the task.
- Any code written during a probe is thrown away. If the probe's code looks
  worth keeping, that is a signal the plan was wrong — go up.
- **The task resumes with the finding in its brief.** The probe does not
  complete the task.

If the probe exhausts its stated bound without an answer, that is not a failed
probe — it is an unplanned dependency. **Go up.**

## Down — the fix rounds

- The **attempt budget** is three unless the plan says otherwise.
- Each attempt must change something real: more context in the brief, a
  narrower target, or a probe first so the attempt stops guessing.
  Re-dispatching the same brief to the same tier is not an attempt, it is a
  coin flip charged to your attempt budget.
- `aikit:implementer` defaults to **sonnet**. **Attempts 1-2 stay on sonnet.**
  If both fail, attempt 3 escalates to **opus**. What triggers it is the ledger,
  never a per-task judgement call made in the moment: two recorded failures,
  then the tier moves. `aikit:probe` keeps the same move under the same
  argument — it too defaults to sonnet, and two recorded failures buy it the
  same rung. Escalating the model does not excuse you from also changing the
  brief — a bigger model on the same brief is still a coin flip.
- **`aikit:planner` and `aikit:reviewer` have no rung above.** Both already run
  at the top tier, so a failed planning pass or a review that missed something
  cannot be answered by escalating the model. What replaces escalation for
  those two, in order of cost: more context in the brief; a narrower target;
  a probe first, so the next pass stops guessing; **fresh eyes at the same
  tier** — a new instance with the failure written into its brief, which is not
  a coin flip when the brief has changed; and finally a higher `effort`, an
  accepted frontmatter field for plugin agents and the only genuine notch above
  once a role is already at its ceiling.
- After the attempt budget: escalate. Do not extend it silently.

## Out — the derived need

Some problems answer to neither direction. A probe is useless — **no fact
resolves it**, because what is missing is a decision about what the system
should do. And going up would throw away correct code — **the current work is
not wrong**, it simply ran into a question nobody has answered yet.

That problem goes **out**: it becomes a need of its own, with its own directory,
its own spec, and its own cycle — entered at phase 1 like any other need, not
bolted onto the one that found it.

**The lineage is written down, in both directions.** The parent's `status.md`
gets `derived: <child-slug>`; the child's `spec.md` and `status.md` get
`derived_from: <parent-slug>`. Without both, the derived need reads months later
as an orphan nobody can explain.

**Depth is capped at one.** A derived need may never itself derive. Anything
discovered inside one goes **up**, to the parent — which is the direction that
was available all along. The cap is what stops a single feature from spawning a
tree of half-specified children, each blocked on the next.

**The discriminant, answered in writing, in the ledger:**

> **Does the current cycle need the answer to continue?**

- **Yes** → derive now. The current cycle stops at the tasks that do not depend
  on the missing decision, exactly as an upward failure does.
- **No** → it is a **candidate**, not a derived need. Write it to
  `vault/<project>/candidates.md` and keep going. A candidate costs one line
  now; a derived need costs a full cycle, and deriving one the current work did
  not need is how a plan quietly doubles in size.

The answer goes in the ledger because the question is easy to re-answer
differently an hour later, under the pressure of wanting to be done.

## The attempt budget lives in the ledger, not in your head

An agent that retries does not reliably remember it is on attempt three;
compaction erases that first. **An attempt budget that is not written down
does not exist.**

Write, in the plan's ledger, before each attempt:

```
Task 4: attempt 2/3 — reviewer flagged the consumer name; re-dispatching with the naming table in the brief
```

Before starting any attempt, read the ledger's lines for this task. If you find
attempt 3 already recorded, the attempt budget is spent — escalate, even if you
have no memory of the earlier attempts.

## Up — the cycle report

Going up does not mean stopping everything.

1. **Finish the tasks that do not depend on the broken assumption.** Half a
   plan delivered is better than a plan abandoned mid-flight, and the finished
   work narrows what the next plan has to cover.
2. Do not start any task that touches the broken assumption.
3. Write the cycle report to `vault/<project>/<feature>/status.md`:

```markdown
## Cycle 2 — stopped at execution, going up to planning

**Done:** Tasks 1, 2, 5 (commits abc1234, def5678, 9012abc)
**Not started:** Tasks 3, 4 — both depend on the consumer naming decision
**What broke:** Task 3 needs a consumer name the plan derives by rule; the
rule does not hold for `Billing.PriceSink` (registry trap, example-service).
**Re-enter at:** planning — the naming step needs its own task, before 3 and 4.
**Open question for the human:** none / <the question>
```

4. Re-enter at the named phase. Carry the report; do not re-derive it.

## Escalating to the human

Escalate when the attempt budget is spent, when the spec is ambiguous, or when
the project has no automatic verdict (see the registry — on such projects the
human is the default verifier, not the last resort).

**"I'm blocked" is not an escalation.** It makes the human pay the whole cost of
reconstructing the context. An escalation contains:

- the question or decision, in one sentence;
- what was tried, and what was observed — concretely;
- the options you see, with the trade-off of each;
- **your recommendation**, and what you will do if the human says nothing.

## Red flags

| Thought | Reality |
|---|---|
| "One more try and it'll work" | Attempt four is where the method stops paying. Read the ledger. |
| "I'll just adjust the plan a bit as I go" | Silent re-planning is the drift the method exists to prevent. Go up, in writing. |
| "The spec doesn't say, I'll pick something sensible" | An unspecified case is an upward failure. Ask. |
| "A probe will sort this out" | Probes answer questions. They do not fix wrong plans. |
| "I'll note the blocker in my todo" | Todos die with the context. The ledger and status file don't. |
| "Escalating looks like failure" | Escalating late is the failure. |
