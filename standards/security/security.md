# Security standards

Normative rules for every repository and workload. The agent-facing summary lives in
[`instructions/security.instructions.md`](../../instructions/security.instructions.md).

## 1. Identity

- Microsoft Entra ID is the only identity provider for users and workloads.
- Workloads authenticate with **managed identities** (user-assigned by default). Service principal secrets require
  a documented exception with an expiry date.
- GitHub Actions authenticate to Azure with **workload identity federation (OIDC)**. No stored Azure credentials.
- Access is least privilege, assigned to groups, reviewed quarterly, and granted with built-in roles at the
  narrowest resource scope.

## 2. Secrets

| Rule | Detail |
| --- | --- |
| Storage | Azure Key Vault; referenced by the app, never copied into configuration files |
| Access | Managed identity + `Key Vault Secrets User`; purge protection and soft delete enabled |
| Rotation | At most 90 days, automated where the service supports it |
| Detection | GitHub secret scanning with push protection enabled on every repository |
| Leak response | Rotate immediately, revoke, audit usage, then remove from history |

## 3. Secure development

- Threat model any new externally reachable component or trust boundary.
- Validate all input at the boundary; encode all output by context.
- Parameterised queries only. No dynamic SQL from user input, including column names.
- Cryptography: TLS 1.2+, AES-256 at rest, SHA-256+, `secrets`/`RandomNumberGenerator` for tokens.
  Never implement your own cryptographic primitive.
- Errors return a correlation id, not a stack trace.

## 4. Supply chain

- Pin dependency versions and commit lock files.
- Resolve Python and .NET packages from the Microsoft-protected feeds
  (see [`development/package-feeds.md`](../development/package-feeds.md)).
- Enable Dependabot alerts and security updates; remediate critical findings within 7 days, high within 30.
- Pin GitHub Actions to a full commit SHA; set explicit minimum `permissions` on every workflow.
- Generate an SBOM for released artefacts.

## 5. Required repository controls

- [ ] Branch protection on the default branch: review required, status checks required, no force push.
- [ ] CodeQL code scanning enabled.
- [ ] Secret scanning + push protection enabled.
- [ ] Dependabot enabled.
- [ ] `SECURITY.md` with a disclosure contact.

## 6. Runtime hardening

- Public network access disabled on data stores; private endpoints where available.
- WAF in front of internet-facing applications; HSTS and a strict `Content-Security-Policy`.
- Containers run as a non-root user with a read-only root filesystem where possible; base images patched monthly.
- Diagnostic logs and audit logs shipped to Log Analytics with a defined retention period.

## 7. Incident response

Detect (Defender for Cloud alerts) → contain (revoke identity/rotate secret) → eradicate → recover → post-mortem
within five working days. Record the timeline and the corrective actions.
