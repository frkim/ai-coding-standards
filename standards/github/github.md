# GitHub standards

How repositories, branches, pull requests, and workflows are managed.

## 1. Repository setup

- `README.md`, `LICENSE`, `CONTRIBUTING.md`, `SECURITY.md`, `CODEOWNERS`, `.gitignore`, `.editorconfig`.
- `AGENTS.md` or `.github/copilot-instructions.md` from [`templates/`](../../templates/) so agents inherit these standards.
- Issue and pull request templates in `.github/`.
- Topics and a one-line description set for discoverability.

## 2. Branching and commits

- Trunk-based: short-lived branches off `main`, merged within a few days.
- Branch names: `feature/<issue>-<slug>`, `fix/<issue>-<slug>`, `chore/<slug>`.
- **Conventional Commits**: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`, `ci:`, `perf:`;
  append `!` or a `BREAKING CHANGE:` footer for breaking changes.
- Squash merge into `main` with a conventional commit title; the history stays linear and readable.

## 3. Pull requests

- Small and focused; one logical change. Link the issue with `Closes #123`.
- Description states what changed, why, how it was verified, and any risk or follow-up.
- At least one approving review from a code owner; all status checks green; no unresolved conversations.
- Draft pull requests for work in progress.

## 4. Branch protection (default branch)

- [ ] Require a pull request and at least one approval.
- [ ] Dismiss stale approvals on new commits.
- [ ] Require status checks: build, lint, test, CodeQL.
- [ ] Require branches to be up to date before merging.
- [ ] Block force pushes and deletions.
- [ ] Require signed commits where policy demands it.

## 5. Actions

CI gives fast, deterministic feedback. Keep workflows intentionally simple: plain steps calling the repository's
own lint, build, and test commands — no extra orchestration framework.

- Prefer supported **LTS** runtimes and deterministic CI over automatically tracking the newest "Current" release.
- Pin actions to a full commit SHA, not a tag, with a trailing comment recording the version (`# v7`). This is the
  baseline for every action and mandatory for third-party actions, where it is the main software-supply-chain control.
- Set `permissions: contents: read` at workflow level; grant anything more (`id-token: write`,
  `pull-requests: write`) only on the job that needs it.
- Authenticate to Azure with OIDC (`azure/login@<sha>` + job-level `id-token: write`); never store cloud credentials.
- Use `concurrency` to cancel obsolete runs for the same branch or pull request.
- Install dependencies deterministically from committed lock files (`npm ci` with `package-lock.json`,
  `dotnet restore --locked-mode` with `packages.lock.json`) and cache them keyed on the lock file hash.
- Never interpolate untrusted input directly into `run:` — pass it through `env:`.
- Environments with required reviewers gate production deployments.

```yaml
permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true
```

### 5.1 Action and runtime versions

