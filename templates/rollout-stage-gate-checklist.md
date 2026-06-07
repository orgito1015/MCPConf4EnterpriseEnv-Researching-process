# Staged Rollout Stage-Gate Checklist

**Section Reference:** Paper §10.1  
**Purpose:** Enforce stage-gate advancement criteria before expanding MCP server rollout  
**Instruction:** ALL items in each stage must be checked before advancing to the next stage. These are not timelines to be bypassed.

---

## Pre-Deployment: Governance Prerequisites

Before any rollout stage begins, verify:

- [ ] Server registered in central MCP registry with approved status
- [ ] Security review completed and documented
- [ ] Data classification assessment recorded in registry entry
- [ ] Business justification and designated owner assigned
- [ ] Approval obtained from: IT Security / Data Privacy Officer / Business Unit Owner
- [ ] Tool schema hash recorded in registry
- [ ] Secrets management configured (vault integration, no hardcoded credentials)
- [ ] RBAC policy configured (role allowlists, tool filtering at transport layer)
- [ ] Audit logging configured and forwarding to SIEM (or log management for Stage 1)
- [ ] Rollback procedure documented and tested
- [ ] Container image signed (for SSE/HTTP servers)

---

## Stage 1: Controlled Pilot (2–4 weeks)

**Target population:** 5–20 users from the primary business unit

### Setup Checklist
- [ ] Pilot user group identified and notified
- [ ] Security monitoring plan defined (daily manual log review assigned)
- [ ] Weekly security review meeting scheduled
- [ ] Helpdesk contact for MCP issues designated
- [ ] Rollback trigger criteria defined in writing

### Monitoring During Stage 1
- [ ] Daily audit log review performed by security team member
- [ ] Weekly security review meeting held with documented minutes
- [ ] All anomalies logged and investigated

### Advancement Criteria (must ALL be met before Stage 2)
- [ ] Zero policy violations in the final week of the pilot
- [ ] All audit log anomalies have documented explanations
- [ ] Helpdesk incident rate within acceptable bounds (define threshold: _______)
- [ ] No unauthorized tool invocations detected
- [ ] No credential exposure incidents

**Sign-off:**  
Security Lead: _____________________ Date: _________  
Business Unit Owner: _____________________ Date: _________

---

## Stage 2: Departmental Rollout (4–8 weeks)

**Target population:** Full department or business unit (50–200 users)

### Prerequisites (must be completed BEFORE starting Stage 2)
- [ ] SIEM alerting configured and validated with known-good test cases
- [ ] SIEM alerting validated with known-bad test cases (inject anomaly, confirm alert fires)
- [ ] All 6 anomaly signals from §9.3 configured in SIEM
- [ ] IT helpdesk support procedures for MCP-related issues documented
- [ ] Helpdesk staff trained on MCP incident handling
- [ ] Rate limiting enforced at gateway layer
- [ ] Circuit breaker pattern configured

### Monitoring During Stage 2
- [ ] SIEM alerts reviewed on defined SLA (target: <15 min response for HIGH/CRITICAL)
- [ ] Weekly security review meeting continues
- [ ] Audit log completeness rate tracked (target: >99%)

### Advancement Criteria (must ALL be met before Stage 3)
- [ ] SIEM alerting validated (documented test results attached)
- [ ] Zero critical security incidents during departmental rollout
- [ ] Compliance team sign-off on audit log completeness
- [ ] Helpdesk runbooks tested and validated
- [ ] Drift detection configured and tested

**Sign-off:**  
Security Lead: _____________________ Date: _________  
Compliance Team: _____________________ Date: _________  
Business Unit Owner: _____________________ Date: _________

---

## Stage 3: Organization-Wide Deployment (ongoing)

**Target population:** All authorized users across the organization

### Prerequisites (must be completed BEFORE starting Stage 3)
- [ ] Full SIEM integration active with documented detection rules for ALL anomaly signals
- [ ] Runbooks for all anticipated incident types in place and tested:
  - [ ] Unauthorized tool invocation runbook
  - [ ] Token theft / session compromise runbook
  - [ ] Prompt injection campaign runbook
  - [ ] Runaway agent / DoS runbook
  - [ ] Credential exposure runbook
  - [ ] Audit log integrity failure runbook
- [ ] Compliance review completed for ALL applicable frameworks:
  - [ ] HIPAA (if applicable)
  - [ ] GDPR (if applicable)
  - [ ] SOC 2 (if applicable)
  - [ ] PCI-DSS (if applicable)
- [ ] Severity classification and escalation matrix defined and distributed
- [ ] On-call rotation established for security alerts

### Ongoing Monitoring (Stage 3 steady state)
- [ ] SIEM alerts reviewed per defined SLA
- [ ] Quarterly security review of MCP infrastructure (replaces weekly once stable)
- [ ] Monthly KPI report to security leadership
- [ ] Annual compliance audit with MCP in scope

### KPI Targets (from `../maturity-model/kpi-framework.md`)
- Unauthorized Tool Access Incidents: 0 per month
- Audit Log Coverage Rate: >99%
- Policy Compliance Rate: >95%
- Shadow MCP Server Detection: 0 undetected
- Tool Approval SLA Adherence: >90%

**Sign-off:**  
CISO / Security Leadership: _____________________ Date: _________  
Compliance Officer: _____________________ Date: _________  
IT Operations Lead: _____________________ Date: _________

---

## Emergency Rollback Procedure

If a critical security incident is detected, rollback must be executable within 15 minutes:

1. **Decision authority:** _____________________ (define who can authorize rollback)
2. **Rollback trigger criteria:** (document specific conditions that require rollback)
3. **Rollback steps:** (link to version-controlled rollback runbook)
4. **Communication:** Notify _____________________ within _____ minutes of rollback decision
5. **Post-rollback:** Document within 2 hours; retrospective review within 24 hours

---

## Related Documents
- [`mcp-server-registry-entry.json`](mcp-server-registry-entry.json) — Registry entry template
- [`tool-schema-review-checklist.md`](tool-schema-review-checklist.md) — Pre-deployment tool review
- [`incident-response-runbook.md`](incident-response-runbook.md) — Incident response procedures
- [`../compliance/audit-logging-schema.md`](../compliance/audit-logging-schema.md) — Audit event requirements
