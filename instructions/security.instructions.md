---
applyTo: "**"
description: "Security requirements for code, dependencies, and Azure workloads."
---

# Security instructions

Security is a blocking requirement, not a follow-up task.

## Identity and secrets

- **Use managed identities whenever possible.** User-assigned managed identity is the default for Azure workloads;
  fall back to workload identity federation for GitHub Actions. Connection strings and client secrets are a last
  resort and require a documented justification.
- Never commit secrets. Use Azure Key Vault (via `DefaultAzureCredential`) at runtime and GitHub Actions OIDC in CI.
- Rotate anything that leaks, then purge it from history — assume a committed secret is compromised.
- Scope permissions with least privilege: prefer built-in Azure RBAC roles at the narrowest resource scope.

## Input handling

- Validate and normalise all external input (HTTP bodies, query strings, headers, files, queue messages).
- Use parameterised queries or an ORM; never build SQL by string concatenation.
- Encode output by context (HTML, attribute, URL, shell) to prevent XSS and injection.
- Enforce a maximum size and an allowlist of content types on uploads.

## AuthN / AuthZ

- Authenticate with Microsoft Entra ID; validate issuer, audience, signature, and expiry on every token.
- Authorise on the server for every request; never rely on hidden UI controls.
- Apply rate limiting and lockout on authentication endpoints.

## Transport and data

- HTTPS/TLS 1.2+ everywhere; enable HSTS.
- Encrypt data at rest (Azure platform encryption by default; customer-managed keys where required).
- Minimise personal data collection; document retention and deletion for anything stored.

## Dependencies and supply chain

- Pin dependency versions and commit lock files.
- Pull Python and .NET packages from Microsoft-protected feeds
  (`packagefeedproxy.microsoft.io`) — see [`package-feeds.md`](../standards/development/package-feeds.md).
- Enable Dependabot, secret scanning with push protection, and CodeQL on every repository.
- Pin GitHub Actions to a commit SHA and grant workflows the minimum `permissions`.

## Headers and defaults

Set at minimum: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`,
`Strict-Transport-Security`, and a restrictive CORS policy (no wildcard origins with credentials).

## Review checklist

- [ ] No hardcoded credentials, keys, tokens, or connection strings.
- [ ] Managed identity used for every Azure service-to-service call that supports it.
- [ ] All input validated; all queries parameterised.
- [ ] AuthZ enforced server-side on every new endpoint.
- [ ] Errors do not leak stack traces, internal paths, or SQL to clients.
- [ ] Logs contain no secrets or personal data.
- [ ] New dependencies reviewed for vulnerabilities and licence.
