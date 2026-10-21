# Package feeds (PyPI, NuGet and npm)

Microsoft-managed devices block direct access to public package registries to reduce software supply-chain risk.
The following endpoints are **not reachable** from a managed device:

- `pypi.org/simple`
- `files.pythonhosted.org`
- `api.nuget.org`
- `nuget.org/api/v2`
- `registry.npmjs.org`
- `registry.yarnpkg.com`
- `registry.npmmirror.com`

Packages are served instead through Microsoft-protected feeds backed by Central Feed Services (CFS), which screen
packages before they reach developer machines and build agents.

| Ecosystem | Approved feed |
| --- | --- |
| PyPI | `https://packagefeedproxy.microsoft.io/pypi/simple` |
| NuGet | `https://packagefeedproxy.microsoft.io/nuget/v3/index.json` |
| npm | `https://packagefeedproxy.microsoft.io/npm/` |

Projects already configured against a dedicated Azure Artifacts feed keep working unchanged — do not migrate them.

## Python

Configure the index globally (`~/.config/pip/pip.conf`, or `%APPDATA%\pip\pip.ini` on Windows) or per project:

```ini
# pip.conf
[global]
index-url = https://packagefeedproxy.microsoft.io/pypi/simple
```

Environment variable form, useful in containers and CI:

```bash
export PIP_INDEX_URL=https://packagefeedproxy.microsoft.io/pypi/simple
export UV_INDEX_URL=https://packagefeedproxy.microsoft.io/pypi/simple
```

Poetry:

```toml
[[tool.poetry.source]]
name = "cfs"
url = "https://packagefeedproxy.microsoft.io/pypi/simple"
priority = "primary"
```

## .NET

Commit a `NuGet.config` at the repository root so every machine and agent resolves the same sources:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="cfs" value="https://packagefeedproxy.microsoft.io/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

`<clear />` is important: it removes the inherited public `nuget.org` source instead of leaving a blocked
fallback that produces confusing restore timeouts.

## npm (npm, pnpm, Yarn)

Following the earlier rollout that pointed the global npm defaults on Microsoft-managed devices at a Central Feed
Services (CFS)-protected internal feed, a new policy blocks **direct** access to the public npm registry.

### TL;DR

- Company-managed devices cannot directly access the `npmjs.org` package registry using `npm`, `pnpm`, or `yarn`.
- Your client tools should already be configured to pull from a CFS-protected feed, so you should see no impact on
  your package-management workflow.

### What is changing

Company-managed devices receive a policy update that adds `registry.npmjs.org`, `registry.yarnpkg.com`, and
`registry.npmmirror.com` to the machine's internal block list. Once deployed, you cannot browse, download content
from, or otherwise connect directly to these registries from a Microsoft-managed device. Access is blocked based on
the destination domain.

### Why

Supply-chain attacks on npm are no longer rare events. They are automated, fast-moving, and highly scalable. In the
past year, attackers have compromised hundreds of packages at once, stolen developer credentials, and spread malware
across dependency trees in minutes to hours. Public npm offers no inline blocking: the moment a version is published,
it is live and installable worldwide, malicious or not.

Most npm supply-chain attacks are detected and pulled within days of publication. After observing recent attack
patterns in the public registry, a **7-day hold** gives the ecosystem (and us) time to catch and remove bad packages
before they reach your machine. By requiring all npm packages to be pulled through CFS-protected feeds, each package
has either followed the quarantine process, been properly published by Microsoft, or been manually vetted.

This is not a theoretical risk. Recent attacks against widely used npm packages and related build pipelines have
shown how quickly malicious code can spread through software supply chains before organizations have time to respond.
Because these attacks can execute on installation and propagate through dependency ecosystems, direct access to newly
published public packages creates risk for Microsoft developers and environments.

### What you need to do

- **No action is needed at this time**; most developers should not experience any impact.
- If a tool fails or warns because it cannot access npm, follow the tool's documentation to configure it to use
  `https://packagefeedproxy.microsoft.io/npm/` as its upstream feed.
- If you have an urgent business need for a newly published package version (for example, a critical security or
  production hotfix), **do not attempt to bypass or work around the npm Minimum Release control**. As outlined in the
  Microsoft Security Policy, employees are required to follow security controls and use approved exception processes
  when additional access is needed.

Example `.npmrc` (also honored by `pnpm`; for Yarn, set `npmRegistryServer` in `.yarnrc.yml`):

```ini
registry=https://packagefeedproxy.microsoft.io/npm/
```

## Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| `pip install` hangs or times out on `files.pythonhosted.org` | Public index still configured | Set `PIP_INDEX_URL` / `pip.conf` as above |
| `dotnet restore` fails with `Unable to load the service index for source https://api.nuget.org/v3/index.json` | Public source inherited from the machine-level config | Add a repository `NuGet.config` with `<clear />` |
| `npm install` / `pnpm install` / `yarn` fails or warns on `registry.npmjs.org`, `registry.yarnpkg.com` or `registry.npmmirror.com` | Public registry still configured | Set the registry/upstream to `https://packagefeedproxy.microsoft.io/npm/` as above |
| A required package is missing from the feed | Not yet mirrored or blocked by review | Request it through the Central Feed Services exception process |

## Rules

- Never work around the block with a proxy, VPN, or vendored wheel/`.nupkg`/npm tarball from an unverified source.
- Keep lock files committed so restores are reproducible from the protected feed.
- Reference this document from the README of any repository whose setup steps install Python, .NET or npm packages.
