---
name: azure
description: Design, build, and deploy workloads on Azure using managed identities, Bicep infrastructure as code, and least-privilege RBAC. Use when the task involves Azure hosting, Azure SDKs, Microsoft Foundry models or agents, authentication to Azure services, or deployment.
---

# Azure skill

Azure is the default hosting target. Prefer managed, serverless-leaning services and passwordless authentication.

## Service selection

| Need | Default choice | Use instead when |
| --- | --- | --- |
| Web frontend | Azure Static Web Apps | Server-rendered Next.js → Azure Container Apps / App Service |
| API / backend | Azure Container Apps | Simple web app → App Service; event-driven → Azure Functions |
| Relational data | Azure SQL Database | Open-source stack → PostgreSQL flexible server |
| Document data | Azure Cosmos DB | Low volume → Azure SQL JSON columns |
| Secrets | Azure Key Vault | App config values → App Configuration |
| Files/blobs | Azure Blob Storage | — |
| Messaging | Azure Service Bus | High-throughput telemetry → Event Hubs |
| Observability | Application Insights + Log Analytics | — |
| AI models and agents | **Microsoft Foundry (new)** — Foundry resource + Foundry project | Never Foundry (classic), hub-based projects, or a standalone Azure OpenAI resource |

## Managed identity first

Use a **user-assigned managed identity** for each workload and assign built-in RBAC roles at the narrowest scope.

```python
# Python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()  # managed identity in Azure, developer sign-in locally
client = SecretClient(vault_url=settings.key_vault_url, credential=credential)
```

```csharp
// C#
builder.Services.AddSingleton(new SecretClient(
    new Uri(builder.Configuration["KeyVault:Uri"]!),
    new DefaultAzureCredential()));
```

```csharp
// Azure SQL with managed identity — no password in the connection string
"Server=tcp:<server>.database.windows.net,1433;Database=<db>;Authentication=Active Directory Default;"
```

For GitHub Actions, use **workload identity federation** (OIDC) with `azure/login@v2`; never store a client secret.

Common role assignments: `Key Vault Secrets User`, `Storage Blob Data Contributor`,
`Azure Service Bus Data Sender/Receiver`, `AcrPull`. Avoid `Contributor` and `Owner` for workloads.

## Infrastructure as code

- Author infrastructure in **Bicep** under `infra/`, deploy with `az deployment group create` or `azd up`.
- Parameterise per environment (`dev`, `test`, `prod`); no hardcoded names, regions, or SKUs.
- Tag every resource: `env`, `owner`, `costCenter`, `workload`.
- Enable diagnostic settings to Log Analytics on every resource.

```bicep
resource identity 'Microsoft.ManagedIdentity/userAssignedIdentities@2023-01-31' = {
  name: '${prefix}-id'
  location: location
}

resource kvRole 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  scope: keyVault
  name: guid(keyVault.id, identity.id, 'kv-secrets-user')
  properties: {
    principalId: identity.properties.principalId
    principalType: 'ServicePrincipal'
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', '4633458b-17de-408a-b874-0445c86b69e6')
  }
}
```

## Microsoft Foundry (new), never classic

Build every AI workload on **Microsoft Foundry (new)**. The full rules and the classic-to-new mapping live in
[`standards/azure/azure.md` §8](../../standards/azure/azure.md#8-ai-workloads-microsoft-foundry-new).

- Provision a **Foundry resource** (`Microsoft.CognitiveServices/accounts`, kind `AIServices`) with child
  **Foundry projects** — not a hub-based project or a standalone Azure OpenAI resource.
- Use the **Foundry SDK 2.x** (`azure-ai-projects` 2.x) and the `openai` package on the **Responses API**. Do not use
  `azure-ai-projects` 1.x, `azure-ai-inference`, `AzureOpenAI()` with an `api-version`, or the Assistants API
  (`create_agent()`, threads, runs).
- Read only the Foundry (new) docs at `learn.microsoft.com/azure/foundry`; skip pages marked
  "Applies only to Foundry (classic) portal".
- Authenticate with `DefaultAzureCredential`, disable local (key) auth, and grant the **Foundry User** role.

```python
# Python — Foundry (new): project endpoint, agent version, Responses API
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import PromptAgentDefinition
from azure.identity import DefaultAzureCredential

# https://<resource>.services.ai.azure.com/api/projects/<project>
project = AIProjectClient(endpoint=settings.foundry_project_endpoint, credential=DefaultAzureCredential())

agent = project.agents.create_version(
    agent_name="orders-assistant",
    definition=PromptAgentDefinition(model=settings.model_deployment, instructions="Answer order questions."),
)

openai = project.get_openai_client(agent_name=agent.name)
conversation = openai.conversations.create()
response = openai.responses.create(conversation=conversation.id, input="Where is order 42?")
```

```bicep
resource foundry 'Microsoft.CognitiveServices/accounts@2025-06-01' = {
  name: 'aif-${workload}-${env}'
  location: location
  kind: 'AIServices'
  sku: { name: 'S0' }
  identity: { type: 'SystemAssigned' }
  properties: {
    allowProjectManagement: true
    customSubDomainName: 'aif-${workload}-${env}'
    disableLocalAuth: true
  }
}

resource project 'Microsoft.CognitiveServices/accounts/projects@2025-06-01' = {
  parent: foundry
  name: 'proj-${workload}-${env}'
  location: location
  identity: { type: 'SystemAssigned' }
  properties: {}
}
```

Check the Responses API region availability before choosing `location`, and confirm the latest GA API version of
`Microsoft.CognitiveServices` before you deploy.

## Networking and hardening

- Disable public network access on data stores; use private endpoints where the tier supports it.
- Terminate TLS at Front Door / App Gateway; enable WAF for internet-facing apps.
- Store nothing sensitive in app settings — reference Key Vault.

## Cost and resilience

- **Pick the smallest SKU that meets the requirement** (consumption/serverless tiers first) and only move up when
  a measured limit or a required feature demands it; document the reason.
- Right-size SKUs, enable autoscale, and set budgets with alerts.
- Choose zone-redundant SKUs for production; document the RPO/RTO in an ADR.
- Check the current documentation for the latest generally available SDK, API, and resource-provider versions
  before you add an Azure service — for example the latest **Microsoft Foundry** SDK and API version.

## Checklist

- [ ] Managed identity used for every supported service-to-service call.
- [ ] No keys or connection strings with secrets in configuration.
- [ ] Bicep in `infra/`, parameterised and tagged.
- [ ] Diagnostics and Application Insights wired up.
- [ ] Health probes configured; autoscale rules set.
- [ ] AI workloads use Microsoft Foundry (new): Foundry resource + project, SDK 2.x, Responses API — nothing classic.
