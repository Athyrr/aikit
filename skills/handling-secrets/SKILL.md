---
name: handling-secrets
description: Use before writing config, before any commit, and whenever a codebase turns out to hold credentials in clear - separates rotation from relocation, fixes what a secret may never touch, and defines the three pieces every project needs
---

# Handling Secrets

**Announce:** "Using handling-secrets to secure this credential path."

## The rule

> **A secret that has been versioned is a public secret.**

Moving it into `.env` tidies the code. **Only rotation cuts access.** These are
two different actions, and doing the first while calling it the second is the
most common failure in this area.

| You did | You achieved |
|---|---|
| Moved the value to `.env.local` | The next commit is clean |
| Rewrote git history | The value is harder to find |
| **Rotated the credential** | **The old value opens nothing** |

The first two without the third leave the door open.

## What a secret may never touch

Fixed list. No judgement calls.

| Never | Why |
|---|---|
| The conversation transcript | It is stored, and it may be summarized, shared, or pasted |
| An artifact, a report, a screenshot | Published means distributed |
| A commit, in any file, ever | Including "temporarily", including in a branch you will delete |
| A log line, an error message, a stack trace | Logs travel further than code |
| The browser — any client-side bundle | `NEXT_PUBLIC_*` and equivalents are public by definition |
| A CI environment that does not need it | Build credentials and runtime credentials are different sets |

**When you must show a credential's location**, name the file and the constant.
Never the value. When a grep would print values, mask them in the same command
(`sed -E 's/(password|token)[^,}]*/\1: <MASKED>/I'`) rather than after the fact.

## The three pieces every project needs

Copy them from a project in the workspace that already does it right, rather
than inventing a shape.

1. **`.env.example`** — every variable, **no values**, one comment per variable
   saying where the real one comes from and what it changes.
2. **`.env.local`** — the real values. Covered by `.gitignore` **before** the
   first one is written, not after.
3. **A config check** — a `REQUIRED` list read *before* the first network call.
   Without it, a missing variable surfaces as a stack trace inside an HTTP
   client, which names everything except the thing that is actually missing.

## Accounts, not just values

A rotated password on a bad account is still a bad account.

| Smell | What it should be |
|---|---|
| A **person's** address as a service login | A service account, owned by the project |
| One account for read and write | Read-only where the code only reads |
| One account across environments | One per environment — and per environment secrets, like a pinned certificate fingerprint |
| An admin account "because it works" | The narrowest role that does the job |

A nominative account fails silently the day that person changes a password or
leaves, and every action the system takes is attributed to them.

## Where this fires in the method

| Phase | What it demands |
|---|---|
| **1 — Understand** | If the codebase holds secrets in clear, that is a finding for the registry file, not a detail. Count them. |
| **2 — Specify** | The spec carries the credential inventory: which system, which account, which privilege. Rotation is a task, not a note. |
| **5 — Execute** | `.gitignore` before the first value. Review what a broad `git add` staged, every time. |
| **6 — Verify** | Secret scanning **fails** the build. A warning that nobody reads reproduces the state you were fixing. |

## When you inherit a compromised repository

Do not start by cleaning. Start by counting, then by rotating.

1. **Inventory** — how many files, which systems, which accounts. A number makes
   the problem arguable; "there are secrets in there" does not.
2. **Classify** — an admin account on a production host and a webhook URL are not
   the same emergency.
3. **Rotate**, in that order of severity.
4. *Then* relocate to `.env`, and only then consider history rewriting.

Report the inventory to your human partner **before** touching anything: they
own the decision of what to rotate first, and rotation breaks running systems.

## Red Flags

| Thought | Reality |
|---|---|
| "I'll move it to `.env` and it's fixed" | Relocation is not rotation. The old value still works. |
| "It's only the recette credentials" | Recette often shares an identity provider, a network, or a password with production. |
| "I'll paste it here so we can see it" | The transcript is storage. Name the file, not the value. |
| "It's a webhook URL, not a password" | The URL *is* the token. Anyone holding it can post. |
| "I'll delete the commit afterwards" | It was pushed. Treat it as published. |
| "Secret scanning is noisy, I'll make it a warning" | A warning changes nothing. It must fail. |
| "The CI needs the app's credentials to test" | Then the tests touch real systems, which is a second bug. |
