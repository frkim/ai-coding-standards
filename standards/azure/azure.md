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

## 8. Deployment checklist

- [ ] Bicep deployed via pipeline using OIDC; no manual portal edits.
- [ ] Managed identity + RBAC assignments in code.
- [ ] Secrets in Key Vault, referenced not copied.
- [ ] Diagnostics, alerts, and dashboards configured.
- [ ] Autoscale, health probes, and backups configured.
- [ ] Tags applied; budget in place.
