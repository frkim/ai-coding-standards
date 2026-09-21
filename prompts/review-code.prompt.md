---
mode: agent
description: Review a change set for correctness, security, tests, design, and performance, and report prioritised findings.
---

# Review code

Review ${input:target:the current diff, a pull request, or a file} using
[`skills/code-review/SKILL.md`](../skills/code-review/SKILL.md) and
[`skills/security-review/SKILL.md`](../skills/security-review/SKILL.md).

## Steps

1. Summarise what the change is trying to achieve in one or two sentences.
2. Read the diff, then the surrounding code needed to judge correctness.
3. Evaluate in this order: correctness → security → tests → design → performance → contract/UX → maintainability.
4. Verify each finding against the code before reporting it.

## Report

```markdown
## Summary
<what the change does and the overall verdict>

## Blocking
- `path/file.ext:line` — <problem>. <consequence>. <fix>.

## Should fix
- ...

## Nits
- ...

## Verdict
Approve / Approve with nits / Request changes
```

## Rules

- No formatting or style comments — the linter owns those.
- Every finding cites a file and line and proposes a concrete fix.
- Confirm explicitly whether tests cover the new behaviour and whether they were run.
- Flag any secret, missing authorisation check, or Azure call not using a managed identity as blocking.
