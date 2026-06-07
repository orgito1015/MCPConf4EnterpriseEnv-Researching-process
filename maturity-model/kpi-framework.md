# Operational KPI Framework for Enterprise MCP Governance

**Section Reference:** Paper §12.6 (Table 9)  
**Minimum Maturity Level:** Level 3 (tracking); Level 4 (achieving targets)  
**Review Cadence:** Monthly tracking, quarterly report to security leadership

---

## KPI Definitions and Targets

| KPI | Target | Measurement Source | Min. GMM Level | Escalation Trigger |
|---|---|---|---|---|
| Unauthorized Tool Access Incidents | 0 per month | SIEM alert log | Level 3 | Any single incident |
| Audit Log Coverage Rate | >99% | SIEM ingestion vs. gateway event count | Level 3 | Drop below 95% |
| Config Drift Detection Time (MTTD) | <1 hour | Drift detection alert log | Level 4 | MTTD >4 hours |
| Critical Incident MTTD | <15 minutes | SIEM alert timestamp vs. triage open ticket | Level 4 | MTTD >30 min |
| Policy Compliance Rate | >95% | Config scan vs. approved baseline | Level 3 | Drop below 90% |
| Shadow MCP Server Detection | 0 undetected servers | Network traffic analysis vs. registry | Level 3 | Any unregistered server found |
| Tool Approval SLA Adherence | >90% within SLA | Approval ticket system | Level 3 | Drop below 80% |

---

## KPI Detail Cards

### KPI 1: Unauthorized Tool Access Incidents

**Definition:** Count of confirmed unauthorized tool invocations (invocations that bypassed or violated RBAC policy) in the measurement period.

**How to Measure:**
- Source: SIEM alert log, filtered to `event_type: authorization_decision` where `decision: deny` was preceded by an actual invocation (i.e., the invocation was attempted and executed before denial — indicating a control failure)
- Distinguish from: authorization denials that were correctly caught at the gateway/transport layer before execution (these are expected and indicate controls are working)

**Target:** 0 per month

**Escalation:** Any confirmed unauthorized execution → immediate security incident (HIGH severity per §10.5 runbook)

---

### KPI 2: Audit Log Coverage Rate

**Definition:** Percentage of tool invocations recorded by the MCP gateway that have a corresponding audit log entry in the SIEM.

**How to Measure:**
- Gateway event count (source of truth): pull total invocation count from gateway metrics for the measurement period
- SIEM ingestion count: query SIEM for `event_type: tool_invocation` count for same period
- Coverage Rate = (SIEM count / Gateway count) × 100

**Target:** >99%

**Escalation:** Drop below 95% → investigate log pipeline; potential compliance gap

**Note:** A coverage rate of <99% could indicate: log pipeline latency/failure; MCP servers bypassing the gateway (shadow server indicator); log redaction errors dropping events.

---

### KPI 3: Configuration Drift Detection MTTD

**Definition:** Mean time from a configuration drift event occurring (deployed config diverges from registry-approved baseline) to the security team receiving an alert.

**How to Measure:**
- Inject a known drift event in a non-production environment on a randomized schedule
- Record time from injection to alert receipt
- Calculate mean across test events per quarter

**Target:** <1 hour

**Escalation:** MTTD >4 hours → drift detection system review required; potential Level 3 regression

---

### KPI 4: Critical Incident Mean Time to Detect (MTTD)

**Definition:** Mean time from a critical security event occurring (e.g., unauthorized data access, active prompt injection) to the security operations team opening a triage ticket.

**How to Measure:**
- Source: SIEM alert timestamps vs. ticketing system (JIRA/ServiceNow) create timestamps
- Filter: tickets with severity classification HIGH or CRITICAL
- Calculate mean across incidents per measurement period

**Target:** <15 minutes

**Escalation:** MTTD >30 minutes → SIEM alert configuration review; on-call rotation review

---

### KPI 5: Policy Compliance Rate

**Definition:** Percentage of deployed MCP server configurations that match their registry-approved baseline, measured by automated drift detection scans.

**How to Measure:**
- Run automated config scan against all registered servers (minimum every 15 minutes)
- Compliance Rate = (servers matching baseline / total registered servers) × 100
- Use point-in-time measurement at end of each month; also track time-averaged compliance

**Target:** >95%

**Escalation:** Drop below 90% → compliance gap; review drift detection and change management processes

---

### KPI 6: Shadow MCP Server Detection

**Definition:** Number of MCP servers discovered in network traffic analysis that are NOT registered in the central MCP registry.

**How to Measure:**
- Source: Network traffic analysis / NGFW logs — filter for MCP protocol traffic patterns
- Cross-reference source/destination pairs against registry endpoint inventory
- Unmatched MCP traffic sources = shadow servers

**Target:** 0 undetected

**Escalation:** Any unregistered server → immediate security incident; shadow server remediation within 24 hours or decommission

---

### KPI 7: Tool Approval SLA Adherence

**Definition:** Percentage of MCP tool approval requests fulfilled within the committed SLA (5 business days for standard tools; 10 business days for restricted-data tools).

**How to Measure:**
- Source: Approval ticketing system
- SLA Adherence = (tickets closed within SLA / total tickets closed in period) × 100
- Track separately for standard vs. restricted-data requests

**Target:** >90% overall

**Escalation:** Drop below 80% → security team capacity review; this KPI directly measures shadow IT pressure (poor SLA adherence is a leading indicator of shadow deployments)

**Note from case study (§12.3):** SLA adherence below 80% was strongly correlated with shadow MCP server proliferation. Improving SLA from unconstrained to a 5-day commitment reduced shadow server incidence substantially.

---

## Monthly KPI Report Template

```
MCP Governance KPI Report — [Month Year]
Reported by: [Security Team]
Reviewed by: [CISO / Security Leadership]

┌─────────────────────────────────┬──────────┬──────────┬──────────┐
│ KPI                             │ Target   │ Actual   │ Status   │
├─────────────────────────────────┼──────────┼──────────┼──────────┤
│ Unauthorized Tool Access        │ 0        │          │ 🟢/🔴    │
│ Audit Log Coverage Rate         │ >99%     │          │ 🟢/🔴    │
│ Config Drift MTTD               │ <1 hr    │          │ 🟢/🔴    │
│ Critical Incident MTTD          │ <15 min  │          │ 🟢/🔴    │
│ Policy Compliance Rate          │ >95%     │          │ 🟢/🔴    │
│ Shadow MCP Servers              │ 0        │          │ 🟢/🔴    │
│ Tool Approval SLA Adherence     │ >90%     │          │ 🟢/🔴    │
└─────────────────────────────────┴──────────┴──────────┴──────────┘

Incidents this period: [count and brief summary]
Escalations triggered: [list]
Trend vs. prior month: [improving / stable / degrading]
Next actions: [list]
```

---

## Related Documents
- [`MCP-GMM.md`](MCP-GMM.md) — Maturity model definitions
- [`self-assessment.md`](self-assessment.md) — Governance maturity self-assessment
- [`../compliance/audit-logging-schema.md`](../compliance/audit-logging-schema.md) — Event schema for SIEM queries
- [`../templates/incident-response-runbook.md`](../templates/incident-response-runbook.md) — Escalation procedures
