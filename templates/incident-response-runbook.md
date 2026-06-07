# MCP Incident Response Runbook

**Section Reference:** Paper §10.5  
**Prerequisite:** Must be in place before Stage 3 (organization-wide) rollout  
**Review Cadence:** Quarterly test; update after any real incident

---

## Severity Classification

| Severity | Criteria | Response SLA |
|---|---|---|
| **Low** | Anomalous tool invocation rate; contained by rate limiting; no confirmed data exposure | 4 hours |
| **Medium** | Unauthorized tool invocation attempt; blocked by RBAC or gateway; no data exposure confirmed | 2 hours |
| **High** | Successful unauthorized tool invocation; potential data access; user session contained | 30 minutes |
| **Critical** | Confirmed data exfiltration; destructive action executed; active prompt injection campaign | 15 minutes |

---

## HIGH / CRITICAL Incident Response Steps

### Step 1: Immediate Isolation (within 15 minutes)

- [ ] Revoke affected user's **session tokens** immediately (via IdP/gateway admin)
- [ ] Revoke affected user's **MCP gateway access** (block user ID at gateway layer)
- [ ] If server-side compromise is suspected: route traffic away from the affected server (via gateway version routing policy)
- [ ] Preserve server process state **before** any restart (memory dump if malware suspected)
- [ ] Notify: Security Lead, CISO, Legal/Compliance (as required by severity)

**Decision authority for isolation:** ___________________________

### Step 2: Forensic Preservation

- [ ] Snapshot audit logs **before** any remediation that could overwrite state
- [ ] Export SIEM events for affected user ID, session token, and server ID for the 48 hours preceding detection
- [ ] Capture network flow logs for affected MCP server connections
- [ ] Export configuration state of affected MCP server at time of incident
- [ ] Do NOT restart or redeploy the affected server until forensic capture is complete (unless continued operation poses active risk)

### Step 3: Scope Determination

- [ ] Query SIEM: all tool invocations by affected user in the 30 days prior to detection
- [ ] Identify: which backend systems were reached; what data classification was accessed
- [ ] For PHI access: extract `phi_record_ids` from Data Access events for breach notification scope
- [ ] For personal data access: extract `data_subject_ids` for GDPR Article 33 notification scope
- [ ] Determine: was this an isolated incident or part of a pattern?

### Step 4: Containment

- [ ] Revoke **all credentials** exposed to the affected MCP server
- [ ] Rotate secrets for **all backend systems** the server could reach (not just confirmed affected ones)
- [ ] If token theft: force re-authentication for all users whose tokens were issued in the same time window
- [ ] If prompt injection: review and quarantine tool outputs from the session; assess context contamination

### Step 5: Notification Requirements

| Regulation | Trigger | Deadline | Recipient |
|---|---|---|---|
| GDPR Article 33 | Personal data breach | **72 hours** from awareness | Supervisory Authority |
| GDPR Article 34 | High risk to individuals | Without undue delay | Affected data subjects |
| HIPAA Breach Notification | PHI breach ≥500 individuals | **60 calendar days** | HHS + affected individuals |
| HIPAA (state law) | PHI breach | Varies by state | State AG / individuals |
| PCI-DSS 12.10.4 | CHD breach | Immediate | Payment brands + acquiring bank |

**Legal/Compliance contact for notification decisions:** ___________________________

### Step 6: Recovery

- [ ] Rebuild affected MCP server from approved immutable image (do not restore from backup of running state)
- [ ] Verify new deployment against registry baseline before returning to service
- [ ] Re-enable user access only after root cause is confirmed and controls are validated
- [ ] Update drift detection baseline to new approved configuration

### Step 7: Post-Incident Review

- [ ] Complete incident report within 5 business days
- [ ] Root cause analysis: which STRIDE threat category materialized?
- [ ] Gap analysis: which control failed or was absent?
- [ ] Update runbook based on lessons learned
- [ ] Update threat model if a new attack vector was observed
- [ ] Report to security leadership: incident timeline, scope, containment actions, remediation

---

## Specific Runbooks by Incident Type

### Runaway Agent / DoS
1. Check circuit breaker status in gateway metrics
2. If not triggered: manually suspend affected agent session via gateway
3. Identify triggering tool (highest invocation count in SIEM)
4. Apply temporary tool suspension for that tool for the affected agent role
5. Root cause: agent bug vs. prompt injection (review session context)

### Suspected Prompt Injection Campaign
1. Identify session(s) where behavior changed post-tool-invocation (SIEM: behavior anomaly signal)
2. Quarantine tool outputs from those sessions
3. Trace the tool output content that contained the adversarial instruction
4. Identify source data record (database row, document, etc.) containing the injection payload
5. Remove or sanitize the adversarial content from the data source
6. Review all agent sessions that invoked the same tool in the preceding 7 days

### Credential Exposure (Hardcoded Secret Found)
1. **Immediately revoke** the exposed credential — do not wait
2. Rotate all credentials that shared the same vault path or service account
3. Scan git history across **all repositories** for the same credential pattern
4. Identify all environments where the credential was deployed (check container registries)
5. Audit all backend systems the credential could access for unauthorized access events
6. File credential exposure security incident; notify Data Privacy Officer if PII-touching credential

---

## Contact List (fill in before Stage 3 rollout)

| Role | Name | Contact | On-call? |
|---|---|---|---|
| Security Lead (primary) | | | Yes |
| Security Lead (backup) | | | Yes |
| CISO | | | Escalation only |
| Legal / Compliance | | | As needed |
| Data Privacy Officer | | | As needed |
| Infrastructure On-call | | | Yes |
| MCP Gateway Owner | | | Yes |
