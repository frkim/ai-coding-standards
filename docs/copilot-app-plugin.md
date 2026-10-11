# Use ai-coding-standards in the GitHub Copilot app

Install this repository as a plugin once and its skills and custom agents are available in every project you open in
the [GitHub Copilot app](https://docs.github.com/en/copilot/how-tos/github-copilot-app/customize-github-copilot-app).

## What the plugin gives you

| Component | Names | How it is used |
| --- | --- | --- |
| Skills (5) | `api-development`, `azure`, `code-review`, `security-review`, `testing` | Loaded automatically when the task matches the skill description |
| Custom agents (4) | `architecture-agent`, `code-review-agent`, `documentation-agent`, `security-agent` | Selected by you in the agent picker or with `/agent` |

`instructions/`, `prompts/`, and `templates/` are **not** activated by the plugin (a plugin carries agents, skills,
hooks, and MCP/LSP servers only). See [Add the repository instructions](#add-the-repository-instructions-optional).

## 1. Add the marketplace

1. Click **Customize** in the app sidebar, then click **Plugins**.
2. Click the gear icon next to the marketplace dropdown.
3. Add the GitHub repository `frkim/ai-coding-standards` and confirm. This repository hosts its own marketplace in
   [`.github/plugin/marketplace.json`](../.github/plugin/marketplace.json).

## 2. Install the plugin

1. In the **Plugins** view, choose the `ai-coding-standards` marketplace in the dropdown.
2. Find the `ai-coding-standards` plugin and click **Install**.
3. Click **Customize** → **Installed** and check that the plugin is listed and enabled.
4. Start a new session so the new skills and agents are loaded.

Verify in a session: type `/agent` and confirm the four agents are listed; open **Customize** → **Skills** and confirm
the five skills are listed.

## 3. Use it

### Custom agents

Select an agent in the prompt box agent picker before or during a session, or type `/agent`.

| Agent | Use it to | Access |
| --- | --- | --- |
| `architecture-agent` | Choose Azure services and record an ADR before implementing | Read, search, web, edit |
| `code-review-agent` | Review a branch or pull request and get prioritised feedback | Read-only |
| `security-agent` | Audit code, dependencies, and Azure configuration | Read-only |
| `documentation-agent` | Write READMEs, API docs, ADRs, and runbooks | Read, search, web, edit |

Example prompts:

```text
/agent → security-agent
Review the changes on this branch for secrets, identity, and input-validation issues.
```

```text
/agent → architecture-agent
Design a document-ingestion service on Azure for 50 requests/second and write the ADR.
```

### Skills

Skills need no selection: Copilot loads one when your request matches its description. Name the skill in the prompt to
force it, for example `Use the azure skill to add a Bicep deployment for this API`.

| Skill | Triggered by |
| --- | --- |
| `api-development` | Creating or changing an endpoint, DTO, or API contract (FastAPI / ASP.NET Core) |
| `azure` | Azure hosting, SDKs, Microsoft Foundry, managed identity, Bicep deployment |
| `code-review` | Reviewing a pull request or change set |
| `security-review` | Threat-driven review or hardening of a workload |
| `testing` | Adding tests, fixing a bug, or verifying a feature |

## Add the repository instructions (optional)

The always-on rules in [`instructions/`](../instructions/) and the task prompts in [`prompts/`](../prompts/) are
copied with the repository but not activated by the plugin. To apply them:

- **Per repository (recommended):** copy the files you need to `.github/instructions/` and `.github/prompts/` in your
  project, plus [`templates/AGENTS.md`](../templates/AGENTS.md) or
  [`templates/copilot-instructions.md`](../templates/copilot-instructions.md). The app uses the repository's
  customizations automatically.
- **In the app only:** open the app settings, click the repository under **Projects**, and paste the rules into
  **Instructions**. To apply them to every session, use **Sessions** → **Instructions** → **App instructions**.

## Update, disable, or remove

- **Update:** bump `version` in [`plugin.json`](../plugin.json) and the marketplace entry when publishing; users then
  update the plugin from **Customize** → **Plugins** (or `copilot plugin update ai-coding-standards`).
- **Disable or remove:** use **Customize** → **Installed**.
- **Same plugin in the CLI:** `copilot plugin marketplace add frkim/ai-coding-standards`, then
  `copilot plugin install ai-coding-standards@ai-coding-standards`.

## Share with a team

Commit this to `.github/copilot/settings.json` so the plugin is enabled for everyone working in the repository,
including Copilot cloud agent and the Copilot CLI:

```json
{
  "enabledPlugins": {
    "ai-coding-standards@ai-coding-standards": true
  },
  "extraKnownMarketplaces": {
    "ai-coding-standards": {
      "source": { "source": "github", "repo": "frkim/ai-coding-standards" }
    }
  }
}
```

Organisations that restrict marketplaces with managed settings must allow `frkim/ai-coding-standards`.

## Test local changes before publishing

Register the working copy as a local marketplace; it is loaded live, so edits apply to the next session:

```shell
copilot plugin marketplace add <path-to-this-repository>
copilot plugin install ai-coding-standards@ai-coding-standards
copilot plugin list
```

In the app, add the same local path through the marketplace gear icon. Remove it with
`copilot plugin uninstall ai-coding-standards` and `copilot plugin marketplace remove ai-coding-standards`.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| Marketplace does not appear | Check that the repository is accessible to your GitHub account (SSO-authorise it if required) and that `.github/plugin/marketplace.json` is on the default branch |
| Plugin installed but no agents | Start a new session; agent files must be named `*.agent.md` and sit in `agents/` |
| A skill is not picked up | Each skill must be `skills/<name>/SKILL.md` with `name` and `description` front matter |
| Rules from `instructions/` are ignored | Plugins do not ship instructions; copy them as described above |
| Installation blocked | An organisation or enterprise managed policy restricts plugins; ask an administrator |
