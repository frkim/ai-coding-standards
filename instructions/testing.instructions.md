---
applyTo: "**"
description: "Testing expectations: what to test, how, and the requirement to actually run the feature."
---

# Testing instructions

**When you implement a feature, test it to ensure it works properly as expected.** Writing the test is not enough —
run it, run the app, and exercise the new code path before declaring the work done.

## Test pyramid

| Level | Scope | Tooling |
| --- | --- | --- |
| Unit | One function/class, no I/O | Vitest/Jest, `pytest`, xUnit |
| Integration | Module + real dependency (DB, HTTP, queue) | Testcontainers, `pytest` fixtures, `WebApplicationFactory` |
| End-to-end | User journey through the UI/API | Playwright |

Most tests are unit tests; integration tests cover every persistence and external-call boundary; a small set of
end-to-end tests covers critical journeys (sign-in, primary create/read flow).

## Rules

- Name tests `should_<expected>_when_<condition>` (or the idiomatic equivalent for the framework).
- Arrange–Act–Assert, one behaviour per test, no logic in tests.
- Tests are deterministic and isolated: no shared mutable state, no real clock, no network to third parties.
- Fake at the boundary (HTTP client, repository), not the internals of the unit under test.
- Every bug fix starts with a failing regression test.
- Cover failure paths: validation errors, auth failures, timeouts, empty and large result sets.

## Coverage

Target ≥ 80% line coverage on changed code, and 100% of new branching logic in domain/business code.
Coverage is a signal, not a goal — do not write assertions-free tests to move the number.

## UI tests

For data tables (see [`architecture.instructions.md`](architecture.instructions.md)), assert:

- sorting toggles ascending/descending on each sortable column header;
- column filters narrow the result set;
- pagination renders the correct page size and navigates between pages;
- the global search box filters on the relevant columns;
- the dark/light mode toggle switches theme and persists the choice;
- accessibility: keyboard navigation and visible focus.

## Manual verification

Before finishing a change, state in the pull request what you ran and what you observed, for example:

```text
Verified: `npm run dev`, created an item through the UI, confirmed it appears in the table,
sorted by "Created", searched "acme" (2 rows), switched to dark mode and reloaded (persisted).
```
