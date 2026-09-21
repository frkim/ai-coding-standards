# AGENTS.md

> Copy this file to the root of a repository and replace the placeholders. It is the contract every AI coding
> agent (GitHub Copilot coding agent, Copilot CLI, Copilot in the IDE, and others) reads before working here.

## Project overview

<One paragraph: what this project does, who uses it, and how it is deployed.>

- **Frontend**: <Next.js | Vue.js> with Material UI
- **Backend**: <Python FastAPI | C# ASP.NET Core>
- **Data**: <Azure SQL | Cosmos DB | PostgreSQL>
- **Hosting**: Azure (<Container Apps | App Service | Static Web Apps>)

## Setup

```bash
# prerequisites: <Node.js LTS | Python 3.x | .NET SDK x>, Azure CLI
<install command>
```

Configure the Microsoft-protected package feeds before the first restore — see
`standards/development/package-feeds.md` in the ai-coding-standards repository:

- PyPI: `https://packagefeedproxy.microsoft.io/pypi/simple`
- NuGet: `https://packagefeedproxy.microsoft.io/nuget/v3/index.json`

## Commands

| Task | Command |
| --- | --- |
| Run locally | `<run command>` |
| Lint | `<lint command>` |
| Build | `<build command>` |
| Test | `<test command>` |
| Deploy | `<deploy command>` |

## Project structure

```text
src/            application code
tests/          unit, integration, and end-to-end tests
infra/          Bicep infrastructure as code
docs/adr/       architecture decision records
```

## Standards to follow

- Coding: `instructions/coding-standards.instructions.md`
- Security: `instructions/security.instructions.md`
- Testing: `instructions/testing.instructions.md`
- Documentation: `instructions/documentation.instructions.md`
- Architecture and UI: `instructions/architecture.instructions.md`

## Non-negotiables

- Use **managed identities** for Azure access; never commit secrets or connection strings.
- Every UI exposes a **dark/light mode toggle** that respects the OS preference and persists the user's choice.
- Every data table supports **sorting and filtering on column headers**, **pagination** for large data sets, and a
  **global search box** across the relevant columns.
- Validate all input; paginate every collection endpoint.
- **When you implement a feature, test it** — run the tests and exercise the feature, then report what you observed.
- Update documentation in the same pull request as the code.

## Pull requests

- Conventional Commit titles (`feat:`, `fix:`, `docs:`, ...), squash merge.
- Describe what changed, why, and how it was verified.
- CI (lint, build, test, CodeQL) must be green before merge.

## Gotchas

- <Anything surprising: required env vars, long-running migrations, flaky integrations.>
