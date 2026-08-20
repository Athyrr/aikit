---
name: ezyflow-ng
path: ezyflow-ng
summary: plateforme d'integration evenementielle .NET 8 / NATS entre Shopify, Odoo, Reflex et Ketra
---

# ezyflow-ng

| | |
|---|---|
| Git root | `ezyflow-ng/` |
| Base branch / PR base | `release` |
| Remote | `git@github.com:Ezytail/ezyflow-ng.git` |

Event-driven integration platform on .NET Aspire orchestrating data flow
between e-commerce (Shopify), ERP (Odoo), WMS (Reflex) and warehouse (Ketra)
systems, over NATS. Backend: .NET 8 microservices under `src/Connectors/`.
Frontend: Next.js 16 dashboard under `src/Front/`.

## Load before working

`CLAUDE.md` (724 lines — the architecture reference), `EVENT_CHAINS.md`, and
`src/Front/CLAUDE.md` for frontend tasks.

## Domain agents

None yet. Dispatch generic archetypes.

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
