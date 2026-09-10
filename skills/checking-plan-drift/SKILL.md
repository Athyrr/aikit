---
name: checking-plan-drift
description: Use after every task completes - compares the files the task declared against what git actually changed, turning silent plan drift into an immediate, mechanical signal
---

# Checking Plan Drift

A plan is not respected because everyone intended to respect it. It is
respected because deviation is **visible**.

**Announce:** "Using checking-plan-drift to verify task N against the plan."

## When

After every task completes and before its review closes — while the fix is
still cheap. Drift found three tasks later has already been built on.

## How

From the project repository root:

```bash
<skill-dir>/scripts/plan-drift PLAN_FILE TASK_N BASE [HEAD]
```

`BASE` is the commit recorded before dispatching the implementer. **Never
`HEAD~1`** — it silently drops all but the last commit of a multi-commit task,
and a partial diff reports CLEAN on drifted work.

Exit status: `0` clean, `1` drift, `2` the plan cannot be checked.

## The three verdicts

**CLEAN** — the diff matches the declaration. Proceed to review.

**EXTRA — touched but not declared.** The task needed something the plan did
not foresee. That is the definition of an unplanned dependency: an **upward**
failure. Route with `aikit:handling-blockers`. Do not wave it through because
the code looks right — the code is not the question; the plan's accuracy is.

The honest exception: a file the whole plan needed and no single task owns
(a lockfile, a generated manifest). Name it in Global Constraints so it stops
surfacing as drift.

**UNTOUCHED — declared but not touched.** Either the task is incomplete, or the
plan named a file the work did not need. Both are real findings. Decide which,
**in writing**, in the ledger. An untouched test file almost always means the
test was never written.

## Exit 2 — the plan cannot be checked

The task has no `**Files:**` block. This is not a tooling problem to work
around; it is a plan defect. Every task declares the files it touches, because
that one declaration does three jobs: it enables this check, it bounds the
implementer's scope, and disjoint file sets are what make parallel dispatch
safe. Fix the plan.

## Projects with no repository

A project with no git repository has no diff to compare against, so this check
is unavailable. Compare against the declared list by
hand, or record the gap explicitly in `status.md`. Do not report a task as
verified on a check that never ran.

## Why this works at all

Because aiKit writes nothing inside the project repositories. Specs, plans,
ledgers and scratch all live under `work/`, so **every file in the diff is
production code by construction**. Method artifacts can never show up as false
drift.

## Red flags

| Thought | Reality |
|---|---|
| "The extra file is obviously needed" | Then the plan was wrong. Say so and go up. |
| "I'll use HEAD~1, it's easier" | Multi-commit tasks report CLEAN while drifted. |
| "I'll note the drift and keep going" | Drift compounds. The next task builds on it. |
| "The task has no Files block, skip the check" | Then add the Files block. That is the fix. |
