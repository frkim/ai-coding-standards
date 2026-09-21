# ai-coding-standards

Central repository for reusable instructions, skills, agents, prompts, and engineering standards for agentic
software development with GitHub Copilot and AI coding agents.

Use it as the single source of truth from GitHub, GitHub Copilot in VS Code and other IDEs, the GitHub Copilot
app, the GitHub Copilot CLI, and the Copilot coding agent.

## Contents

| Folder | What it holds | Consumed by |
| --- | --- | --- |
| [`instructions/`](instructions/) | `*.instructions.md` rules applied to matching files | Copilot custom instructions |
| [`skills/`](skills/) | `SKILL.md` capability packs loaded on demand | Agent skills |
| [`agents/`](agents/) | Specialised agent definitions | Custom agents / subagents |
| [`prompts/`](prompts/) | Reusable `*.prompt.md` task prompts | `/` prompt files in Copilot Chat |
| [`standards/`](standards/) | Normative engineering standards | Humans and agents |
| [`templates/`](templates/) | Starting points to copy into a repository | New repositories |

```text
├── instructions/     security · coding-standards · testing · documentation · architecture
├── skills/           azure · api-development · testing · security-review · code-review
├── agents/           architecture · security · code-review · documentation
├── prompts/          create-api · create-tests · review-code
├── standards/        security/ · azure/ · github/ · development/
└── templates/        AGENTS.md · copilot-instructions.md · skill-template/
```

## How to use it in your repository

1. **Bootstrap the agent contract.** Copy [`templates/AGENTS.md`](templates/AGENTS.md) to your repository root, or
   [`templates/copilot-instructions.md`](templates/copilot-instructions.md) to `.github/copilot-instructions.md`,
   and fill in the placeholders.
2. **Adopt the instructions.** Copy the files you need from [`instructions/`](instructions/) to
   `.github/instructions/`. Each file's `applyTo` front matter controls which files it applies to.
3. **Add the skills.** Copy the relevant folders from [`skills/`](skills/) to `.github/skills/`
   (or your agent's skills directory). Start a new one from [`templates/skill-template/`](templates/skill-template/).
4. **Add the prompts.** Copy [`prompts/`](prompts/) files to `.github/prompts/` and invoke them with `/` in chat.
5. **Link the standards** from your README so humans and agents read the same rules.

## Engineering baseline

These defaults apply unless a project documents a reasoned exception in an ADR.

- **Frontend**: Next.js or Vue.js with **Material UI** — professional, consistent, accessible (WCAG 2.1 AA).
- **Dark/Light mode**: every app has a theme toggle that follows the OS preference and persists the user's choice.
- **Data tables**: sortable and filterable column headers, pagination for large data sets, and a global search box
  across the relevant columns (server-side for large data sets).
- **Backend**: Python (FastAPI) or C# (ASP.NET Core), layered, paginated, versioned APIs.
- **Cloud**: Azure as the primary hosting platform, with Bicep infrastructure as code.
- **Identity**: **managed identities wherever possible** — no keys, no connection strings with secrets.
- **Testing**: when you implement a feature, test it — write the tests, run them, and exercise the feature.
- **Packages**: install Python and .NET dependencies from the Microsoft-protected feeds, not public PyPI/NuGet —
  see [`standards/development/package-feeds.md`](standards/development/package-feeds.md).

## Contributing

Keep every document short, normative, and actionable. Prefer a checklist over prose, cite a command that has
actually been run, and update the related standard when you change an instruction or skill.

## Licence

[MIT](LICENSE)
