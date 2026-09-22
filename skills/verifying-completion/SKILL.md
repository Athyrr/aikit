---
name: verifying-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - requires running verification commands and confirming output before making any success claims; evidence before assertions always
---

# Verifying Completion

## Overview

**Core principle:** Evidence before claims, always.

**Violating the letter of this rule is violating the spirit of this rule.**

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

## The Command Is Not Yours To Invent

**The command that proves the claim is written in the project's registry file**
(`vault/projects/<name>.md`, "Completion criterion"). Run that one, as written.
An equivalent-looking substitute is not evidence — `dotnet test` from the wrong
directory, or a per-tool suite run from the repository root, fails or passes for
reasons that have nothing to do with the work.

**Some projects in this workspace have no automated suite.** Their registry
file says so. There, the Iron Law does not relax — it changes shape: run the
build and lint gates, then hand the human the manual verification steps from
the spec, and claim `GATES_PASS — human verification required`. Never `PASS`.
A verdict you cannot establish is worse than no verdict, because it stops
anyone else from looking.

Dispatch `aikit:verifier` for this when you want the measurement separated from the
agent that did the work.

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?  (the registry says)
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

Once step 5 passes, close the artifact: set `status: done` and `updated` in
`vault/<project>/<feature>/spec.md`. It is the only thing that tells the vault
this feature is finished; a passing test suite nobody recorded leaves it
looking in flight.

## Closing `vault/<project>/<feature>/plan.md` — the happy path too

Its frontmatter used to be written only when a cycle went **up**. On the happy
path nobody wrote it, and the file was deleted at the end of the plan — so
after a successful run there was neither a record nor a state.

Phase 6 closes it, before deleting it. Before the verdict is reported:

- `status:` becomes `done`, `phase:` becomes `verify`, `updated:` gets today's
  date.
- The verdict goes in verbatim — the command, and its output.
- **What the verification does not cover** is named explicitly, as a list. Green
  gates say nothing about a screen nobody looked at. A completion verdict that
  hides its blind spots is worse than no verdict, because it is believed.
- Harvested candidates are listed, with a pointer to `candidates.md`.

Phase 1 initialises `vault/<project>/<feature>/plan.md`, `aikit:routing-failures` writes to it
when a cycle goes up, and phase 6 closes it before the file is removed. It is
never absent after a successful run, until the run itself is over.

## Writing to `heuristics.md` — orchestrator only, at close, one bullet

`vault/<project>/heuristics.md` is a living, append-only record of empirical
rules and traps for this project, capped at 50 lines. Two rules govern it:

- **Only the orchestrator writes to it, and only here, at phase 6.** A
  subagent never writes to it — heuristics earn their place by surviving to
  the end of a cycle, not by seeming true mid-task.
- **Write one bullet, and only if this cycle hit an unexpected trap** — a
  fact about this project nothing already written down would have told you,
  that cost real time to discover. A clean run that hit no surprises adds
  nothing; padding the file with things everyone already knew is how a
  50-line cap stops meaning anything.

Format, one line:

```markdown
- <Condition> -> <Action or thing to avoid>
```

If the file is at or near 50 lines, the new bullet earns its place only by
being more load-bearing than the oldest one — replace, don't just append.

A rule about this **machine** (OS, shell, environment) rather than this
project belongs in `vault/_global/heuristics.md` instead, under the same two
rules.

## Candidates — what survives the need

A **candidate** is a real problem that nothing is waiting on. It is not a
derived need — a missing decision the current cycle needs in order to continue,
which routes `out` — see `aikit:routing-failures` — and it is not a review
finding the human partner ruled acceptable after the attempt budget was spent
(that is **parked**, and it dies with the plan).

They live at **project** level, durable:

```
vault/<project>/candidates.md
```

**Classify by subject, record the origin.** A need's own documents — `spec.md`,
`runs/` — are built to stop being read: a `status: done`, then an `rm -rf`.
Writing something there that has to survive is burying it.

Every entry carries four things, and the fourth is what keeps the file alive:

```markdown
## <subject>

- **Observed:** what actually happened, concretely.
- **Where:** file, command, or the moment it showed up.
- **Cost:** what handling it would take. An estimate, not a promise.
- **Origin:** `from: <need-slug>, cycle N`
```

**The cost estimate is mandatory.** Without it two entries can never be weighed
against each other, and a file that cannot be arbitrated stops being read.

Harvesting them is part of phase 6, not a nicety: a candidate noticed during
execution and never written down is a problem discovered twice.

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

## Red Flags - STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!", etc.)
- About to commit/push/PR without verification
- Trusting agent success reports
- Relying on partial verification
- Thinking "just this once"
- Tired and wanting work over
- **ANY wording implying success without having run verification**

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ compiler |
| "Agent said success" | Verify independently |
| "I'm tired" | Exhaustion ≠ excuse |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |

## Key Patterns

**Tests:**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**Regression tests (TDD Red-Green):**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**Build:**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**Requirements:**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**Agent delegation:**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

## When To Apply

**ALWAYS before:**
- ANY variation of success/completion claims
- ANY expression of satisfaction
- ANY positive statement about work state
- Committing, PR creation, task completion
- Moving to next task
- Delegating to agents

**Rule applies to:**
- Exact phrases
- Paraphrases and synonyms
- Implications of success
- ANY communication suggesting completion/correctness
