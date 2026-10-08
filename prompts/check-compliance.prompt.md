---
mode: agent
description: Audit an existing repository against the ai-coding-standards compliance checklist and report evidence-backed findings.
---

# Check compliance

Audit ${input:target:the current repository, or a folder or service within it} against
[`standards/compliance/compliance-checklist.md`](../standards/compliance/compliance-checklist.md).

## Steps

1. Read the checklist and the standards it cites for any item you are unsure how to judge.
2. Identify the stack from the files present (`package.json`, `*.csproj`, `pyproject.toml`, `infra/*.bicep`, AI SDK
   references) and mark every section that does not apply as `N/A` with a one-line reason.
3. Run the checklist's quick evidence commands, then read the files they point to.
4. Assess every remaining item in order. Give each one a status — `Pass`, `Fail`, `Exception`, `N/A`, or
   `Unverified` — and evidence as `path:line`, a setting, or command output.
5. For each `Fail`, state the consequence and a concrete fix. Treat a deviation as `Exception` only when an ADR in
   `docs/adr/` documents it.
6. Run the documented lint, build, and test commands when the environment allows, and record the result for
   `CODE-01` and `TEST-02`; otherwise mark them `Unverified`.
7. Check repository settings (the GH items) with `gh api` if you have access; otherwise mark them `Unverified` and
   name the role that can check them.

## Report

Use the checklist's report template, listing blocking failures first, and finish with the verdict.

## Rules

- Audit only — do not change the code, configuration, or repository settings.
- Verify every finding against the files before reporting it; no speculative findings.
- Never copy a secret value into the report; cite its location and say it must be rotated.
- Use the checklist IDs (`SEC-01`, `UI-06`, …) so findings can be tracked and re-checked.
