# ai-coding-standards

Central repository for reusable instructions, skills, agents, prompts, and engineering standards for agentic
software development with GitHub Copilot and AI coding agents.

Use it as the single source of truth from GitHub, GitHub Copilot in VS Code and other IDEs, the GitHub Copilot
app, the GitHub Copilot CLI, and the Copilot coding agent.

## Contents

| Path | What it holds | Consumed by |
| --- | --- | --- |
| [`instructions/`](instructions/) | `*.instructions.md` rules applied to matching files | Copilot custom instructions |
| [`skills/`](skills/) | `SKILL.md` capability packs loaded on demand | Agent skills |
| [`agents/`](agents/) | Specialised agent definitions | Custom agents / subagents |
| [`prompts/`](prompts/) | Reusable `*.prompt.md` task prompts | `/` prompt files in Copilot Chat |
| [`standards/`](standards/) | Normative engineering standards | Humans and agents |
| [`templates/`](templates/) | Starting points to copy into a repository | New repositories |
| [`plugin.json`](plugin.json) | Copilot plugin manifest that ships `skills/` and `agents/` | Copilot app, CLI, and cloud agent |
| [`.github/plugin/`](.github/plugin/marketplace.json) | `marketplace.json` that lists the plugin | Copilot plugin marketplaces |

```text
├── instructions/     security · coding-standards · testing · documentation · architecture
├── skills/           azure · api-development · testing · security-review · code-review
├── agents/           architecture · security · code-review · documentation
├── prompts/          create-api · create-tests · review-code · check-compliance
├── standards/        security/ · azure/ · github/ · development/ · compliance/
├── templates/        AGENTS.md · copilot-instructions.md · skill-template/
├── plugin.json       Copilot plugin manifest (ships skills/ and agents/)
└── .github/plugin/   marketplace.json (plugin marketplace catalog)
```

## Install as a GitHub Copilot plugin

