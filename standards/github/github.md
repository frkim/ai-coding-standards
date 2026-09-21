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

## 6. Security features

Enable on every repository: Dependabot alerts and security updates, secret scanning with push protection,
CodeQL code scanning, and private vulnerability reporting for public repositories.

## 7. Releases

- Semantic versioning; tags `v<major>.<minor>.<patch>`.
- Generated release notes from conventional commits; attach the SBOM and build artefacts.
