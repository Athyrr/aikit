---
name: delegating-to-a-perimeter
description: Use when a question needs a project's own MCP servers or skills - opens that project's perimeter in a separate process and works inside it, instead of loading its tools into the orchestrator
---

# Delegating to a Perimeter

**Announce:** "Using delegating-to-a-perimeter to work inside <project>."

## When

The question needs a project's **own tools or knowledge**: the live state of an
order, why a delivery is stuck, whether a merged fix is actually deployed,
anything the toolbelt's MCP servers answer. You cannot answer it from the code.

## Why a process, and not a subagent

An Agent-tool subagent runs **inside your session**. It inherits the MCP
servers your session connected at startup; it cannot connect its own, and it
cannot re-scope itself to a directory. Dispatching `aikit:explorer` "into" a
project loads none of that project's skills or agents.

A **process** launched with `cwd = <perimeter>` loads that directory's
`.mcp.json`, its skills and its agents natively — and the subagents *it*
dispatches inherit them in turn. All measured.

Two of those three now travel on their own, since the toolbelt is installed in
the workspace: `.mcp.json` lookup walks **up** the tree, so any session in the
workspace already holds the six servers. Skills do not travel — they are read
from the session's own project root, and `ezyflow-tools` only exists at the
workspace root.

So what a process still buys you is precise: **the routing skills**, and a
context you throw away. You see a conclusion, not forty tool calls.

## The shape of the plan

The perimeter is a dependency, so it is task 1 and everything hangs off it:

```
Task 1  Open the <project> perimeter          ← creates the session
Task 2  <the actual investigation>            ← resumes it
Task 3  <the next step, informed by task 2>   ← resumes it
```

The session persists between tasks, so tasks 2..n keep everything task 1
established — routing, vocabulary, server ids — without paying discovery again.

## How

```bash
aikit/bin/scoped <project> <feature> "<instruction>"
aikit/bin/scoped <project> <feature> @work/<project>/<feature>/brief.md
```

The first call opens the session and prints its id; later calls resume it. The
session id is kept in `work/<project>/<feature>/.session` — delete that file to
start a clean perimeter. The directory it runs in is the registry's
`perimeter:`, falling back to `path:` — they differ when a project's tools are
installed somewhere other than its own repository.

Permissions come from the project's registry frontmatter (`allow:`, `deny:`),
never from a blanket bypass. A tool that is neither allowed nor denied simply
blocks: nobody is there to approve it. **Widening the perimeter means editing
the registry, deliberately** — not passing a flag in the heat of an
investigation.

## What to send, and what comes back

**Send** one question and the context it cannot infer: the reference, the
environment, what has already been ruled out. Not your session's history.

**Require back**: the conclusion in three lines, the two or three pieces of
evidence that carry it, and the path of a full trace written to
`work/<project>/<feature>/diagnostic-N.md`. Everything else stays over there.

That trace carries the same frontmatter as any artifact — `project`, `feature`,
`created`, `updated`, plus `phase: diagnostic` and `status: done` once the
conclusion holds. `work/` is a vault; a note without those properties is
invisible to it.

Tell it to use the project's own routing skills — on the toolbelt,
`ezyflow-tools` routes to the right server and `ezy-expert` holds the domain
vocabulary. A perimeter agent that ignores them is guessing at tool names.

## When the diagnosis says the code must change

It stops being a diagnosis. Write the finding to the feature's spec input and
re-enter at phase 1 for the **target** project — the perimeter agent never
edits code, and the toolbelt writes nothing back to the ecosystem.

## Red flags

| Thought | Reality |
|---|---|
| "I'll just dispatch a subagent into the toolbelt" | Subagents inherit your session. They cannot re-scope, so they get its MCP but never a project's skills. |
| "I'll dispatch `aikit:explorer`, it has the MCP tools" | It does not. An agent that declares a `tools:` list sees no `mcp__*` tool at all — measured. |
| "Faster to load the tools here" | Then forty tool calls land in your context and stay there. |
| "I'll pass the whole conversation as context" | Send the question and the facts. History is not context. |
| "It's blocked, I'll add --dangerously-skip-permissions" | The block is the registry telling you the perimeter is too narrow. Edit the registry. |
| "I'll open a fresh session each task" | Then each task re-derives what the last one learned. Resume. |
