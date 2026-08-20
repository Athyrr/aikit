---
name: ezylive
path: ezylive/ezy_live
summary: extension navigateur (Chrome MV3 / Firefox MV2) + dashboard Next.js pour les back-offices marchands
---

# ezylive

| | |
|---|---|
| Git root | `ezylive/` — **work in `ezylive/ezy_live/`** |
| Base branch / PR base | `ezylive-next-gen` |
| Remote | `git@github.com:Ezytail/ezylive.git` |

Two front-ends over one Ezytail backend: `extension/` injects order and incident
status cards into Shopify / PrestaShop / Odoo back-offices; `web-app/` is a
standalone Next.js 15 dashboard replacing the legacy Flutter app.

## Load before working — read the slice

The docs here are already scoped, and small. Read only what the task touches.

| The task touches | Read | ~tok | Consult |
|---|---|---|---|
| anything (orientation) | `ezy_live/AGENTS.md` | 330 | — |
| the Next.js dashboard, the popup UI | `web-app/AGENTS.md` | 505 | `next_expert` |
| background / content scripts, manifests, CSP, packaging | `extension/AGENTS.md` | 925 | `extension_expert` |
| an order status that is wrong, missing or platform-dependent | `extension/AGENTS.md` | 925 | `ezy-api-expert` |
| layout, palette, status colours, popup ergonomics | `web-app/AGENTS.md` | 505 | `designer` |

Reading all three costs 1,760 tokens — cheap enough that orientation plus one
scoped file is always the right call. Never read an expert's own file to
"get its knowledge": dispatch it instead, so the knowledge arrives applied and
in a context that dies.

## Domain agents

They live in `ezylive/ezy_live/.claude/agents/` and are only reachable if the
session was launched with `--add-dir ezylive/ezy_live` (the default for
`aikit/bin/ezy`).

| Expert | Owns | Explicitly not |
|---|---|---|
| `next_expert` | App Router, RSC vs `use client`, React Query, Zustand, Shadcn/Tailwind, `lib/api`, static-export navigation, `delivery.ts`/`order.ts` status logic | extension background/content scripts, backend query bodies |
| `extension_expert` | background/content/popup architecture, message passing, service-worker lifecycle, both manifests, CSP, platform handlers, Liquid templates, build and packaging | API request bodies and token semantics, the Next.js front |
| `ezy-api-expert` | `chrome/src/backgroundPage.ts`, deliveryng/orderng/reject requests, match/project shapes, CMDB/Trigram fallback, anomaly overlay, `/user/me`, 401 revocation, `EZL_FT_*` allowances | UI and Liquid rendering, build and manifests, web-app React Query |
| `designer` | mockups, ergonomics, design system, `ezy.*` and `status.*` tokens, mobile-first layout, the ~400×600 popup constraint | logic and data implementation |

They cross-route explicitly in their own descriptions. Follow that routing
rather than picking by intuition — and **use them at phase 2.5** to answer what
the docs cannot: given this spec, which files, which traps, how to cut it.

| Expert | Its own model | For implementation work |
|---|---|---|
| `extension_expert` | opus | dispatch as-is |
| `ezy-api-expert` | opus | dispatch as-is |
| `next_expert` | sonnet | **override to opus** |
| `designer` | sonnet | **override to opus** |

These files belong to the ezylive repository, not to aiKit — do not edit them
to fix the tier. Override on the dispatch instead. They cross-route explicitly
("NE PAS utiliser pour X → agent Y"); follow that routing rather than picking
by intuition.

## Completion criterion

**There is no automated test suite.** `AGENTS.md`: *"verification is manual
(load the unpacked extension, or run a dev server)."*

Consequence for the method: the fix loop has no automatic verdict, so it
cannot terminate on its own. **On this project the human is the default
verifier, not the escape hatch.** Budget the retries anyway, then escalate.

Minimum gate before asking for manual verification:

```bash
cd web-app  && npm run lint && npm run build
cd extension && npm run build:nextjs && npm run build:chrome-scripts
```

## Traps

- **Never hardcode a URL.** Runtime URLs, env and version live in
  `extension/shared/config.ts` (`CURRENT_ENV`: production ↔ recette). The
  web-app reads it through the `@shared` alias.
- **The version is duplicated in five places plus both manifests**
  (`extension/package.json`, `extension/popup/package.json`,
  `web-app/package.json`, `shared/config.ts`, both manifests). A release bumps
  all of them together — a task that touches one must touch all.
- `extension-build/` and `deploy/` are generated. Never hand-edit.
- External backends: `api.ezytail.com`, `cmdb.ezytail.cloud`, `auth.ezytail.com`.
