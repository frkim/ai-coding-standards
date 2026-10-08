# Compliance checklist

Use this checklist to audit an **existing** project against the ai-coding-standards framework. It works for a human
reviewer or an AI agent — agents can run it with
[`prompts/check-compliance.prompt.md`](../../prompts/check-compliance.prompt.md).
Each item cites the standard it comes from; the standard wins if the two ever disagree.

## How to use it

1. Copy this file into an issue or a `docs/compliance-<date>.md` file in the audited repository.
2. Identify the stack (frontend, backend, Azure, AI) and mark every section that does not apply as `N/A` with a
   one-line reason — for example "no UI" for [§9](#9-frontend-and-ux).
3. For each item, record a status and the evidence (`path:line`, a setting, or the command output):

   | Status | Meaning |
   | --- | --- |
   | `Pass` | Met, with evidence |
   | `Fail` | Not met — record the gap and a concrete fix |
   | `Exception` | Deviates, but an ADR in `docs/adr/` documents the reason and an expiry or migration date |
   | `N/A` | Does not apply to this project's stack |
   | `Unverified` | Needs access the reviewer does not have (for example repository settings) — name who can check |

4. Fill in the [report](#report-template) and give a verdict.

Items marked **(blocking)** must pass, or be `N/A`, for the project to be compliant. Committed secrets can never be
an `Exception`. Every other `Fail` is a "should fix" finding that needs an owner and a target date.

## 1. Repository and agent setup

Sources: [`github.md` §1](../github/github.md#1-repository-setup),
[`documentation.instructions.md`](../../instructions/documentation.instructions.md),
[`development.md` §7, §9](../development/development.md#7-git-hygiene).

- [ ] **REPO-01** `README.md` explains purpose, prerequisites, how to run, test, and deploy, and how to get help.
- [ ] **REPO-02** `LICENSE`, `CONTRIBUTING.md`, `SECURITY.md` (with a disclosure contact), `CODEOWNERS`,
      `.gitignore`, and `.editorconfig` exist.
- [ ] **REPO-03** `AGENTS.md` or `.github/copilot-instructions.md` exists, is based on
      [`templates/`](../../templates/), and has no unfilled `<placeholder>` left. **(blocking)**
- [ ] **REPO-04** The relevant `*.instructions.md` files are in `.github/instructions/` (or linked from the agent
      contract) with `applyTo` front matter; skills and prompts in use live in `.github/skills/` and
      `.github/prompts/`.
- [ ] **REPO-05** Issue and pull request templates exist in `.github/`.
- [ ] **REPO-06** The repository has a one-line description and topics.
- [ ] **REPO-07** `docs/adr/` exists and holds an ADR for each significant decision and each documented exception.
- [ ] **REPO-08** No build output, `node_modules/`, `bin/`, `obj/`, `.venv/`, `tmp/`, or real `.env` is committed;
      a `.env.example` with placeholder values is.
- [ ] **REPO-09** `tmp/` is in `.gitignore` and throwaway scripts stay in `tmp/scripts/`; any script kept in the
      tree lives in a documented `scripts/` folder.

## 2. GitHub controls

Sources: [`github.md` §4, §6](../github/github.md#4-branch-protection-default-branch),
[`security.md` §5](../security/security.md#5-required-repository-controls).
Check under **Settings → Rules / Branches** and **Settings → Code security**, or with `gh api` (see
[quick evidence commands](#quick-evidence-commands)).

- [ ] **GH-01** The default branch requires a pull request with at least one approval and dismisses stale
      approvals. **(blocking)**
- [ ] **GH-02** Required status checks include lint, build, test, and CodeQL; branches must be up to date.
- [ ] **GH-03** Force pushes and deletions of the default branch are blocked. **(blocking)**
- [ ] **GH-04** Signed commits are required where policy demands it.
- [ ] **GH-05** CodeQL code scanning is enabled with no open critical or high alerts. **(blocking)**
- [ ] **GH-06** Secret scanning with push protection is enabled and has no open alerts. **(blocking)**
- [ ] **GH-07** Dependabot alerts and security updates are enabled; `.github/dependabot.yml` has a `github-actions`
      entry plus one per package ecosystem used.
- [ ] **GH-08** Open Dependabot alerts meet the SLA: critical fixed within 7 days, high within 30. **(blocking)**
- [ ] **GH-09** Private vulnerability reporting is enabled (public repositories).

## 3. Branches, commits, pull requests, and releases

Sources: [`github.md` §2, §3, §7](../github/github.md#2-branching-and-commits).

- [ ] **FLOW-01** Recent commits on `main` use Conventional Commit titles (`feat:`, `fix:`, `docs:`, …) and are
      squash merged, keeping history linear.
- [ ] **FLOW-02** Branches follow `feature/<issue>-<slug>`, `fix/<issue>-<slug>`, or `chore/<slug>` and are
      short-lived.
- [ ] **FLOW-03** Recent pull requests link an issue and state what changed, why, and how it was verified.
- [ ] **FLOW-04** Releases use semantic version tags `v<major>.<minor>.<patch>` with generated release notes and an
      attached SBOM.

## 4. CI workflows

Sources: [`github.md` §5](../github/github.md#5-actions),
[§5.1](../github/github.md#51-action-and-runtime-versions),
[§5.2](../github/github.md#52-reference-ci-workflow),
[§5.3](../github/github.md#53-bicep-validation).

- [ ] **CI-01** A workflow runs lint, build, and test on every pull request and on pushes to `main`.
      **(blocking)**
- [ ] **CI-02** Every workflow sets `permissions: contents: read` at workflow level; broader permissions appear
      only on the job that needs them. **(blocking)**
- [ ] **CI-03** Every `uses:` is pinned to a full 40-character commit SHA with a version comment (`# v7`).
      **(blocking)** for third-party actions.
- [ ] **CI-04** Actions run on Node 24 (`actions/checkout` v7, `actions/setup-node` v7, `actions/setup-dotnet` v6,
      and the current majors of `upload-artifact`, `download-artifact`, and `cache`); custom JavaScript actions use
      `runs.using: node24`.
- [ ] **CI-05** Workflows use `concurrency` with `cancel-in-progress: true`.
- [ ] **CI-06** Dependencies install from committed lock files (`npm ci`, `dotnet restore --locked-mode`, or the
      equivalent) with a cache keyed on the lock file.
- [ ] **CI-07** The CI runtime is read from, or matches, `.nvmrc` / `package.json` `engines` / `global.json`.
- [ ] **CI-08** No untrusted input (issue or pull request titles, bodies, branch names) is interpolated into
      `run:`; it is passed through `env:`. No `pull_request_target` workflow checks out pull request code.
      **(blocking)**
- [ ] **CI-09** Azure authentication uses OIDC (`azure/login` with job-level `id-token: write`); no cloud
      credentials are stored as secrets. **(blocking)**
- [ ] **CI-10** Production deployments are gated by an environment with required reviewers.
- [ ] **CI-11** Bicep projects run `az bicep lint` and `az bicep build` on every pull request with a committed
      `bicepconfig.json` that raises key rules to `error`, plus deployment validation and `what-if` when an Azure
      environment is configured.

## 5. Runtimes, dependencies, and package feeds

Sources: [`development.md` §1, §8](../development/development.md#1-local-environment),
[`package-feeds.md`](../development/package-feeds.md),
[`coding-standards.instructions.md`](../../instructions/coding-standards.instructions.md#dependencies-and-package-feeds).

- [ ] **DEP-01** Runtimes are supported LTS releases — Node.js 24 LTS, .NET 10 LTS, latest supported Python —
      pinned in `.nvmrc` / `engines`, `global.json`, or `.python-version`. Any other runtime is documented in the
      README and an ADR.
- [ ] **DEP-02** Lock files are committed: `package-lock.json` or `pnpm-lock.yaml`; `packages.lock.json` with
      `RestorePackagesWithLockFile` enabled; `uv.lock`, `poetry.lock`, or `requirements.txt` with hashes.
- [ ] **DEP-03** .NET restores through a root `NuGet.config` that uses `<clear />` and the Microsoft-protected feed
      (or an existing Azure Artifacts feed). **(blocking)** where .NET is used.
- [ ] **DEP-04** Python installs through the Microsoft-protected feed (`pip.conf`, `PIP_INDEX_URL` /
      `UV_INDEX_URL`, or a Poetry source), and no configuration points at `pypi.org` or `api.nuget.org`.
      **(blocking)** where Python or .NET is used.
- [ ] **DEP-05** The README links [`package-feeds.md`](../development/package-feeds.md) when setup installs Python
      or .NET packages.
- [ ] **DEP-06** Direct dependencies are on a current stable release; none is deprecated or end of life.
- [ ] **DEP-07** No two dependencies cover the same concern (for example two UI kits or two HTTP clients).

## 6. Code quality

Sources: [`coding-standards.instructions.md`](../../instructions/coding-standards.instructions.md),
[`development.md` §3–§5](../development/development.md#3-quality-gates).

- [ ] **CODE-01** The lint, build, and test commands are documented in the README and pass on a clean clone.
      **(blocking)**
- [ ] **CODE-02** TypeScript uses `strict: true`, ESLint, and Prettier; every `any` carries a written
      justification.
- [ ] **CODE-03** Python uses type hints checked by `mypy`, `ruff` for lint and format, and `pydantic` for schemas
      and settings.
- [ ] **CODE-04** C# enables nullable reference types and treats warnings as errors; `dotnet format
      --verify-no-changes` passes; no `.Result` or `.Wait()` on tasks.
- [ ] **CODE-05** File and folder names follow the naming conventions; code is grouped by feature.
- [ ] **CODE-06** Configuration comes from environment variables, and startup fails fast naming any missing key.
- [ ] **CODE-07** Logs are structured JSON with a correlation id; nothing logs secrets, tokens, or personal data.
- [ ] **CODE-08** No swallowed exceptions; infrastructure errors are mapped to domain errors with the cause kept.
- [ ] **CODE-09** Public functions, endpoints, and exported components document their contract.

## 7. Security

Sources: [`security.md`](../security/security.md),
[`security.instructions.md`](../../instructions/security.instructions.md),
[`skills/security-review`](../../skills/security-review/SKILL.md).

- [ ] **SEC-01** No credentials, keys, tokens, or connection strings with secrets anywhere in the repository,
      including tests, fixtures, infrastructure, and history. **(blocking)**
- [ ] **SEC-02** Every Azure service-to-service call that supports it uses a managed identity via
      `DefaultAzureCredential`; account keys, SAS tokens, and service principal secrets appear only with an ADR
      that sets an expiry date. **(blocking)**
- [ ] **SEC-03** Runtime secrets live in Key Vault and are referenced, not copied into configuration.
- [ ] **SEC-04** All external input is validated at the boundary; uploads enforce a size limit and a content-type
      allowlist. **(blocking)**
- [ ] **SEC-05** Queries are parameterised or use an ORM; client-supplied sort and filter fields are allowlisted.
      **(blocking)**
- [ ] **SEC-06** Users authenticate with Microsoft Entra ID; tokens are validated for issuer, audience, signature,
      and expiry. **(blocking)**
- [ ] **SEC-07** Authorisation is enforced server-side on every endpoint, including object-level ownership checks.
      **(blocking)**
- [ ] **SEC-08** Authentication and public endpoints are rate limited.
- [ ] **SEC-09** Responses set `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, and
      `Strict-Transport-Security`; CORS has no wildcard origin with credentials.
- [ ] **SEC-10** Errors return a correlation id, never a stack trace, internal path, or SQL.
- [ ] **SEC-11** TLS 1.2+ everywhere; no custom cryptography; tokens come from a cryptographic random source.
- [ ] **SEC-12** Containers run as non-root with a read-only root filesystem where possible.
- [ ] **SEC-13** A threat model exists for each externally reachable component or trust boundary.
- [ ] **SEC-14** Personal data collection is minimised, with retention and deletion documented.

## 8. Backend and API

Sources: [`architecture.instructions.md`](../../instructions/architecture.instructions.md#backend-baseline),
[`skills/api-development`](../../skills/api-development/SKILL.md).

- [ ] **API-01** The backend is Python (FastAPI) or C# (ASP.NET Core), or the choice is recorded in an ADR.
- [ ] **API-02** Code is layered `api/` → `services/` → `domain/` → `infrastructure/` with dependencies pointing
      inwards and thin routers or controllers.
- [ ] **API-03** Routes are versioned (`/api/v1`) with plural resource nouns and correct status codes; errors use
      `application/problem+json`.
- [ ] **API-04** Every collection endpoint is paginated (`page`; `pageSize` default 25, maximum 100, larger values
      rejected with `400`), supports `sort`, `order`, and `q` global search, and returns
      `{ items, total, page, pageSize }`. **(blocking)**
- [ ] **API-05** OpenAPI is generated from code, not hand-maintained.
- [ ] **API-06** `/health/live` and `/health/ready` exist; readiness checks dependencies.
- [ ] **API-07** Outbound calls have timeouts and cancellation, retries with exponential backoff and jitter, and
      circuit breakers; message handlers are idempotent with dead-letter queues.
- [ ] **API-08** Services are stateless; state lives in a managed data store.
- [ ] **API-09** Columns used for sorting, filtering, and search are indexed; there are no N+1 queries.

## 9. Frontend and UX

Sources: [`architecture.instructions.md`](../../instructions/architecture.instructions.md#frontend-baseline).

- [ ] **UI-01** The frontend is Next.js (App Router, TypeScript) or Vue.js 3 (`<script setup>`, TypeScript), or the
      choice is recorded in an ADR.
- [ ] **UI-02** Material UI (MUI or Vuetify) is used with a single theme file owning palette, typography, spacing,
      and component overrides; there is no ad-hoc inline styling.
- [ ] **UI-03** The app bar has a dark/light mode toggle that defaults to `prefers-color-scheme`, persists the
      user's choice, and restores it before first paint. **(blocking)**
- [ ] **UI-04** Both palettes meet WCAG 2.1 AA contrast; markup is semantic, controls are labelled, and the UI is
      keyboard navigable with visible focus.
- [ ] **UI-05** Layouts are responsive and mobile first; tables degrade gracefully on small screens.
- [ ] **UI-06** Every domain data table uses MUI DataGrid or `v-data-table` and provides sortable column headers,
      per-column filters, pagination, and a debounced global search box. **(blocking)**
- [ ] **UI-07** Large data sets push sorting, filtering, paging, and search to the server.
- [ ] **UI-08** Tables and data views show explicit empty, loading, and error states.

## 10. Azure and infrastructure

Sources: [`azure.md`](../azure/azure.md), [`skills/azure`](../../skills/azure/SKILL.md).

- [ ] **AZ-01** All infrastructure is Bicep under `infra/` with a parameter file per environment, deployed only
      through CI; there are no manual portal changes in production. **(blocking)**
- [ ] **AZ-02** Resources follow the naming patterns and carry the tags `env`, `workload`, `owner`, `costCenter`,
      and `dataClassification`.
- [ ] **AZ-03** Each workload has a user-assigned managed identity with built-in roles at resource scope; no
      workload identity holds `Owner` or `Contributor`. **(blocking)**
- [ ] **AZ-04** Key Vault uses RBAC with soft delete and purge protection enabled.
- [ ] **AZ-05** Data stores disable public network access and use private endpoints where available;
      internet-facing apps sit behind a WAF.
- [ ] **AZ-06** Diagnostic settings send to Log Analytics; Application Insights collects traces with W3C
      `traceparent` propagation; alerts cover availability, p95 latency, error rate, queue depth, and cost.
- [ ] **AZ-07** SKUs are the smallest that meet the requirement, with the reason for any larger SKU recorded;
      autoscale is configured and non-production scales to zero.
- [ ] **AZ-08** Budgets alert at 50, 80, and 100%.
- [ ] **AZ-09** Production uses zone-redundant SKUs; backups are automated with restores tested; RPO and RTO are in
      an ADR.

## 11. AI workloads (Microsoft Foundry new)

Sources: [`azure.md` §8](../azure/azure.md#8-ai-workloads-microsoft-foundry-new).
Mark the section `N/A` if the project uses no AI models or agents.

- [ ] **AI-01** Models and agents run on a Foundry resource (`Microsoft.CognitiveServices/accounts`, kind
      `AIServices`, `allowProjectManagement: true`) with Foundry projects — no hub-based project or standalone
      Azure OpenAI resource. **(blocking)**
- [ ] **AI-02** The Foundry resource sets `disableLocalAuth: true`; callers use a managed identity with a Foundry
      role, never API keys. **(blocking)**
- [ ] **AI-03** Code uses the Foundry SDK 2.x (`azure-ai-projects` 2.x for Python) and the `openai` package on the
      Responses API — not `azure-ai-projects` 1.x, `azure-ai-inference`, `AzureOpenAI()` with an `api-version`, or
      the Assistants API (`create_agent()`, threads, runs). **(blocking)**
- [ ] **AI-04** The Foundry resource is in a region that supports the Responses API and Foundry Agent Service.
- [ ] **AI-05** Each preview feature in use, and each remaining classic dependency (with a migration date), has an
      ADR.

## 12. Testing

Sources: [`testing.instructions.md`](../../instructions/testing.instructions.md),
[`skills/testing`](../../skills/testing/SKILL.md).

- [ ] **TEST-01** Unit tests exist, integration tests cover persistence and external-call boundaries, and
      Playwright end-to-end tests cover critical journeys. **(blocking)** for unit tests.
- [ ] **TEST-02** The test suite passes locally and in CI.
- [ ] **TEST-03** Tests are deterministic and isolated: no shared mutable state, real clock, or third-party
      network.
- [ ] **TEST-04** Line coverage on recently changed code is at least 80%.
- [ ] **TEST-05** UI tests cover table sorting, column filters, pagination, global search, the theme toggle with
      persistence, and keyboard navigation.
- [ ] **TEST-06** Recent bug-fix pull requests include a regression test.
- [ ] **TEST-07** Recent pull requests state what was run and observed to verify the change.

## 13. Documentation

Sources: [`documentation.instructions.md`](../../instructions/documentation.instructions.md).

- [ ] **DOC-01** Every command in the README runs as written on a clean clone.
- [ ] **DOC-02** ADRs follow the template: status, date, context, decision, consequences, alternatives considered.
- [ ] **DOC-03** API documentation is generated and covers auth, parameters, schemas, errors, and an example per
      endpoint; breaking changes are in a changelog.
- [ ] **DOC-04** Diagrams are Mermaid code blocks (no binary diagram as the source of truth); interactive
      visualisations use D3.js; presentations are Marp Markdown.
- [ ] **DOC-05** Links between repository documents are relative and fenced code blocks have a language tag.
- [ ] **DOC-06** Recent behaviour changes updated the README and API docs in the same pull request.

## Quick evidence commands

Run these from the root of the audited repository. No output means no match.

```bash
# CI-03: actions not pinned to a full commit SHA (local ./ and docker:// actions excluded)
grep -rnE '^\s*-?\s*uses:' .github/workflows .github/actions 2>/dev/null \
  | grep -vE '@[0-9a-f]{40}' | grep -vE 'uses:\s*(\./|docker://)'

# CI-02: workflows without a top-level permissions block
for f in .github/workflows/*.y*ml; do [ -e "$f" ] && ! grep -qE '^permissions:' "$f" && echo "$f"; done

# DEP-04: public package registries referenced in configuration
grep -rnE 'pypi\.org|files\.pythonhosted\.org|api\.nuget\.org|nuget\.org/api' \
  --exclude-dir={.git,node_modules,.venv,bin,obj} .

# AI-03: Foundry (classic) or legacy SDK usage
grep -rnE 'azure-ai-inference|AzureOpenAI\(|api_version=|create_agent\(|threads\.(create|runs)' \
  --exclude-dir={.git,node_modules,.venv,bin,obj} .

# SEC-02: key-based connection strings
grep -rnE 'AccountKey=|SharedAccessKey=|Password=' --exclude-dir={.git,node_modules,.venv,bin,obj} .
```

Repository settings for the GH items need the [GitHub CLI](https://cli.github.com/) signed in with admin read access:

```bash
gh api repos/{owner}/{repo}/branches/main/protection       # GH-01 to GH-04 (branch protection)
gh api repos/{owner}/{repo}/rules/branches/main            # GH-01 to GH-04 (rulesets)
gh api repos/{owner}/{repo} --jq '.security_and_analysis'  # GH-06 and GH-07
```

## Report template

```markdown
# Compliance report: <repository> — <date>

Reviewer: <name or agent> · Commit: <sha> · Stack: <frontend / backend / Azure / AI>

## Verdict
<Compliant | Compliant with findings | Non-compliant> — <one-sentence summary>

## Blocking failures
- **SEC-01** `src/settings.py:12` — storage account key committed. Rotate it, purge history, use a managed identity.

## Other failures
- **CI-05** `.github/workflows/ci.yml` — no `concurrency` block. Add the block from github.md §5. Owner: <name>.

## Exceptions
- **DEP-01** .NET 8 retained — `docs/adr/0004-stay-on-net8.md`, migration due <date>.

## Not applicable and unverified
- §11 AI workloads — N/A, no AI features.
- **GH-01** — Unverified, needs a repository admin.

## Summary
| Section | Pass | Fail | Exception | N/A | Unverified |
| --- | --- | --- | --- | --- | --- |
| 1. Repository and agent setup | | | | | |
```

Choose the verdict as follows:

- **Non-compliant** — at least one blocking item is `Fail`.
- **Compliant with findings** — no blocking item fails, but at least one item is `Fail` or `Exception`.
- **Compliant** — every item is `Pass` or `N/A`.

A blocking item left `Unverified` makes the verdict provisional; say so in the summary line.
