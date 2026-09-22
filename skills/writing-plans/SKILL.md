---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** If working in an isolated worktree, it should have been created via the `aikit:isolating-the-workspace` skill at execution time.

**Save plans to:** `vault/<project>/<feature>/plan.md`
- (Your human partner's preferences for plan location override this default)

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs while designing the solution. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## The Files Block Is Not Optional

Every task declares the exact files it touches. That single declaration does
three jobs, and dropping it silently disables all three:

1. `aikit:checking-plan-drift` compares it against `git diff --name-only` after
   the task — the only mechanical detector of a plan quietly abandoned;
2. it bounds the implementer's scope, so "I needed one more file" surfaces as a
   report instead of disappearing into a diff;
3. **disjoint file sets are what make parallel dispatch safe.** Two tasks that
   share a file are sequential, whatever their dependency block says.

If a file is needed by the whole plan and owned by no single task (a lockfile,
a generated manifest), name it in Global Constraints. Otherwise it shows up as
drift on whichever task happens to touch it.

## Task Right-Sizing

A task is the smallest unit that carries its own test cycle and is worth a
fresh reviewer's gate. When drawing task boundaries: fold setup,
configuration, scaffolding, and documentation steps into the task whose
deliverable needs them; split only where a reviewer could meaningfully
reject one task while approving its neighbor. Each task ends with an
independently testable deliverable.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use aikit:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Spec:** [path to the spec/design doc this plan implements — the plan
argues from the spec, so the spec travels with it; executors read both. When
the project has a domain expert, the spec's own `## Impact` section (phase
2.5) names the files and traps the expert saw; a Files block that contradicts
it is wrong until argued otherwise.]

## Global Constraints

[The spec's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, with exact
values copied verbatim from the spec. Every task's requirements implicitly
include this section.]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Depends on:** [task numbers, or `none`. `none` plus a disjoint file set is
what makes a task parallelisable.]

**Agent:** [a domain expert from the project's registry file when the task
falls in its area — e.g. `api-expert`. Omit for the generic `aikit:implementer`.]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**One path per line, and nothing after it.** No second path on the same line,
no prose, no parenthesis. Line ranges are allowed as a `:123-145` suffix and
are stripped before comparison. Anything else makes `plan-drift` exit 2 and
refuse to check the task — by design: a parser that guesses produced four false
`DRIFT` verdicts on one cycle, and `EXTRA` is routed as an upward failure.
Explanations go in prose **below** the block.

**Interfaces:**
- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [what later tasks rely on — exact function names, parameter
  and return types. A task's implementer sees only their own task; this
  block is how they learn the names and types neighboring tasks use.]

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — the engineer may be reading tasks out of order)
- Steps that describe what to do without showing how (code blocks required for code steps)
- References to types, functions, or methods not defined in any task

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## The fresh-eye guard — before any execution dispatch

Your Self-Review above is the author checking their own work. It catches
placeholders and type drift. It cannot catch what you did not think of, because
it is you thinking again.

**Before the handoff, dispatch `plan-document-reviewer-prompt.md` (next to this
skill) on a fresh `aikit:reviewer`, on opus.** It has not read the conversation
that produced the plan, which is the entire point.

Every diff in this method already faces a reviewer who did not write it. The
plan — the one artifact where a mistake is paid by every task downstream — had
only its author. This closes that.

Handle its findings the way a task handles review findings: fix, or rule and
record the ruling in `status.md` — the ledger doesn't exist yet at this point,
`status.md` is what a resumption can read. Do not argue with it in your own
head and move on.

**The spec gets no such guard.** Its authority comes from a human partner having
signed it. A subagent re-reading an approved spec adds nothing and invites
re-litigating a settled contract.

## Execution Handoff

After saving the plan, hand it off. **There is no choice to offer:** one skill
carries execution, and its subagent mode is the method.

**"Plan complete and saved to `<vault>/<project>/<feature>/plan.md`.**
**REQUIRED SUB-SKILL:** `aikit:executing-plans` — a fresh subagent
per task, a fresh reviewer per diff, the ledger between them. Its final section
covers the degraded inline mode for a session where subagents are unavailable.
