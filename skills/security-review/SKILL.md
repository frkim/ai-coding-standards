---
name: security-review
description: Perform a threat-driven security review of a change set, covering secrets, identity, input validation, dependencies, and Azure configuration. Use when reviewing code for vulnerabilities or hardening a workload.
---

# Security review skill

Review against [`instructions/security.instructions.md`](../../instructions/security.instructions.md).
Report only findings you can justify from the code; each finding gets a severity, evidence, and a concrete fix.

## Process

1. **Scope** — list the changed files and identify trust boundaries crossed (network, user input, storage, identity).
2. **Threat model quickly** — for each boundary ask: spoofing, tampering, repudiation, information disclosure,
   denial of service, elevation of privilege (STRIDE).
3. **Inspect** the categories below.
4. **Report** findings ordered by severity with file/line evidence and a suggested patch.

## Categories

### Secrets and identity
- Hardcoded keys, tokens, passwords, connection strings, certificates (including in tests and fixtures).
- Azure calls not using managed identity where one is available.
- Over-broad RBAC (`Contributor`/`Owner`), subscription-scoped assignments, long-lived service principal secrets.

### Input and output
- Unvalidated request data reaching a query, file path, command, or template.
- SQL/NoSQL/command/LDAP injection; path traversal; SSRF from user-supplied URLs.
- Missing output encoding, `dangerouslySetInnerHTML`/`v-html` with untrusted data.
- Deserialisation of untrusted data.

### AuthN / AuthZ
- Endpoints missing authorisation, or authorisation based on client-supplied identity.
- Missing object-level checks (IDOR): does the caller own the record they requested?
- Token validation gaps: issuer, audience, expiry, signature, algorithm confusion.

### Data protection
- Sensitive data in logs, telemetry, error responses, or URLs.
- Missing encryption in transit/at rest; weak or custom cryptography; `Random` used for tokens.

### Dependencies and CI
- Unpinned or vulnerable dependencies; public PyPI/NuGet endpoints instead of `packagefeedproxy.microsoft.io`.
- Unpinned GitHub Actions, `pull_request_target` with checkout of untrusted code, excessive workflow `permissions`.
- Actions or custom actions still on the deprecated Node 20 runtime (`runs.using: node20`) instead of `node24`.

### Configuration
- Debug mode, verbose errors, or permissive CORS in production settings.
- Public network access on data stores; missing WAF/TLS/HSTS.

## Severity

| Severity | Meaning |
| --- | --- |
| Critical | Remote unauthenticated compromise, secret exposure in a shipped artefact |
| High | Authenticated privilege escalation, injection, broken access control |
| Medium | Missing hardening with a plausible exploit path |
| Low | Defence in depth, hygiene |

## Output format

```markdown
### [High] Missing object-level authorization in `customers_router.py:48`
**Evidence**: the handler loads the customer by id from the path without checking tenant ownership.
**Impact**: any authenticated user can read any tenant's customer.
**Fix**: filter the query by `current_user.tenant_id` and return 404 when it does not match.
```

End with an explicit statement when no issues are found in a category — silence is ambiguous.
