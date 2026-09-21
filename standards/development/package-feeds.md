# Package feeds (PyPI and NuGet)

Microsoft-managed devices block direct access to public package registries to reduce software supply-chain risk.
The following endpoints are **not reachable** from a managed device:

- `pypi.org/simple`
- `files.pythonhosted.org`
- `api.nuget.org`
- `nuget.org/api/v2`

Packages are served instead through Microsoft-protected feeds backed by Central Feed Services (CFS), which screen
packages before they reach developer machines and build agents.

| Ecosystem | Approved feed |
| --- | --- |
| PyPI | `https://packagefeedproxy.microsoft.io/pypi/simple` |
| NuGet | `https://packagefeedproxy.microsoft.io/nuget/v3/index.json` |

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

## Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| `pip install` hangs or times out on `files.pythonhosted.org` | Public index still configured | Set `PIP_INDEX_URL` / `pip.conf` as above |
| `dotnet restore` fails with `Unable to load the service index for source https://api.nuget.org/v3/index.json` | Public source inherited from the machine-level config | Add a repository `NuGet.config` with `<clear />` |
| A required package is missing from the feed | Not yet mirrored or blocked by review | Request it through the Central Feed Services exception process |

## Rules

- Never work around the block with a proxy, VPN, or vendored wheel/`.nupkg` from an unverified source.
- Keep lock files committed so restores are reproducible from the protected feed.
- Reference this document from the README of any repository whose setup steps install Python or .NET packages.
