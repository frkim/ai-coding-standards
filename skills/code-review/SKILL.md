---
name: code-review
description: Review a pull request or change set for correctness, design, tests, performance, and maintainability, and give actionable prioritised feedback. Use when asked to review code or before opening a pull request.
---

# Code review skill

Review the diff, but read enough surrounding code to judge whether the change is correct in context.

## What to check, in order

1. **Correctness** — does it do what the issue asks? Off-by-one, null/undefined, error paths, concurrency,
   time zones, floating point money, unhandled promise rejections.
2. **Security** — delegate to the [`security-review`](../security-review/SKILL.md) skill for anything touching
   identity, input, data, or configuration.
3. **Tests** — is the new behaviour covered, including failure paths? Does a bug fix include a regression test?
   Were the tests actually run?
4. **Design** — right layer, no duplicated logic, no leaking infrastructure into the domain, dependencies inward.
5. **Performance** — N+1 queries, unbounded result sets, missing pagination, synchronous I/O on hot paths,
   unnecessary re-renders, missing indexes.
6. **API/UI contract** — backwards compatibility, status codes, accessibility, dark/light theme support,
   table sorting/filtering/paging/global search where applicable.
7. **Maintainability** — naming, dead code, magic numbers, comments that explain *why*, documentation updated.

## How to give feedback

- Prioritise: **Blocking** (must fix), **Should fix**, **Nit** (optional).
- Be specific and actionable: point at the line, explain the consequence, propose the fix.
- Prefer a suggested diff over prose when the fix is small.
- Do not comment on formatting — that is the linter's job.
- Acknowledge good decisions briefly; reviews are also a teaching tool.

```markdown
**Blocking** — `services/customer_service.py:73`
`page_size` is passed straight to the query, so a caller can request 1,000,000 rows and exhaust memory.
Clamp it to the documented maximum of 100 (or return `400`) before hitting the repository.
```

## Approve when

- [ ] Requirements met and edge cases handled.
- [ ] No blocking security findings.
- [ ] Tests exist, are meaningful, and pass.
- [ ] Design fits the existing architecture.
- [ ] Docs and ADRs updated where behaviour or decisions changed.
- [ ] CI is green.
