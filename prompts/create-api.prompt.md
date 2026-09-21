---
mode: agent
description: Create a new REST API endpoint or resource in Python (FastAPI) or C# (ASP.NET Core) following the repository standards.
---

# Create an API

Create the API resource described below, following
[`skills/api-development/SKILL.md`](../skills/api-development/SKILL.md),
[`instructions/security.instructions.md`](../instructions/security.instructions.md), and
[`instructions/testing.instructions.md`](../instructions/testing.instructions.md).

## Inputs

- **Resource**: ${input:resource:e.g. customers}
- **Operations**: ${input:operations:e.g. list, get, create, update, delete}
- **Stack**: ${input:stack:Python FastAPI or C# ASP.NET Core}
- **Data store**: ${input:store:e.g. Azure SQL}

## Requirements

1. Route under `/api/v1/<resource>`, plural noun, verbs as HTTP methods.
2. Request/response DTOs with schema validation (pydantic / FluentValidation or data annotations).
3. The list endpoint supports `page`, `pageSize` (default 25, max 100), `sort`, `order`, per-field filters, and
   `q` global search over the relevant text columns; it returns `{ items, total, page, pageSize }`.
   Allowlist sortable and filterable fields.
4. Correct status codes and RFC 9457 `problem+json` error responses; no internal details leaked.
5. Authentication via Entra ID, authorisation enforced per endpoint, object-level ownership checked.
6. Data access through the repository/service layer; parameterised queries only; managed identity for Azure resources.
7. Structured logging with a correlation id; timeouts and cancellation on outbound calls.
8. OpenAPI metadata: summary, parameters, response models, error codes.
9. Tests: unit tests for the service, integration tests for the endpoint covering the happy path, validation error,
   unauthorised access, not found, and pagination boundaries.

## Deliver

- The implementation, the tests, and the documentation update.
- Run the tests and start the app to exercise at least one call; report the commands you ran and what you observed.
