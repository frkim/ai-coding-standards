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

- Pin actions to a full commit SHA, not a tag.
- Set the minimum `permissions` at workflow level, e.g. `contents: read`.
- Authenticate to Azure with OIDC (`azure/login@<sha>` + `id-token: write`); never store cloud credentials.
- Use `concurrency` to cancel superseded runs; cache dependencies by lock file hash.
- Never interpolate untrusted input directly into `run:` — pass it through `env:`.
- Environments with required reviewers gate production deployments.

```yaml
permissions:
  contents: read
  id-token: write

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### 5.1 Node 24 runtime

Node.js 20 is deprecated on the runners: actions that target it are force-run on Node 24 and will eventually
fail. See the
[deprecation changelog](https://github.blog/changelog/2025-09-19-deprecation-of-node-20-on-github-actions-runners/).

- Use action versions that run on **Node 24** — the pinned commit's `action.yml` must declare `runs.using: node24`.
  Older majors of `actions/checkout`, `actions/setup-node`, `actions/upload-artifact`, `actions/download-artifact`
  and `actions/cache` still target Node 20: upgrade to the latest major and re-pin the SHA.
- Write custom JavaScript actions against `node24` and build them with the matching Node version.
- With `actions/setup-node`, pin `node-version` to `24` (or the version in `.nvmrc` / `package.json` `engines`)
  so the workflow runtime matches the application runtime.
- Dependabot with a `github-actions` ecosystem entry keeps the pinned SHAs and majors current.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # pin to the SHA; the comment records the major version
      - uses: actions/checkout@<sha> # v5 or later — runs.using: node24
      - uses: actions/setup-node@<sha> # v5 or later — runs.using: node24
        with:
          node-version: '24'
          cache: npm
```

```yaml
# action.yml of a custom JavaScript action
runs:
  using: node24
  main: dist/index.js
```

## 6. Security features

Enable on every repository: Dependabot alerts and security updates, secret scanning with push protection,
CodeQL code scanning, and private vulnerability reporting for public repositories.

## 7. Releases

- Semantic versioning; tags `v<major>.<minor>.<patch>`.
- Generated release notes from conventional commits; attach the SBOM and build artefacts.
