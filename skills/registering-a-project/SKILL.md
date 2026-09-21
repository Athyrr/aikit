---
name: registering-a-project
description: Use when adding a project to the workspace registry or editing an existing entry - the frontmatter contract the tooling parses, and the six sections a registry file must carry
---

# Registering a Project

One file per project, at `<workspace>/vault/projects/<name>.md`. It is **data,
not plugin code**: the hook reads it from the source tree, so an edit is live in
the next session with no deploy.

## What a registry file is for

It **routes**. It never duplicates what the project already documents — it
points at the slice, and adds the three things a project cannot know about
itself:

1. how **this method** decides the work is finished;
2. which model tier to use, and where the project's own agents get it wrong;
3. what perimeter it lives in — its tools, its permissions, its boundaries.

Everything else belongs in the project's own docs, and the registry sends you
there.

## Frontmatter — parsed by tooling

Get this wrong and things fail silently: the hook drops the row, the launchers
skip the project.

| Field | Required | Read by | Meaning |
|---|---|---|---|
| `name` | yes | hook | identifier; must equal the filename. The primary key: a repo can be renamed or transferred without the file moving |
| `repo` | no | hook, `bin/ezy`, `bin/scoped` | normalized `host/org/repo`, ssh and https alike. Absent means the project has no repository |
| `dir` | no | hook, launchers | where the working directory is. Relative to the **located repo root** when `repo` is set, to the search root otherwise; defaults to `name`. `repo` and `dir` are orthogonal — a project can need both (a repo whose working tree is a subdirectory) |
| `kind` | no | hook | `depot` (implicit default) or `conception` |
| `summary` | yes | hook | one line; **the only always-on part** |
| `perimeter` | no | `bin/scoped` | directory the scoped process runs in, **relative to the located project**; defaults to `.` |
| `allow` | no | `bin/scoped` | space-separated tools pre-approved for an unattended process |
| `deny` | no | `bin/scoped` | space-separated tools withheld from it |

**Single-line values only.** The parser is a one-line `awk` — no YAML lists, no
block scalars, no quotes needed.

`summary` deserves care: it is injected into every session in the workspace, and
it is what a request gets matched against. Write it so someone who knows the
domain can tell, from that line alone, whether their question belongs here.

## The six sections, in order

**1. Identity** — a small table: git root, base branch / PR base, remote. If the
project is not under version control, say so here and say what it costs (no
worktrees, no drift check, no rollback).

**2. What it is** — two to four lines, in domain terms. Enough to confirm the
routing was right.

**3. Load before working** — the routing table. This is the section that makes
`aikit:budgeting-context` work:

```
| The task touches | Read | ~tok |
```

Measure the costs, do not guess them. For a large document, give section names
and line ranges, and state the total so the reader sees what they are avoiding.

**4. Domain agents** — a coverage table: what each owns, what it explicitly does
not, its model. Or `None.` — which is also information.

**5. Completion criterion** — the exact command, from the exact directory. Where
there is no automated suite, say so, give the gates that do exist, and state
what verdict is honest: `GATES_PASS — human verification required`, never
`PASS`.

**6. Traps** — what silently breaks work here. A trap earns its place if
someone competent would get it wrong without being told.

## A project with no repository — `kind: conception`

Phase 2 is exactly where projects that do not exist yet are born. A conception
project is a project: same file, same six sections, three fields it cannot have.

- **Section 1 Identity** states the absence and its cost: no worktree, no
  `aikit:checking-plan-drift`, no rollback, no diff to review.
- **Section 5 Completion criterion** carries no command. The criterion is **the
  spec is agreed**; phase 6 reads as the human's review of `spec.md`. The honest
  verdict is `SPEC ACTEE`, never `PASS`.
- `repo:` is absent; the project is located by directory name, or by `dir:`.

Its `summary:` is injected every session like any other. That is the point: a
project nobody can route to is a project whose wrong description goes
uncorrected.

`dir` degrades rather than guesses. Both cases resolve to **absent**, never to
an approximation:

- a `dir` that does not exist inside the located repo — the fiche is wrong, and
  a wrong fiche must be visible rather than silently resolve to the repo root;
- an absolute `dir` — it is joined to the repo root, so it cannot match.

Nothing is written to stderr: the hook runs at every session start.

## Costs are measured

Every token figure in a registry file was measured, not estimated. A wrong
figure is worse than none: it is trusted, and it misroutes the context budget.

```bash
python3 -c "import os;print(os.path.getsize('FILE')//4)"          # a whole file
awk 'NR>=A && NR<=B' FILE | wc -c                                  # a section
```

## Adding one

1. Create `<workspace>/vault/projects/<name>.md`, frontmatter first.
2. Measure the project's docs; write the routing table.
3. Establish the completion criterion by **running it**, not by reading a README.
4. Check the hook picks it up:
   ```bash
   AIKIT_VAULTS=~/.config/aikit/vaults CLAUDE_PLUGIN_ROOT=<aikit clone> \
     bash <aikit clone>/hooks/session-start | python3 -m json.tool | grep <name>
   ```
5. No deploy. The registry is read from source.

## Red flags

| Thought | Reality |
|---|---|
| "I'll paste the project's conventions in" | The registry routes. Duplicated docs rot. |
| "~10k tokens, roughly" | Measure it. A wrong figure misroutes every future read. |
| "The completion criterion is in the README" | READMEs describe intent. Run the command. |
| "No traps come to mind" | Then the section says `None known.` — do not omit it. |
| "It's a YAML list, that reads better" | The parser takes one line. It will silently drop it. |
