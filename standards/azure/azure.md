# Azure standards

Azure is the primary hosting platform. Practical patterns live in [`skills/azure/SKILL.md`](../../skills/azure/SKILL.md).

## 1. Landing zone

- One subscription per environment class (`dev`/`test`, `prod`) where governance requires isolation.
- One resource group per workload and environment: `rg-<workload>-<env>-<region>`.
- Deploy to a primary region with a documented secondary; use zone-redundant SKUs in production.

## 2. Naming

| Resource | Pattern | Example |
| --- | --- | --- |
| Resource group | `rg-<workload>-<env>-<region>` | `rg-orders-prod-weu` |
| Container app | `ca-<workload>-<env>` | `ca-orders-api-prod` |
| App Service | `app-<workload>-<env>` | `app-orders-prod` |
| Function app | `func-<workload>-<env>` | `func-orders-sync-prod` |
| Key Vault | `kv-<workload>-<env>` | `kv-orders-prod` |
| Storage account | `st<workload><env>` | `stordersprod` |
| SQL server / db | `sql-<workload>-<env>` / `sqldb-<name>-<env>` | `sql-orders-prod` |
| Managed identity | `id-<workload>-<env>` | `id-orders-prod` |
| Log Analytics | `log-<workload>-<env>` | `log-orders-prod` |
| Foundry resource / project | `aif-<workload>-<env>` / `proj-<workload>-<env>` | `aif-orders-prod` |

Required tags on every resource: `env`, `workload`, `owner`, `costCenter`, `dataClassification`.

## 3. Identity and access

- Every workload has a **user-assigned managed identity**; application code uses `DefaultAzureCredential`.
- Keys, SAS tokens, and connection strings containing secrets are prohibited where a managed identity works.
- Use built-in RBAC roles at resource scope. `Owner`/`Contributor` are never assigned to a workload identity.
- Key Vault uses RBAC (not access policies), with soft delete and purge protection enabled.

## 4. Infrastructure as code

- All infrastructure in **Bicep** under `infra/`, deployed through CI with OIDC. No portal changes in `prod`.
- Parameter files per environment; `what-if` runs on pull requests, deployment gated on approval for production.
- Store deployment state and outputs in the pipeline, never in the repository.

## 5. Observability

- Application Insights for traces, metrics, and dependencies; Log Analytics for platform diagnostics.
- Distributed tracing with W3C `traceparent` propagated across services.
- Alerts on availability, p95 latency, error rate, queue depth, and cost anomalies, routed to an on-call channel.
- Health probes: `/health/live` for liveness, `/health/ready` for readiness including dependencies.

## 6. Reliability

- Retries with exponential backoff and jitter on transient failures; circuit breakers on outbound dependencies.
- Idempotent message handlers with dead-letter queues.
- Backups: automated, geo-redundant where classification requires it, and restore tested at least twice a year.
- Document RPO and RTO per workload in an ADR.

## 7. Cost

- **Start with the smallest SKU that meets the requirement** (for example Container Apps consumption, App Service
  B1/S1, Azure SQL serverless General Purpose, Standard Blob Storage) and scale up only when a measured limit —
  latency, throughput, quota, or a required feature — forces it. Record the reason in the pull request or an ADR.
- Autoscale rules with sensible minimums; scale to zero in non-production.
- Budgets with alerts at 50/80/100%; reserved capacity for steady production workloads.
- Review the top five cost drivers monthly.

## 8. AI workloads: Microsoft Foundry (new)

**Always use Microsoft Foundry (new)** — the current Foundry portal, the Foundry resource, and Foundry projects —
for models, agents, evaluations, and AI tools. **Never start new work on Foundry (classic).** Classic exists only to
reach legacy hub-based projects; Microsoft's new investment, GA scope, and features (Responses API, Agents v2,
hosted agents, tool catalog) land in Foundry (new) only.

| Concern | Use — Foundry (new) | Do not use — Foundry (classic) |
| --- | --- | --- |
| Resource | Foundry resource (`Microsoft.CognitiveServices/accounts`, kind `AIServices`, `allowProjectManagement: true`) with child Foundry projects | Hub-based projects (Azure Machine Learning hub), standalone Azure OpenAI resources |
| Portal | [ai.azure.com](https://ai.azure.com) with the **New Foundry** toggle on | Foundry (classic) portal and its Management center |
| Documentation | [learn.microsoft.com/azure/foundry](https://learn.microsoft.com/azure/foundry/what-is-foundry) | Pages marked "Applies only to Foundry (classic) portal" (`/azure/foundry-classic/`) |
| Agents | Foundry Agent Service on the **Responses API**: `agents.create_version()` with `PromptAgentDefinition`, conversations and items | Assistants API, `create_agent()`, threads and polled runs (sunset 26 August 2026) |
| Model inference | `openai` package with the standard `OpenAI()` client on the OpenAI **v1** route (no `api-version`) | `azure-ai-inference` (retired 26 August 2026), `AzureOpenAI()` with monthly `api-version` values |
| Project SDK | Foundry SDK **2.x** (`azure-ai-projects` 2.x for Python) | `azure-ai-projects` 1.x, `azure-ai-generative`, `azure-ai-ml` for project work |
| Endpoints | One project endpoint `https://<resource>.services.ai.azure.com/api/projects/<project>` plus the OpenAI v1 endpoint | Per-service endpoints (`openai`, `cognitiveservices`, `azureml`, …) |
| Access | Microsoft Entra ID with managed identity; **Foundry User** / **Foundry Project Manager** / **Foundry Owner** roles; `disableLocalAuth: true` | API keys; `Cognitive Services OpenAI User` as the default role |

Rules:

- Create the Foundry resource in a region that supports the
  [Responses API and Foundry Agent Service](https://learn.microsoft.com/azure/foundry/openai/how-to/responses#supported-regions);
  agents do not work in Foundry (new) from an unsupported region.
- Copy samples only from the Foundry (new) documentation. Do not mix SDK majors — a 2.x sample against a 1.x
  (classic) setup fails.
- Prefer GA capabilities. Using a preview feature (for example multi-agent workflows, agent memory, Foundry IQ)
  requires an ADR.
- Existing classic workloads migrate: [upgrade Azure OpenAI resources](https://learn.microsoft.com/azure/foundry/how-to/upgrade-azure-openai)
  to a Foundry resource, [move hub-based projects](https://learn.microsoft.com/azure/foundry-classic/how-to/migrate-project)
  to Foundry projects, and rewrite Assistants API agents on the Responses API. Record any remaining classic
  dependency in an ADR with a migration date.

Reference: [Migrate from the Foundry (classic) portal](https://learn.microsoft.com/azure/foundry/how-to/navigate-from-classic).

## 9. Deployment checklist

- [ ] Bicep deployed via pipeline using OIDC; no manual portal edits.
- [ ] AI workloads on a Foundry (new) resource and project — no hub-based project, Azure OpenAI resource, or
      Assistants API.
- [ ] Managed identity + RBAC assignments in code.
- [ ] Secrets in Key Vault, referenced not copied.
- [ ] Diagnostics, alerts, and dashboards configured.
- [ ] Autoscale, health probes, and backups configured.
- [ ] Tags applied; budget in place.
