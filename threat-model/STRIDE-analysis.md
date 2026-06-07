# STRIDE Threat Analysis — MCP Architectural Boundaries

**Section Reference:** Paper §4  
**Methodology:** Microsoft STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege)

---

## Architectural Boundaries Analyzed

STRIDE was applied at each of the four MCP trust boundaries:

1. **User ↔ MCP Host** — identity, session, and context integrity
2. **MCP Host ↔ MCP Server** — transport security and tool invocation integrity
3. **MCP Server ↔ Backend Resources** — data access and credential security
4. **MCP Registry ↔ MCP Server** — configuration integrity and approval enforcement

---

## S — Spoofing: Identity Fraud

**Attack Scenario**  
An attacker compromises a developer's endpoint and uses a cached OAuth token to invoke MCP tools under the developer's identity, accessing data the attacker is unauthorized to view. Because MCP's stdio transport inherits the client's process credentials and environment variables, a compromised client process immediately grants the attacker the full MCP access of that user.

**Risk Level:** 🔴 CRITICAL  
**Likelihood:** High (token theft via endpoint compromise is well-documented; long-TTL cached OAuth tokens common in dev environments)  
**Impact:** High (attacker gains full MCP tool access and data permissions of the user)

**Required Mitigations**
- Short-lived tokens (<5 minutes) with continuous re-authentication at the MCP server layer, independent of transport-layer session persistence
- Device trust scoring integrating endpoint health signals into authorization decisions, with revocation for untrusted devices

**Recommended Mitigations**
- Hardware-backed 2FA (YubiKey, platform authenticator) for high-sensitivity tool categories

**Detection Signals**
- Alert when a single token is observed from multiple source IPs within 5 minutes
- Alert on access from new geographic location outside established baselines

---

## T — Tampering: Unauthorized Modification

**Attack Scenario**  
An attacker gains write access to an MCP tool code repository and modifies a database query tool to exfiltrate additional customer columns alongside authorized query results. The tool continues to function normally while silently broadening its data access. Detection is difficult because results appear structurally valid.

**Risk Level:** 🔴 CRITICAL  
**Likelihood:** Medium (Hasan et al. [5]: 17% of MCP servers had credentials hardcoded in source repositories, indicating repositories are actively targeted)  
**Impact:** Critical (undetected data exfiltration; business logic corruption)

**Required Mitigations**
- All code changes require pull request review with mandatory security team sign-off for any changes affecting data access scope or tool schema
- SAST scanning on all commits with blocking on high-severity findings
- Immutable container images for MCP server deployment — no runtime modification permitted
- Cryptographic image signing with registry-enforced signature verification before deployment

---

## R — Repudiation: Denial of Responsibility

**Attack Scenario**  
An AI agent deletes customer records. The user whose credentials were used claims their account was compromised. Without non-repudiation controls, the organization cannot prove the user initiated the operation, creating liability exposure and making breach investigation impossible.

**Risk Level:** 🟠 HIGH  
**Likelihood:** Medium (increasingly common as AI agents perform consequential autonomous operations)  
**Impact:** High (liability disputes, contractual repudiation, audit failures, compliance violations)

**Required Mitigations**
- Explicit user confirmation required before destructive operations, with the confirmation event logged independently of the operation itself
- Write-once, append-only audit log storage ensuring logs cannot be modified after creation

**Recommended Mitigations**
- Cryptographic signing of audit log chains, where user identity, operation parameters, timestamp, and result are all signed with a user-controlled key

---

## I — Information Disclosure: Confidentiality Breach

Two distinct sub-threats apply:

### Sub-Threat A: Unauthorized Data Access

**Attack Scenario**  
A user with scoped permissions crafts tool invocation parameters that return data beyond their clearance level. Enabled by the Confused Deputy pattern when tool execution uses service account credentials rather than user credentials.

**Risk Level:** 🔴 CRITICAL  
**Likelihood:** High (Rexhepi [15]: broad service account pattern extremely common in production)  
**Impact:** Critical (data breach, privacy violation, regulatory fine)

**Required Mitigations**
- RBAC at backend layer with database row-level security filtering results per user identity, not service account identity
- MCP server validates query scope against user permissions before execution

### Sub-Threat B: Prompt Injection via Tool Output

**Attack Scenario**  
A data record controlled by an attacker (a customer record, a document, a database row) contains adversarial instructions. When retrieved by an MCP tool and incorporated into the model's context, these instructions redirect AI agent behavior — causing it to exfiltrate data, invoke additional tools, or modify state on the attacker's behalf.

