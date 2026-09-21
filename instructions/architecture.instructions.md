---
applyTo: "**"
description: "Architecture and UX baseline: layering, Azure topology, Material UI, theming, and data tables."
---

# Architecture instructions

## Principles

- Start with the simplest architecture that meets the requirement; add layers only when a constraint demands it.
- Separate concerns: presentation → application/service → domain → infrastructure. Dependencies point inwards.
- Stateless services; keep state in a managed data store (Azure SQL, Cosmos DB, PostgreSQL flexible server).
- Design for failure: timeouts, retries with exponential backoff and jitter, circuit breakers, idempotent handlers.
- Make everything observable: structured logs, metrics, and traces to Application Insights.

## Reference topology (Azure)

```text
Browser ── Azure Front Door / Static Web Apps ── Next.js or Vue.js frontend
                                                     │ (Entra ID token)
                                                     ▼
                                    Azure Container Apps / App Service  (Python FastAPI or ASP.NET Core)
                                                     │ managed identity
                          ┌──────────────────────────┼──────────────────────────┐
                          ▼                          ▼                          ▼
                    Azure SQL / Cosmos DB      Azure Key Vault          Azure Storage / Service Bus
```

All service-to-service calls use **managed identities** — no keys, no connection strings.
See [`standards/azure/azure.md`](../standards/azure/azure.md).

## Frontend baseline

- **Framework**: Next.js (App Router, TypeScript) or Vue.js 3 (`<script setup>`, TypeScript).
- **Design system**: Material UI (MUI for React, Vuetify for Vue). Professional, consistent, and appealing:
  a single theme file owns palette, typography, spacing, and component overrides. No ad-hoc inline styling.
- **Dark/Light mode**: every application exposes a theme toggle button in the app bar.
  - Default to the OS preference (`prefers-color-scheme`).
  - Persist the explicit user choice (`localStorage`) and restore it before first paint to avoid a flash.
  - Both palettes must meet WCAG 2.1 AA contrast.
- **Accessibility**: semantic markup, labelled controls, keyboard-navigable, visible focus states.
- **Responsive**: mobile-first breakpoints; tables degrade gracefully on small screens.

## Data table baseline

Any table rendering domain data must provide:

| Capability | Requirement |
| --- | --- |
| Sorting | Every relevant column header sorts ascending/descending |
| Filtering | Per-column filters on the relevant columns |
| Pagination | Tables with large data sets are paged; server-side paging beyond a few hundred rows |
| Global search | A single text box that searches across the relevant columns, debounced (~300 ms) |
| Empty/loading/error states | Explicit, non-blank UI for each |

Use `MUI DataGrid` (React) or `v-data-table` (Vuetify) rather than a hand-rolled table. For large data sets, push
sorting, filtering, paging, and search to the backend (`?page=&pageSize=&sort=&order=&q=&filter[col]=`) and return
`{ items, total, page, pageSize }`.

## Backend baseline

- Layer as `api/` (routing, DTOs) → `services/` (use cases) → `domain/` (entities, rules) → `infrastructure/` (data, clients).
- Keep controllers/routers thin: validate, delegate, map the result.
- Expose `/health/live` and `/health/ready`.
- Paginate every collection endpoint; never return an unbounded list.
- Version the API and keep breaking changes behind a new version.

## Decisions

Record significant choices as ADRs (see [`documentation.instructions.md`](documentation.instructions.md)).
