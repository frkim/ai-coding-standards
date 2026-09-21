# Development standards

Day-to-day engineering workflow. Language rules live in
[`instructions/coding-standards.instructions.md`](../../instructions/coding-standards.instructions.md).

## 1. Local environment

| Stack | Toolchain |
| --- | --- |
| Next.js / Vue.js | Node.js LTS, npm or pnpm, TypeScript, ESLint, Prettier, Vitest, Playwright |
| Python | Latest supported Python, `uv` or `poetry`, `ruff`, `mypy`, `pytest` |
| C# | Current .NET LTS SDK, `dotnet format`, xUnit |
| Azure | Azure CLI, Bicep CLI, optionally Azure Developer CLI (`azd`) |

Use a dev container where possible so the toolchain is reproducible.
Configure package feeds as described in [`package-feeds.md`](package-feeds.md) before the first restore.

## 2. Workflow

1. Pick up an issue and confirm the acceptance criteria.
2. Branch from `main` (see [`github/github.md`](../github/github.md)).
3. Implement in small commits; keep the build green.
4. Write and run tests; exercise the feature manually.
5. Update documentation in the same pull request.
6. Open a pull request, address review feedback, squash merge.

## 3. Quality gates

Every repository exposes the same three commands (script names may differ per stack, document them in the README):

```bash
<lint>    # ruff check . | npm run lint | dotnet format --verify-no-changes
<build>   # npm run build | dotnet build -warnaserror
<test>    # pytest | npm test | dotnet test
```

CI runs all three on every pull request and they must pass before merge.

## 4. Configuration

- Twelve-factor: configuration comes from environment variables, secrets from Key Vault.
- Commit a `.env.example` with placeholder values; never commit a real `.env`.
- Fail fast at startup when required configuration is missing, with a message naming the missing key.

## 5. Logging and errors

- Structured JSON logs with level, timestamp, correlation id, and event name.
- `DEBUG` locally, `INFO` in production; `ERROR` only for actionable failures.
- Never log secrets, tokens, or personal data.
- Convert infrastructure exceptions into domain errors at the boundary, preserving the cause.

## 6. Performance

- Paginate every collection, at the API and in the database query.
- Index the columns used for sorting, filtering, and global search.
- Avoid N+1 queries; measure before optimising and record the benchmark.
- Cache deliberately with an explicit invalidation strategy.

## 7. Git hygiene

- Commit lock files; never commit build output, `node_modules`, `bin/`, `obj/`, `.venv/`, or secrets.
- Rebase or merge `main` before requesting review so the pull request is conflict free.
- One reviewer-sized pull request beats one giant one.
