---
name: ezyflow-delivery-board
path: ezyflow-delivery-board
summary: front Next.js listant les livraisons a preparer et en preparation - PAS ENCORE SOUS GIT
---

# ezyflow-delivery-board

## Identity

| | |
|---|---|
| Git root | **none — not a repository yet** |
| Base branch | n/a |
| Remote | n/a |

## What it is

Next.js front listing deliveries *to prepare* and *in preparation*, all
contexts merged, read from the Ezytail data API — the same one the production
viewer uses (`viewer.ezyflowng.ezytail.cloud`).

## Not under version control yet

This changes how the method runs here, concretely:

- `aikit:using-git-worktrees` and `aikit:finishing-a-development-branch` do not
  apply. Do not try to branch.
- **The plan-drift check (`git diff --name-only` against the files a task
  declared) is unavailable.** Compare against the task's declared file list by
  hand, or accept the gap and say so in the status file.
- The recovery map after a compaction is normally "the ledger plus `git log`".
  Here **the ledger is the only memory.** Write to it more often, not less.
- There is no rollback. A destructive edit is final — read before overwriting.

Raise `git init` with the human before starting anything substantial; it is
cheap and restores all of the above.

## Load before working

`README.md`. Needs `.env.local` (copy from `.env.example` and fill).

## Domain agents

None.

## Completion criterion

```bash
npm run typecheck      # tsc --noEmit
npm test               # pretest compiles src/lib, then node --test tests/
npm run lint
```

## Traps

- `pretest` compiles `src/lib/statuses.ts` and `src/lib/resolve.ts` into
  `.test-build/` before the suite runs. A test failing on a stale `.test-build`
  is not a test failure — remove the directory and run again.
- `.env.local` is required and not committed. A missing one fails at runtime,
  not at build: the symptom looks like an API problem.
- No repository, so nothing above can be recovered by `git checkout`. Read
  before overwriting.
