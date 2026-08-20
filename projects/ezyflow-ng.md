---
name: ezyflow-ng
path: ezyflow-ng
summary: plateforme d'integration evenementielle .NET 8 / NATS entre Shopify, Odoo, Reflex et Ketra
---

# ezyflow-ng

## Identity

| | |
|---|---|
| Git root | `ezyflow-ng/` |
| Base branch / PR base | `release` |
| Remote | `git@github.com:Ezytail/ezyflow-ng.git` |

## What it is

Event-driven integration platform on .NET Aspire orchestrating data flow
between e-commerce (Shopify), ERP (Odoo), WMS (Reflex) and warehouse (Ketra)
systems, over NATS. Backend: .NET 8 microservices under `src/Connectors/`.
Frontend: Next.js 16 dashboard under `src/Front/`.

## Load before working — read the slice, never the file

`EVENT_CHAINS.md` is ~27,400 tokens and `CLAUDE.md` ~8,900. Opening either
whole spends more than the entire method costs. Both are cleanly sectioned, so
route to what the task needs. Line ranges are given because they are stable
enough to seek with `sed -n 'A,Bp'`; if they have drifted, read the file's
headings first, then the section.

**Always, for any event-chain work** — `EVENT_CHAINS.md` § *Convention de
Subject Keys et Champs de Liaison*, l.25-280 (~2,300). Nothing about a chain
parses without it.

| The task touches | Read (in addition) | ~tok |
|---|---|---|
| Shopify orders | `EVENT_CHAINS.md` § 1, l.703-895 | 2,300 |
| Odoo orders | § 2, l.896-1041 | 1,230 |
| Odoo status changes | § *Changements de Statut Odoo*, l.281-599 | 3,980 |
| silent substate flows | § *Flows Silencieux*, l.600-680 | 1,120 |
| Ketra (fulfillment, states, parcels, prep) | §§ 3-6, l.1042-1195 | 1,210 |
| product sync | § 7, l.1196-1520 | 2,845 |
| stock and inventory | § 8, l.1521-1856 | 3,560 |
| Reflex preparation | § 9, l.1857-1894 | 375 |
| partners, invoices, tracking, procurement | §§ 10-13, l.1895-2126 | 1,735 |
| an event that chains to nothing | § 14, l.2127-2209 | 755 |
| architecture, projections, NATS model | `CLAUDE.md` § *Key Architecture Concepts*, l.13-85 | 1,035 |
| build, tests, running a connector | `CLAUDE.md` § *Common Development Tasks*, l.86-333 | 1,660 |
| a trap, a convention, a gotcha | `CLAUDE.md` § *Important Notes*, l.334-560 | 4,175 |
| the Next.js dashboard | `src/Front/CLAUDE.md` | 2,550 |

A Shopify-order task therefore costs ~4,600 tokens of documentation instead of
27,400 — six times less, for strictly more relevance.

## Domain agents

None. Dispatch generic archetypes, and give them the slice above rather than
the file.

## Completion criterion

`dotnet test` green. Eight test projects under `src/Tests/`.

**Run it from `src/`** — the solution is `src/EzyFlow.slnx` and there is no
solution file at the repository root, so the bare `dotnet test` shown in
`CLAUDE.md` only resolves from `src/`.

```bash
cd src && dotnet test
cd src && dotnet csharpier .    # formatting gate
```

## Traps

- **NATS consumer naming** is `{ThisConnector.Name}-{FlowClassName}`, with
  irregularities that break the pattern (`Odoo.PriceSink` →
  `connectors-odoo-price-list-sink`). Never derive a consumer name by rule
  alone — check the existing one.
- **`docs/superpowers/specs/` and `docs/superpowers/plans/` (30 files, dated
  2026-05 to 2026-07) are not this project's method.** They are leftovers from
  another contributor, incomplete against the features actually shipped. Read
  them as history if useful; never treat them as the authority for current
  work, and never extend the series.
