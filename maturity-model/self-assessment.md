# MCP-GMM Self-Assessment Questionnaire

**Purpose:** Determine your organization's current MCP governance maturity level.  
**Instructions:** Answer each question honestly with Yes or No based on current state, not planned state.

---

## Assessment Questions

### Gate 1 — Level 2 Minimum

**Q1.** Is there a central registry of all MCP servers deployed in the organization, with approval status and owner recorded for each?

- [ ] Yes
- [ ] No

---

### Gate 2 — Level 3 Minimum (Part A)

**Q2.** Is RBAC enforced at both the MCP server layer (tool filtering at transport layer) AND the backend layer (identity propagation), with no direct agent-to-server connections permitted outside the gateway?

- [ ] Yes
- [ ] No

---

### Gate 3 — Level 3 Minimum (Part B)

**Q3.** Are all MCP audit logs forwarded to a SIEM in real time, with automated alerting configured for the following anomaly signals?
- Tool invocation rate >3× 7-day rolling baseline
- Data schema access outside user's historical access pattern
- Parameter values containing injection-characteristic patterns
- Sequential read-then-write by read-only authorized user
- Single session token observed from multiple source IPs
- Configuration change without corresponding approval ticket

- [ ] Yes (all signals configured)
- [ ] Partial (some signals configured)
- [ ] No

---

### Gate 4 — Level 4 Minimum

**Q4.** Are security KPIs for MCP operations (MTTD for misconfigurations, unauthorized invocation attempts, audit log completeness) tracked and reviewed by security leadership at least monthly?

- [ ] Yes
- [ ] No

---

### Gate 5 — Level 5 Minimum

**Q5.** Is there an automated policy enforcement engine that blocks non-compliant MCP server deployments before they reach production, with policy defined as code in version control?

- [ ] Yes
- [ ] No

---

## Scoring

Count your "Yes" answers:

| Yes Count | Maturity Level | Regulatory Defensibility |
|---|---|---|
| 0 | **Level 1 — Ad Hoc** | ❌ Not defensible |
| 1 | **Level 2 — Managed** | ⚠️ Insufficient for HIPAA/SOC 2/GDPR |
| 2–3 | **Level 3 or 4 — Defined/Measured** | ✅ See note below |
| 4 | **Level 4 — Measured** | ✅✅ Audit-ready |
| 5 | **Level 5 — Optimizing** | ✅✅✅ Exceeds requirements |

> **Note on Level 3 vs. Level 4 (2–3 Yes answers):** Distinguishing between Level 3 and Level 4 requires additional assessment:
> - Has your organization passed an annual compliance audit with MCP infrastructure in scope? → Level 4
> - Is your MTTD for misconfigurations consistently under 1 hour? → Level 4
> - If neither: → Level 3

---

## Detailed Gap Analysis

For each "No" answer, use the table below to identify the highest-priority remediation actions:

| Question | Gap | Priority Actions | Timeline |
|---|---|---|---|
| Q1 (Registry) | No central registry | Establish registry; implement approval workflow; deploy gateway with auth enforcement | 3–6 months |
| Q2 (RBAC) | Incomplete RBAC | Implement backend identity propagation; configure transport-layer tool filtering; enforce gateway routing | 6–12 months |
| Q3 (SIEM) | No/partial SIEM integration | Integrate SIEM; configure all 6 anomaly signals; validate with test cases | 3–6 months |
| Q4 (KPIs) | No KPI tracking | Define KPIs from `kpi-framework.md`; instrument measurement; establish monthly review cadence | 2–4 months |
| Q5 (Policy-as-Code) | No automated enforcement | Implement policy-as-code in CI/CD; deploy per-invocation risk scoring | 12+ months |

---

## Additional Diagnostic Questions

These questions do not affect scoring but help prioritize remediation:

**Shadow IT**
- Have unauthorized MCP servers been discovered in network traffic analysis in the past 6 months? (Yes = urgent registry gap)

**Credential Security**
- Have any MCP server credentials been found in source code repositories or configuration files outside the secrets vault? (Yes = Hasan et al. [5] pattern; urgent credential remediation)

**Incident History**
- Has a Confused Deputy-style unauthorized data access incident (or near-miss) occurred in the past 12 months? (Yes = RBAC gap confirmed)

**Regulatory Scope**
- Are any MCP servers in scope for HIPAA, GDPR, SOC 2, or PCI-DSS audits? (Yes = Level 3 is the minimum required posture)

---

## Next Steps by Level

**Currently at Level 1 or 2** → Read [`advancement-roadmap.md`](advancement-roadmap.md) for the Level 1→3 path  
**Currently at Level 3** → Read [`kpi-framework.md`](kpi-framework.md) to begin Level 4 advancement  
**Currently at Level 4** → Consult `../policy-framework/` for policy-as-code implementation  
**Currently at Level 5** → Consider contributing empirical outcome data to future research validation

---

## Interactive Version

For a browser-based interactive version of this assessment with automatic scoring, see:  
[`../tools/maturity-self-assessment.html`](../tools/maturity-self-assessment.html)