Node.js 20 is deprecated on the runners: actions that target it are force-run on Node 24 and will eventually
fail. See the
[deprecation changelog](https://github.blog/changelog/2025-09-19-deprecation-of-node-20-on-github-actions-runners/).

| Item | Baseline |
| --- | --- |
| `actions/checkout` | `v7` |
| `actions/setup-node` | `v7` |
| `actions/setup-dotnet` | `v6` |
| Node.js (frontend CI) | **24 LTS** — `node-version: '24'` or `.nvmrc` |
| .NET (backend CI) | **10 LTS** — `dotnet-version: '10.0.x'` or `global.json` |

- Use action versions that run on **Node 24** — the pinned commit's `action.yml` must declare `runs.using: node24`.
  Older majors of `actions/checkout`, `actions/setup-node`, `actions/setup-dotnet`, `actions/upload-artifact`,
  `actions/download-artifact` and `actions/cache` target Node 20: upgrade to the latest major and re-pin the SHA.
- Write custom JavaScript actions against `node24` and build them with the matching Node version.
- Keep the CI runtime identical to the application runtime: read it from `.nvmrc` / `package.json` `engines` and
  `global.json` rather than repeating a different version in the workflow.
- Move .NET projects from .NET 8 to **.NET 10 LTS** when the application and its dependencies support it. If a
  project must stay on another runtime, document the required version explicitly in `global.json`, the README,
  and an ADR, and use that version in CI.
- Dependabot with a `github-actions` ecosystem entry keeps the pinned SHAs and majors current.

```yaml
# action.yml of a custom JavaScript action
runs:
  using: node24
  main: dist/index.js
```

### 5.2 Reference CI workflow

Copy to `.github/workflows/ci.yml` and keep the jobs that match the repository's stack. Replace each `<sha>` with
the full commit SHA of the tagged release named in the comment.

- **Frontend**: Node.js 24 LTS, `npm` cache, `npm ci` from `package-lock.json`.
- **Backend**: .NET 10 LTS, NuGet cache, locked restore. Enable lock files once in `Directory.Build.props` with
  `<RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>` and commit every `packages.lock.json`.
- **Infrastructure**: Bicep lint and build on every run; ARM deployment validation and `what-if` only when an
  Azure environment is configured (see [§5.3](#53-bicep-validation)).

```yaml
name: ci

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true

jobs:
  frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<sha> # v7
      - uses: actions/setup-node@<sha> # v7
        with:
          node-version: '24' # or node-version-file: .nvmrc
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run build
      - run: npm test

  backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<sha> # v7
      - uses: actions/setup-dotnet@<sha> # v6
        with:
          dotnet-version: '10.0.x' # or global-json-file: global.json
          cache: true
          cache-dependency-path: '**/packages.lock.json'
      - run: dotnet restore --locked-mode
      - run: dotnet format --verify-no-changes --no-restore
      - run: dotnet build --no-restore -warnaserror
      - run: dotnet test --no-build

  bicep:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<sha> # v7
      - run: az bicep install
      - run: az bicep lint --file infra/main.bicep
      - run: az bicep build --file infra/main.bicep --stdout > /dev/null

  bicep-validate:
    # runs only when an Azure environment is configured (repository variables) and OIDC is available (not on fork PRs)
    if: vars.AZURE_CLIENT_ID != '' && (github.event_name != 'pull_request' || !github.event.pull_request.head.repo.fork)
    needs: bicep
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
    env:
      RESOURCE_GROUP: ${{ vars.AZURE_RESOURCE_GROUP }}
    steps:
      - uses: actions/checkout@<sha> # v7
      - uses: azure/login@<sha> # v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
      - run: >
          az deployment group validate --resource-group "$RESOURCE_GROUP"
          --template-file infra/main.bicep --parameters infra/main.dev.bicepparam
      - run: >
          az deployment group what-if --resource-group "$RESOURCE_GROUP"
          --template-file infra/main.bicep --parameters infra/main.dev.bicepparam
```

Commands run from the repository root; set `defaults.run.working-directory` on a job when the frontend or backend
lives in a subfolder, and point `cache-dependency-path` at its lock file.

### 5.3 Bicep validation

`az bicep build` only proves the file compiles. Validate infrastructure in layers, cheapest first:

| Check | Command | Needs Azure | Catches |
| --- | --- | --- | --- |
| Lint | `az bicep lint --file infra/main.bicep` | No | Linter rule violations, unsafe defaults, unused code |
| Build | `az bicep build --file infra/main.bicep` | No | Syntax and type errors |
| Deployment validation | `az deployment group validate ...` | Yes | ARM preflight errors: SKUs, names, policy |
| What-if | `az deployment group what-if ...` | Yes | The resource changes the deployment would make |

- Commit a `bicepconfig.json` that raises the important linter rules (for example `no-hardcoded-env-urls`,
  `secure-parameter-default`, `outputs-should-not-contain-secrets`, `use-recent-api-versions`) to `error` so lint
  fails the build.
- Run lint and build on every pull request — they need no credentials.
- Run validation and `what-if` whenever an Azure environment is available, authenticating with OIDC; post or
  review the `what-if` output before approving a deployment.
- Use `az deployment sub` / `az deployment mg` instead of `az deployment group` for templates with a subscription
  or management-group `targetScope`.

## 6. Security features

Enable on every repository: Dependabot alerts and security updates, secret scanning with push protection,
CodeQL code scanning, and private vulnerability reporting for public repositories.

## 7. Releases

- Semantic versioning; tags `v<major>.<minor>.<patch>`.
- Generated release notes from conventional commits; attach the SBOM and build artefacts.
