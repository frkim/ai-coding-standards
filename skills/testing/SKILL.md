---
name: testing
description: Write and run unit, integration, and end-to-end tests for TypeScript, Python, and C# projects, including UI data-table and theme-toggle checks. Use when adding tests, fixing a bug, or verifying that a feature works.
---

# Testing skill

Follow [`instructions/testing.instructions.md`](../../instructions/testing.instructions.md). This skill covers the
mechanics per stack.

## Commands

| Stack | Unit | Coverage | End-to-end |
| --- | --- | --- | --- |
| Next.js / Vue | `npm test` (Vitest) | `npm test -- --coverage` | `npx playwright test` |
| Python | `pytest` | `pytest --cov=src --cov-report=term-missing` | `pytest tests/e2e` |
| C# | `dotnet test` | `dotnet test --collect:"XPlat Code Coverage"` | `dotnet test tests/E2E` |

Python and .NET packages must be restored from the Microsoft-protected feeds — see
[`package-feeds.md`](../../standards/development/package-feeds.md).

## Patterns

### Vitest + Testing Library

```ts
it('should filter rows when the global search box is used', async () => {
  render(<CustomerTable rows={rows} />);
  await userEvent.type(screen.getByRole('searchbox', { name: /search/i }), 'acme');
  expect(await screen.findAllByRole('row')).toHaveLength(3); // header + 2 matches
});
```

### pytest

```python
@pytest.mark.asyncio
async def test_should_return_400_when_page_size_exceeds_max(client: AsyncClient) -> None:
    response = await client.get("/api/v1/customers", params={"pageSize": 1000})
    assert response.status_code == 400
```

Use fixtures for setup, `pytest.mark.parametrize` for table-driven cases, and `respx`/`responses` to fake HTTP.

### xUnit

```csharp
[Fact]
public async Task ListAsync_ReturnsSecondPage_WhenPageIsTwo()
{
    var result = await _service.ListAsync(new PageRequest(Page: 2, PageSize: 25), CancellationToken.None);
    Assert.Equal(2, result.Page);
    Assert.Equal(25, result.Items.Count);
}
```

Use `WebApplicationFactory<Program>` for API integration tests and Testcontainers for a real database.

## Integration tests

- Run the real dependency in a container (SQL Server, PostgreSQL, Azurite) rather than mocking the driver.
- Reset state between tests (transaction rollback or fresh container).
- Keep external SaaS calls faked; assert on the request that was sent.

## UI checks to always include

- Sorting on each sortable column header.
- Per-column filters.
- Pagination (page size, navigation, total count).
- Global search across the relevant columns.
- Dark/light mode toggle, including persistence across reload.
- Keyboard navigation and focus visibility.

## Always run what you wrote

Execute the suite, then exercise the feature manually (`npm run dev`, `uvicorn app:app --reload`, `dotnet run`) and
report what you observed in the pull request.
