# Secrets and Credential Management Architecture

**Section Reference:** Paper §6  
**Context:** The most frequently mishandled aspect of MCP server configuration

---

## The Problem in Production

Hasan et al. [5] found **17% of examined community MCP servers contained hardcoded credentials** in source repositories. Rexhepi [15] documents AWS access keys embedded directly in MCP JSON configuration files appearing in public GitHub repositories — with credentials committed to version control, synced to developer machines, and included in container images.

The blast radius is amplified because **a single MCP server process often holds credentials for multiple downstream systems simultaneously.**

---

## Anti-Patterns to Avoid [PROHIBITED]

The following patterns are explicitly prohibited by organizational policy:

| Anti-Pattern | Why It's Catastrophic |
|---|---|
| Hardcoding secrets in MCP config JSON files | Committed to VCS → distributed to all developer machines → included in container images |
| Secrets in environment variables on shared systems | Any process running as same user can read them |
| Single service account across multiple MCP servers | One compromise = full revocation = multi-service outage |
| Embedding credentials in container images | Images distributed through registries accessible to unintended parties |
| Static credentials with no expiry or rotation | Persist indefinitely after initial compromise without detection |

---

## Required Architecture: Zero-Trust Credential Model

MCP servers retrieve credentials **dynamically at runtime** rather than storing them statically in any persistent form.

### Component 1: Secrets Vault Integration [REQUIRED]

```
┌─────────────────────┐    short-lived machine     ┌─────────────────┐
│   MCP Server        │ ──identity token──────────▶ │  Secrets Vault  │
│  (Kubernetes pod)   │                             │  (HashiCorp /   │
│                     │ ◀──credential (TTL: 15m)─── │   AWS / Azure)  │
└─────────────────────┘                             └─────────────────┘
         │
         │ credential used for backend call
         ▼
┌─────────────────────┐
│  Backend System     │
│  (DB, API, etc.)    │
└─────────────────────┘
```

**Implementation:**
- MCP servers authenticate to vault using Kubernetes workload identity or AWS instance metadata (no static vault token)
- Secrets fetched at server startup or per-invocation (depending on secret type)
- Secrets **never written to disk**
- Vault access token TTL: 15–60 minutes, automatically refreshed

**Supported Vault Platforms:**
- HashiCorp Vault (open source or Enterprise)
- AWS Secrets Manager
- Azure Key Vault
- GCP Secret Manager

---

### Component 2: Least Privilege Service Accounts [REQUIRED]

- **Each MCP server runs under a unique service identity** with only the permissions required for its declared tool set
- Shared service accounts across multiple MCP servers: **PROHIBITED**
- Service account permissions must be validated against the tool schema at deployment time
- Any service account holding permissions beyond what the tool schema declares → deployment blocked

**Enforcement mechanism:** CI/CD pipeline performs permission scope validation before image promotion to production registry.

---

### Component 3: Secret Rotation Policies [REQUIRED]

| Secret Type | Maximum Rotation Interval | Grace Period |
|---|---|---|
| Database credentials | 30 days | 2 hours |
| API keys | 90 days | 4 hours |
| OAuth client secrets | 90 days | 4 hours |
| Internal service tokens | 7 days | 1 hour |
| mTLS certificates | 1 year | 7 days |

**Grace period:** Both old and new credentials remain valid during the transition window to avoid service disruption during rotation. Old credential is invalidated after the grace period expires.

**Automation requirement:** Rotation must be fully automated. Manual rotation processes are not compliant because they are unreliable and create audit gaps.

---

### Component 4: Credential Scanning in CI/CD [REQUIRED]

Credential detection must occur at multiple pipeline stages:

```
git commit → [pre-commit hook: gitleaks/git-secrets] → BLOCK if cred detected
     │
     ▼
CI pipeline → [SAST scan: truffleHog on full git history] → BLOCK if cred detected
     │
     ▼
Container build → [image scan: Trivy/Grype] → BLOCK if cred in image layers
     │
     ▼
Registry push → [registry policy: signature verification] → BLOCK if unsigned
```

**Tools:**
- Pre-commit: `git-secrets`, `gitleaks`, `detect-secrets`
- CI pipeline: `truffleHog`, `gitleaks` (with `--no-git` for arbitrary file scanning)
- Image scanning: `Trivy`, `Grype` (with secret detection mode)
- Registry enforcement: Cosign (for image signing verification)

**Critical requirement:** Scanning must cover **git history**, not just the current commit. Credentials committed and then "deleted" in a later commit remain in git history and are accessible.

---

## Configuration File Security

MCP configuration files that contain **no secrets** must still be treated as sensitive infrastructure artifacts. They reveal:
- Which backend systems exist
- What operations are supported
- What tool schemas are exposed

All configuration files must be:
- Stored in configuration management systems with access controls and audit logging
- Versioned and subject to change approval workflows
- **Not distributed ad hoc** by individual developers outside the approved system

### Local Workstation Configuration

For stdio-based servers on developer workstations:
- IT management tools (Intune, Jamf, Ansible) push approved MCP configurations to **protected locations**
- Developers have **read but not write access** to production configuration locations
- A separate developer-owned "sandbox" configuration allows experimentation
- New server requests go through the standard approval process (§5.1); once approved, IT pushes the update

---

## Quick Reference: Credential Do/Don't

```diff
# DO
+ Fetch credentials from vault at runtime using workload identity
+ Use separate service accounts per MCP server
+ Rotate credentials on defined schedule with grace periods
+ Scan all commits, history, and images for credential patterns
+ Store config files in version-controlled, access-controlled systems

# DON'T
- Hardcode credentials in config JSON, source code, or Dockerfiles
- Use shared service accounts across multiple MCP servers
- Store credentials in environment variables on shared hosts
- Commit config files with architectural information to public repos
- Skip rotation because "it's complex" — automate it instead
```

---

## References
- Paper §6.1–6.3
- [5] Hasan et al., arXiv:2506.13538
- [15] Rexhepi, Tech with Orgito, Mar. 2026
- [8] NIST SP 800-207 (Zero Trust Architecture)
