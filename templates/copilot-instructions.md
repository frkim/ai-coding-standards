# Copilot instructions

> Copy this file to `.github/copilot-instructions.md` in a repository and replace the placeholders.
> GitHub Copilot applies it automatically to every request in that repository.

## About this project

<One paragraph: purpose, users, deployment target.>

**Stack**: <Next.js | Vue.js> + Material UI · <Python FastAPI | C# ASP.NET Core> · Azure

## How to work here

- Read the existing code and match its patterns before introducing a new one.
- Keep changes small and focused; prefer the simplest solution that fully meets the requirement.
- Ask for clarification instead of guessing at ambiguous requirements.

## Commands

```bash
<install>   # install dependencies
<run>       # start the app locally
<lint>      # lint and format check
<test>      # run the test suite
```

## Coding rules

- TypeScript `strict`, Python type hints with `mypy`, C# nullable reference types enabled.
- Layer as presentation → service → domain → infrastructure; dependencies point inwards.
- Validate input at the boundary; parameterised queries only.
- Structured logging with a correlation id; never log secrets or personal data.
- Before adding a library, SDK, or runtime, research its **current stable version** online and use that version
  (for example the latest Microsoft Foundry SDK and API version).
- Prefer the **smallest Azure SKU** that meets the requirement; scale up only when a measured limit forces it.
- Write diagrams as [Mermaid](https://mermaid.js.org/), interactive data visualisations with
  [D3.js](https://d3js.org/), and presentations as [Marp](https://marp.app/) Markdown.
- Put throwaway implementation or troubleshooting scripts in the git-ignored `tmp/scripts/` folder.

## Security rules

- Use **managed identities** (`DefaultAzureCredential`) for Azure services; secrets live in Key Vault.
- Never hardcode credentials, tokens, or connection strings — not even in tests.
- Enforce authorisation server-side on every endpoint, including object-level ownership.
- Install Python and .NET packages from the Microsoft-protected feeds
  (`packagefeedproxy.microsoft.io`), not from public PyPI or NuGet.

## UI rules

- Material UI (MUI or Vuetify) with a single shared theme; professional, consistent, accessible (WCAG 2.1 AA).
- Provide a dark/light mode toggle that defaults to the OS preference and persists the user's choice.
- Data tables: sortable and filterable column headers, pagination for large data sets, and a global search box
  over the relevant columns. Push sorting, filtering, paging, and search to the server for large data sets.

## Testing rules

- Add or update tests with every change; bug fixes start with a failing regression test.
- Cover failure paths, not just the happy path.
- **Run the tests and exercise the feature** before saying the work is done, and report what you ran.

## Do not

- Do not add a dependency that duplicates an existing one.
- Do not disable a lint rule or delete a test to make CI pass.
- Do not leave the README or API docs stale after a behaviour change.
