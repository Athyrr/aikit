---
name: routing-failures
description: Use when a task fails, stalls, or surprises you - decides whether to go down (probe, two fix attempts), up (re-plan, re-spec), or out (a derived need), enforces the attempt budget, and defines what an escalation must contain
---

# Routing Failures

**Announce:** "Using routing-failures to route this failure."

## The rule

> **The nature of the failure decides the direction.**

| What happened | Direction | Mechanism |
|---|---|---|
| Technical unknown — "I don't know how X works" | **down** | Probe. The plan does not move. |
| Execution error — red test, broken build, wrong output | **down** | Two attempts, bounded by the attempt budget. |
| Unplanned dependency; files outside the task's declared set; task too large | **up** | Finish the independent tasks, then re-split or re-plan. |
| Spec ambiguous, contradictory, or silent on the case at hand | **up** | Stop now. Back to phase 2, with the human. |
| A behaviour decision is missing, and the current work is not wrong | **out** | A derived need — its own directory, spec and cycle. |

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
  `vault/<project>/<feature>/probe-<slug>.md`: the question, what was
  tried, what was observed (with file paths, line numbers, command output), the
  answer, and what it implies for the task.
- Any code written during a probe is thrown away. If the probe's code looks
  worth keeping, that is a signal the plan was wrong — go up.
- **The task resumes with the finding in its brief.** The probe does not
  complete the task.

If the probe exhausts its stated bound without an answer, that is not a failed
probe — it is an unplanned dependency. **Go up.**

## Down — two attempts, then stop and ask

- **The attempt budget is two.** Each attempt must change something real: more
  context in the brief, a narrower target, or a probe first so the attempt
  stops guessing. Re-dispatching the same brief unchanged is not an attempt,
  it is a coin flip charged to your attempt budget.
- Every archetype — `aikit:implementer`, `aikit:probe`, `aikit:planner`,
  `aikit:reviewer` — runs at its one fixed model (`aikit:executing-plans`'s
  Model Selection table). There is no automatic model escalation: a second
  failure does not buy a bigger model, it buys a stop.
- **When the second attempt also fails, stop dispatching and ask your human
  partner for arbitration.** Do not open a third round, and do not keep
  inflating the dispatch history hoping the next one lands. State what was
  tried, what happened both times, and what you recommend — then wait.
- A finding the reviewer raised that you believe is wrong, or not worth
  fixing, does not consume an attempt: say so to your human partner and let
  them rule, rather than spending a dispatch to argue with the reviewer.

## The attempt budget lives in `vault/<project>/<feature>/plan.md`, not in your head

An agent that retries does not reliably remember it is on attempt two;
compaction erases that first. **An attempt budget that is not written down
does not exist.**

Write, under the task's checklist in `vault/<project>/<feature>/plan.md`, before each attempt:

```
- [ ] Task 4 — attempt 2/2: reviewer flagged the consumer name; re-dispatching with the naming table in the brief
```

Before starting any attempt, read that task's existing lines. If you find
attempt 2 already recorded and still open, the attempt budget is spent — stop
and ask for arbitration, even if you have no memory of the earlier attempt.

## Out — the derived need

Some problems fit neither direction. A probe is useless — **no fact
resolves it**, because what is missing is a decision about what the system
should do. And going up would throw away correct code — **the current work is
not wrong**, it simply ran into a question nobody has answered yet.

That problem goes **out**: it becomes a need of its own, with its own directory,
its own spec, and its own cycle — entered at phase 1 like any other need, not
bolted onto the one that found it.

**The lineage is written down, in both directions.** The parent's
`vault/<project>/<feature>/plan.md` gets a `derived: <child-slug>` line; the child's `spec.md`
and `vault/<project>/<feature>/plan.md` get `derived_from: <parent-slug>`. Without both, the
derived need reads months later as an orphan nobody can explain.

**Depth is capped at one.** A derived need may never itself derive. Anything
discovered inside one that **blocks its own progress** goes **up**, to the
parent — the direction that was available all along, since it may not derive
again. Anything that does not block its own progress is a candidate, same as
anywhere else. The cap is what stops a single feature from spawning a tree of
half-specified children, each blocked on the next.

**The discriminant, answered in writing, in `vault/<project>/<feature>/plan.md`:**

> **Does the current cycle need the answer to continue?**

- **Yes** → derive now. The current cycle stops at the tasks that do not depend
  on the missing decision, exactly as an upward failure does.
- **No** → it is a **candidate**, not a derived need. Write it to
  `vault/<project>/candidates.md` and keep going — see `aikit:verifying-completion`
  for the required format (four fields, Cost mandatory). A candidate costs one
  line now; a derived need costs a full cycle, and deriving one the current
  work did not need is how a plan quietly doubles in size.

The answer goes in `vault/<project>/<feature>/plan.md` because the question is easy to re-answer
differently an hour later, under the pressure of wanting to be done.

## Up — the cycle report

Going up does not mean stopping everything.

1. **Finish the tasks that do not depend on the broken assumption.** Half a
   plan delivered is better than a plan abandoned mid-flight, and the finished
   work narrows what the next plan has to cover.
2. Do not start any task that touches the broken assumption.
3. Write the cycle report into `vault/<project>/<feature>/plan.md`, under a `## Cycle report`
   heading:

```markdown
## Cycle 2 — stopped at execution, going up to planning

**Done:** Tasks 1, 2, 5 (commits abc1234, def5678, 9012abc)
**Not started:** Tasks 3, 4 — both depend on the consumer naming decision
**What broke:** Task 3 needs a consumer name the plan derives by rule; the
rule does not hold for `Billing.PriceSink` (registry trap, example-service).
**Re-enter at:** planning — the naming step needs its own task, before 3 and 4.
**Open question for the human:** none / <the question>
```

4. Rewrite the top-level `## Re-enter at` heading in place so it matches this
   cycle's **Re-enter at** line — the heading is the live pointer a resumed
   session trusts; the `## Cycle N` sections are history.
5. Re-enter at the named phase. Carry the report; do not re-derive it.

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
| "One more try and it'll work" | Attempt three is where the method stops paying. Read `vault/<project>/<feature>/plan.md`. |
| "I'll just adjust the plan a bit as I go" | Silent re-planning is the drift the method exists to prevent. Go up, in writing. |
| "The spec doesn't say, I'll pick something sensible" | An unspecified case is an upward failure. Ask. |
| "A probe will sort this out" | Probes answer questions. They do not fix wrong plans. |
| "I'll note the blocker in my todo" | Todos die with the context. `vault/<project>/<feature>/plan.md` doesn't. |
| "Escalating looks like failure" | Escalating late is the failure. |
