---
mode: agent
description: Generate meaningful unit, integration, and UI tests for existing code and run them.
---

# Create tests

Write tests for ${input:target:file, module, or feature} following
[`instructions/testing.instructions.md`](../instructions/testing.instructions.md) and
[`skills/testing/SKILL.md`](../skills/testing/SKILL.md).

## Steps

1. Read the target code and list its behaviours, inputs, outputs, and failure modes.
2. Check the existing test suite and match its framework, structure, naming, and fixtures.
3. Write tests covering:
   - the happy path for each public behaviour;
   - boundary values (empty, single, maximum, off-by-one);
   - invalid input and validation errors;
   - failure of dependencies (timeout, exception, non-2xx response);
   - authorisation failures where applicable;
   - concurrency or idempotency if the code claims either.
4. For UI work also assert: column sorting, column filters, pagination, the global search box,
   the dark/light mode toggle including persistence, and keyboard accessibility.
5. Run the suite and fix any failure that is caused by the tests themselves.

## Rules

- No logic, loops, or conditionals inside tests; use parametrised cases instead.
- Fake at the boundary, not the internals; no network calls to third parties.
- Deterministic: inject the clock and any random source.
- A test must fail if the behaviour it describes is broken — verify by temporarily breaking the code if unsure.

## Deliver

The new tests, the command used to run them, and the resulting pass/fail summary and coverage delta.