This repository is a [GitHub Copilot plugin](https://docs.github.com/en/copilot/concepts/agents/about-plugins) named
`ai-coding-standards` and also hosts the marketplace that lists it. Install it once and the skills and agents are
available in every project, without copying files.

| Shipped by the plugin | Source | You get |
| --- | --- | --- |
| 5 skills | [`skills/`](skills/) | `api-development`, `azure`, `code-review`, `security-review`, `testing` |
| 4 custom agents | [`agents/`](agents/) | `architecture-agent`, `code-review-agent`, `documentation-agent`, `security-agent` |

[`instructions/`](instructions/), [`prompts/`](prompts/), and [`templates/`](templates/) are copied with the plugin but
Copilot does not activate them: a plugin carries agents, skills, hooks, and MCP/LSP servers, not `*.instructions.md` or
`*.prompt.md` files. Copy what you need as described in
[How to use it in your repository](#how-to-use-it-in-your-repository).

### GitHub Copilot app

1. Click **Customize** in the app sidebar, then click **Plugins**.
2. Click the gear icon next to the marketplace dropdown and follow the prompts to add the GitHub repository
   `frkim/ai-coding-standards`.
3. Select the `ai-coding-standards` marketplace, find the `ai-coding-standards` plugin, and click **Install**.
4. Choose one of the plugin's agents with the agent picker or `/agent`; its skills load automatically when relevant.

### GitHub Copilot CLI

```shell
copilot plugin marketplace add frkim/ai-coding-standards
copilot plugin install ai-coding-standards@ai-coding-standards
copilot plugin list
```

The install spec is `PLUGIN@MARKETPLACE`; here both are named `ai-coding-standards`. In an interactive session use
`/plugin marketplace add` and `/plugin install` instead, and pick an agent with `/agent`. Update with
`copilot plugin update ai-coding-standards` and remove with `copilot plugin uninstall ai-coding-standards`. From the
shell, plugin agents are prefixed with the plugin name: `copilot --agent ai-coding-standards:code-review-agent`.

### Copilot cloud agent and shared repositories

To enable the plugin for everyone who works in your repository, including Copilot cloud agent and the Copilot CLI,
commit this to `.github/copilot/settings.json`:

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

## How to use it in your repository

1. **Bootstrap the agent contract.** Copy [`templates/AGENTS.md`](templates/AGENTS.md) to your repository root, or
   [`templates/copilot-instructions.md`](templates/copilot-instructions.md) to `.github/copilot-instructions.md`,
   and fill in the placeholders.
2. **Adopt the instructions.** Copy the files you need from [`instructions/`](instructions/) to
   `.github/instructions/`. Each file's `applyTo` front matter controls which files it applies to.
3. **Add the skills.** Install the [plugin](#install-as-a-github-copilot-plugin), or copy the relevant folders from
   [`skills/`](skills/) to `.github/skills/` (or your agent's skills directory). Start a new one from
   [`templates/skill-template/`](templates/skill-template/).
4. **Add the prompts.** Copy [`prompts/`](prompts/) files to `.github/prompts/` and invoke them with `/` in chat.
5. **Link the standards** from your README so humans and agents read the same rules.
6. **Audit an existing project** with the
   [compliance checklist](standards/compliance/compliance-checklist.md) — by hand, or by running
   [`/check-compliance`](prompts/check-compliance.prompt.md) in agent mode.

## Engineering baseline

These defaults apply unless a project documents a reasoned exception in an ADR.

- **Frontend**: Next.js or Vue.js with **Material UI** — professional, consistent, accessible (WCAG 2.1 AA).
- **Dark/Light mode**: every app has a theme toggle that follows the OS preference and persists the user's choice.
- **Data tables**: sortable and filterable column headers, pagination for large data sets, and a global search box
  across the relevant columns (server-side for large data sets).
- **Backend**: Python (FastAPI) or C# (ASP.NET Core), layered, paginated, versioned APIs.
- **Cloud**: Azure as the primary hosting platform, with Bicep infrastructure as code.
- **Runtimes**: supported LTS releases — **Node.js 24 LTS** and **.NET 10 LTS**; CI is secure and deterministic
  (least-privilege `permissions`, `concurrency`, lock-file installs, Bicep lint/validate/what-if) — see
  [`standards/github/github.md` §5](standards/github/github.md#5-actions).
- **AI platform**: always **Microsoft Foundry (new)** — Foundry resource and projects, Foundry SDK 2.x, and the
  Responses API. Never Foundry (classic), hub-based projects, or the Assistants API — see
  [`standards/azure/azure.md` §8](standards/azure/azure.md#8-ai-workloads-microsoft-foundry-new).
- **Identity**: **managed identities wherever possible** — no keys, no connection strings with secrets.
- **Testing**: when you implement a feature, test it — write the tests, run them, and exercise the feature.
- **Packages**: install Python and .NET dependencies from the Microsoft-protected feeds, not public PyPI/NuGet —
  see [`standards/development/package-feeds.md`](standards/development/package-feeds.md).
- **New dependencies**: research the library, SDK, or runtime online first and integrate the **latest stable
  version** (for example the latest Microsoft Foundry SDK and API version).
- **Cost**: choose the smallest Azure SKU that meets the requirement and scale up only when a measured limit
  forces it.
- **Diagrams and decks**: [Mermaid](https://mermaid.js.org/) for diagrams in documentation,
  [D3.js](https://d3js.org/) for interactive data visualisation, [Marp](https://marp.app/) for presentations.
- **Scratch scripts**: throwaway implementation or troubleshooting scripts go in a git-ignored `tmp/scripts/`
  folder, never in the shipped tree.

## Contributing

Keep every document short, normative, and actionable. Prefer a checklist over prose, cite a command that has
actually been run, and update the related standard when you change an instruction or skill.

The plugin reads `skills/` and `agents/` straight from this repository, so a new skill folder or agent file needs no
manifest change. Bump `version` in [`plugin.json`](plugin.json) and in the plugin entry of
[`.github/plugin/marketplace.json`](.github/plugin/marketplace.json) — keep the two identical — when you change a
skill or agent, so plugin users can update.

## Licence

[MIT](LICENSE)
