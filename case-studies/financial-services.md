# Case Study: Financial Services — MCP for Regulated Data Analytics

**Section Reference:** Paper §12.1  
**Organization Type:** Global financial services organization  
**Scale:** >10,000 employees  
**Maturity Progression:** Level 1 → Level 3 (12 months)

---

## Initial Deployment (Level 1 — Ad Hoc)

The analytics engineering team deployed MCP servers to enable AI agents to query internal financial databases and generate regulatory reports for compliance analysts. The deployment was made **without IT security involvement**.

**Governance posture at deployment:**
- No RBAC
- No registry
- No audit logging beyond server-level console output
- MCP server executed queries using a broad service account with read access to all financial tables

---

## The Near-Miss Incident (Week 6)

An AI agent generated a report containing **restricted merger and acquisition (M&A) data** for an analyst whose clearance level did not include M&A data access.

**Root cause:** Classic Confused Deputy pattern (§4.1.6)
- Analyst's RBAC role: limited to customer-facing financial data
- MCP server execution context: service account with READ on **all** financial tables
- AI agent invoked a query tool with parameters that returned M&A data
- No backend access control enforced the analyst's actual permission scope

**Potential impact if not caught:**
- Regulatory violation (insider trading regulations)
- SOC 2 audit failure
- Internal compliance breach

---

## Remediation (3 months)

Governance framework applied progressively:

### Month 1: Identity Propagation
- JWT-based identity propagation replaced service account execution for all database queries
- Backend databases now enforce the **user's** permissions, not the service account's
- Queries returning data outside the user's clearance now return filtered results or access denied

### Month 2: Tool Allowlists and Gateway
- Tool-level allowlists defined per job function, enforced at the server layer
- Centralized MCP gateway deployed with authentication enforcement
- All agent traffic routed through gateway; direct server connections blocked

### Month 3: SIEM Integration and Audit
- Audit logging schema (§9.1) implemented with SIEM integration
- Anomaly detection signals configured (sequential read-then-write for read-only roles, schema access outside baseline)
- M&A data tagged as `restricted` classification; access logged with purpose codes for SOC 2 evidence

---

## Outcomes (12-month post-remediation)

| Metric | Result |
|---|---|
| Unauthorized data access incidents | **0** |
| SOC 2 Type II audit with MCP in scope | **Passed** |
| SOC 2 auditor notes | CC6.3 and CC7.2 criteria met; logging granularity noted as exceeding minimum requirements |
| GMM Level | **Level 3** |

---

## Key Lessons

1. **The Confused Deputy is the #1 risk in financial services MCP deployments.** Service accounts with broad database access are the default — and the most dangerous configuration.

2. **Identity propagation is the single highest-impact control.** It alone would have prevented the near-miss incident. Implementation was faster than expected once JWT infrastructure was already in place for the organization's API layer.

3. **SIEM integration enabled the SOC 2 audit to include MCP in scope.** Without structured logging forwarded to a SIEM, auditors cannot collect evidence for CC6 and CC7. This transformed MCP from an audit liability to an audit-covered component.

4. **Level 1 → Level 3 took 3 months for the technical controls** and 3 additional months for the compliance review to complete. Organizations should budget 6–9 months total.
