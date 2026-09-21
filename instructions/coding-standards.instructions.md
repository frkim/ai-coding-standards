---
applyTo: "**"
description: "Baseline coding standards for all languages and stacks."
---

# Coding standards

## Stack defaults

- **Frontend**: Next.js (React, TypeScript) or Vue.js. Use **Material UI** (MUI for React, Vuetify for Vue) for a professional and appealing UI.
- **Backend**: Python (FastAPI) or C# (ASP.NET Core).
- **Cloud**: Azure is the default hosting target. See [`azure.md`](../standards/azure/azure.md).

Do not introduce a third framework for a concern that an existing dependency already covers.

## General rules

- Prefer clarity over cleverness; optimise for the next reader.
- Keep functions small and single-purpose; extract helpers instead of nesting deeply.
- Fail fast: validate inputs at the boundary (HTTP handler, CLI entry point, message consumer).
- No secrets, connection strings, or tokens in source, tests, or logs. Use Azure Key Vault and managed identities.
- Every public function, endpoint, and exported component has a documented contract (inputs, outputs, errors).
- Use structured logging with correlation IDs; never log personal data or credentials.
- Handle errors explicitly. Never swallow exceptions; wrap and rethrow with context or handle them meaningfully.

## Naming and layout

| Element | Convention |
| --- | --- |
| TypeScript/JavaScript files | `kebab-case.ts`, React components `PascalCase.tsx` |
| Python modules | `snake_case.py` |
| C# files | `PascalCase.cs`, one public type per file |
| Environment variables | `UPPER_SNAKE_CASE` |
| Feature folders | group by feature, not by technical layer |

## Language specifics

### TypeScript / Next.js / Vue.js

- `strict: true` in `tsconfig.json`. No `any` without a written justification.
- Use ESLint + Prettier; formatting is never discussed in review, it is enforced by the tool.
- Prefer server components / server-side data fetching in Next.js; keep client bundles small.
- Co-locate component, styles, and tests.

### Python

- Target the latest supported LTS Python version; use type hints everywhere and validate with `mypy`.
- Format with `ruff format`, lint with `ruff`.
- Use `pydantic` models for request/response schemas and configuration.
- Manage dependencies with a lock file (`uv.lock`, `poetry.lock`, or `requirements.txt` with pinned hashes).

### C#

- Enable nullable reference types and treat warnings as errors.
- Follow the default `dotnet format` / `.editorconfig` rules.
- Use dependency injection; avoid static mutable state.
- Use `async`/`await` end to end; never block with `.Result` or `.Wait()`.

## Dependencies and package feeds

Microsoft-managed devices block direct access to public PyPI and NuGet registries. Configure the approved
Microsoft-protected feeds instead of the public endpoints:

- PyPI: `https://packagefeedproxy.microsoft.io/pypi/simple`
- NuGet: `https://packagefeedproxy.microsoft.io/nuget/v3/index.json`

See [`package-feeds.md`](../standards/development/package-feeds.md) for ready-to-copy `pip.conf` and `NuGet.config`
snippets. Always check a new dependency for known vulnerabilities and an acceptable licence before adding it.

**Before adding a new library, SDK, runtime, or service, research it online first** and integrate the latest
stable version rather than the version you remember:

- Check the official release notes / registry page for the current stable version, then pin it in the lock file.
- Prefer the latest generally available SDK and API version — for example, use the latest **Microsoft Foundry**
  SDK and model/API versions rather than a superseded preview or an older release.
- Note the release you picked (and why, if it is not the latest) in the pull request description.

## Tooling defaults

| Need | Default tool |
| --- | --- |
| Diagrams in documentation | [Mermaid](https://mermaid.js.org/) in `mermaid` fenced code blocks |
| Interactive / data-driven visualisation | [D3.js](https://d3js.org/) |
| Slide decks and presentations | [Marp](https://marp.app/) Markdown |

## Scratch scripts

Scripts written only to implement, troubleshoot, or explore — not part of the shipped product — go in a
temporary, git-ignored folder at the repository root:

```text
tmp/scripts/      throwaway implementation, diagnostic, and migration helper scripts
```

Add `tmp/` to `.gitignore`. Promote a script to `scripts/` (documented and reviewed) only when the team needs to
run it again; otherwise let it stay disposable.

## Definition of done

- [ ] Code builds and lints clean.
- [ ] Tests added or updated, and the feature was actually exercised (see [`testing.instructions.md`](testing.instructions.md)).
- [ ] Documentation updated (README, API docs, ADR if a decision was made).
- [ ] No new secrets, no new high/critical vulnerabilities.