**Empirical Basis**  
- Greshake et al. [12]: demonstrated this attack class against real LLM-integrated applications
- Kang et al. [13]: 60%+ of deployed LLM-integrated applications susceptible to at least one indirect prompt injection variant

**Risk Level:** 🔴 CRITICAL  
**Likelihood:** High  
**Impact:** Critical

**Required Mitigations**
- Tool output treated as untrusted data within the AI framework context, never elevated to instruction-level authority

**Recommended Mitigations**
- Behavioral monitoring detecting unexpected agent behavior changes post-tool-invocation
- Output sanitization scanning for adversarial instruction patterns before context incorporation

---

## D — Denial of Service: Availability

**Attack Scenario**  
An AI agent bug or injected instruction causes a runaway loop invoking expensive MCP tools at high frequency — thousands of database queries or API calls per minute — exhausting server resources, backend rate limits, and incurring significant cost. LLM-mediated runaway loops are harder to detect and terminate than equivalent bugs in conventional software.

**Risk Level:** 🟡 MEDIUM  
**Likelihood:** Medium (AI agent runaway loops are a documented operational failure mode)  
**Impact:** Medium (business continuity disruption, financial cost, backend system degradation)

**Required Mitigations**
- Per-user/per-agent rate limiting enforced at the gateway layer (e.g., maximum 100 tool invocations per minute per agent session)
- Circuit breaker pattern: automatic tool suspension after N consecutive failures within a time window
- Total invocation cost budgets per session, with alerts and automatic session termination on breach

---

## E — Elevation of Privilege: Confused Deputy

**Attack Scenario**  
A customer service agent has "read customer data" permission in organizational RBAC. However, the MCP server is configured to execute backend database calls using a service account that holds UPDATE and DELETE permissions. The AI agent (or an attacker manipulating it via prompt injection) discovers the tool accepts write-pattern parameters and executes destructive operations unintended by the agent's role assignment.

This is the **Confused Deputy problem** applied to MCP: the MCP server acts as a trusted intermediary that inadvertently exercises permissions beyond what the calling principal is authorized to use.

**Risk Level:** 🔴 CRITICAL  
**Likelihood:** High (Rexhepi [15]: broad service accounts are one of the primary configuration failures in production deployments)  
**Impact:** Critical (unintended data modification or deletion; privilege escalation beyond authorized scope)

**Required Mitigations**
- **Identity propagation:** the MCP server must forward the user's authentication token to backend systems rather than substituting a service account
- **Least privilege:** any service account used by an MCP server must have only the permissions declared in the server's tool schema, validated at deployment time
- **Tool schema scope declaration:** each tool explicitly declares its maximum permission scope, reviewed and approved before deployment

---

## Risk Quantification Matrix

| Threat Vector | Likelihood | Impact | Risk Level | Empirical Basis |
|---|---|---|---|---|
| User impersonation (token theft) | High | High | 🔴 CRITICAL | Endpoint compromise prevalence [13] |
| Tool code tampering | Medium | Critical | 🔴 CRITICAL | 17% servers had hardcoded creds [5] |
| Unauthorized data access (RBAC bypass) | High | Critical | 🔴 CRITICAL | Confused Deputy pattern [15] |
| Prompt injection via tool output | High | High | 🔴 CRITICAL | 60%+ apps vulnerable [13] |
| Privilege escalation (Confused Deputy) | High | Critical | 🔴 CRITICAL | Broad service accounts common [15] |
| Audit log tampering | Medium | Critical | 🔴 CRITICAL | No tamper-evident logging in most deployments |
| Runaway tool invocation (DoS) | Medium | Medium | 🟡 MEDIUM | AI agent loop failures documented |
| Configuration drift | Medium | Medium | 🟡 MEDIUM | No centralized config management [15] |
| Tool schema abuse | Medium | Medium | 🟡 MEDIUM | 58% no input validation [5] |

---

## Key Insight: Non-Deterministic Caller

A fundamental distinction between MCP and conventional API security is the **non-deterministic caller**:

- A traditional API client's behavior under adversarial input can be precisely modeled and bounded.
- An LLM's tool invocation behavior is context-sensitive and non-deterministic — creating manipulation opportunities (prompt injection, tool chaining, schema abuse) that have **no analog** in conventional microservice security.

This means that controls sufficient for API security (authentication, rate limiting, schema validation) are **necessary but not sufficient** for MCP security. Behavioral monitoring, output trust tagging, and LLM-specific anomaly detection are additionally required.

---

## References

- [5] Hasan et al., arXiv:2506.13538
- [12] Greshake et al., AISec 2023
- [13] Kang et al., IEEE S&P 2024
- [14] Microsoft STRIDE Methodology
- [15] Rexhepi, Tech with Orgito, 2026
