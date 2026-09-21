---
name: api-development
description: Build HTTP APIs in Python (FastAPI) or C# (ASP.NET Core) with consistent routing, validation, pagination, error handling, and OpenAPI documentation. Use when creating or changing an endpoint, DTO, or API contract.
---

# API development skill

## Contract rules

- Base path `/api/v1`; resources are plural nouns (`/api/v1/customers`), actions are HTTP verbs.
- Status codes: `200` read, `201` + `Location` create, `204` delete, `400` validation, `401` unauthenticated,
  `403` unauthorised, `404` missing, `409` conflict, `422` semantic validation, `429` throttled, `500` unexpected.
- Errors use RFC 9457 `application/problem+json`:

```json
{ "type": "https://example.com/errors/validation", "title": "Validation failed",
  "status": 400, "detail": "name must not be empty", "instance": "/api/v1/customers",
  "traceId": "00-4bf92f...-01" }
```

- Never leak stack traces or SQL to the client; log the detail, return the correlation id.

## Pagination, sorting, filtering, search

Every collection endpoint supports and documents:

```text
GET /api/v1/customers?page=1&pageSize=25&sort=createdAt&order=desc&q=acme&status=active
```

```json
{ "items": [], "total": 1234, "page": 1, "pageSize": 25 }
```

- Default `pageSize` 25, maximum 100 — reject larger values with `400`.
- `q` is the global search applied to the relevant text columns (this backs the UI's global search box).
- Allowlist sortable and filterable fields; never interpolate a client-supplied column name into SQL.

## Python (FastAPI)

```python
from fastapi import APIRouter, Depends, Query, status
from pydantic import BaseModel, Field

router = APIRouter(prefix="/api/v1/customers", tags=["customers"])


class CustomerOut(BaseModel):
    id: str
    name: str = Field(max_length=200)


class Page[T](BaseModel):
    items: list[T]
    total: int
    page: int
    page_size: int


@router.get("", response_model=Page[CustomerOut], status_code=status.HTTP_200_OK)
async def list_customers(
    page: int = Query(1, ge=1),
    page_size: int = Query(25, ge=1, le=100),
    q: str | None = Query(None, max_length=100),
    service: CustomerService = Depends(get_customer_service),
) -> Page[CustomerOut]:
    return await service.list(page=page, page_size=page_size, search=q)
```

## C# (ASP.NET Core)

```csharp
app.MapGet("/api/v1/customers", async (
        [AsParameters] PageRequest request,
        ICustomerService service,
        CancellationToken ct) =>
    {
        var result = await service.ListAsync(request, ct);
        return Results.Ok(result);
    })
    .WithName("ListCustomers")
    .Produces<Page<CustomerDto>>()
    .RequireAuthorization();
```

Use `ProblemDetails` middleware for errors, FluentValidation or data annotations for input, and
`AddApiVersioning()` for versioning.

## Cross-cutting

- Authenticate with Entra ID JWTs; authorise per endpoint (`RequireAuthorization` / `Depends(require_scope(...))`).
- Enforce timeouts and cancellation tokens on every outbound call.
- Rate limit public endpoints.
- Return `ETag`/`If-None-Match` for cacheable reads; make writes idempotent with an `Idempotency-Key` where retried.
- Keep OpenAPI generated from code and published at `/openapi.json` (non-production) for client generation.

## Checklist

- [ ] Versioned route, correct status codes, `problem+json` errors.
- [ ] Input validated by schema; sort/filter fields allowlisted.
- [ ] Collection paginated with `q` global search support.
- [ ] AuthN/AuthZ applied; secrets via managed identity.
- [ ] OpenAPI complete; integration tests cover happy path and failures.
