# Audit Logging Schema — Enterprise MCP

**Section Reference:** Paper §9  
**Requirement Level:** REQUIRED for all regulated enterprise MCP deployments

---

## Why Comprehensive Audit Logging is Non-Negotiable

AI agent interactions with MCP servers introduce event categories that existing logging frameworks were not designed to capture:
- Tool invocations mediated by an LLM (not a deterministic API client)
- Authorization decisions based on dynamic role evaluation
- Data access attributable to AI agents acting on behalf of human users

Standard application logs are insufficient. The schemas below define the minimum event structure required for regulatory compliance.

---

## Mandatory Audit Event Categories

### Event Type 1: Tool Invocation

Every MCP tool call must generate an audit event with the following fields:

```json
{
  "event_type": "tool_invocation",
  "schema_version": "1.0",
  "timestamp": "<ISO 8601 with millisecond precision>",
  "user_id": "<authenticated user identifier — REQUIRED, never null>",
  "agent_id": "<AI agent or session identifier>",
  "server_id": "<MCP server identifier from registry>",
  "tool_name": "<exact tool name as declared in schema>",
  "parameters_hash": "<sha256 of serialized parameters — NOT plaintext>",
  "duration_ms": 0,
  "result_status": "<success | error | rate_limited | unauthorized>",
  "result_size_bytes": 0,
  "data_classification": "<public | internal | confidential | restricted>",
  "purpose_code": "<GDPR/HIPAA processing purpose code>",
  "session_token_fingerprint": "<hash of session token — for correlation>",
  "source_ip": "<client IP>",
  "gateway_id": "<MCP gateway that processed the request>"
}
```

**Retention:** 1–7 years depending on regulation (see mapping below)

---

### Event Type 2: Authentication Events

```json
{
  "event_type": "authentication",
  "schema_version": "1.0",
  "timestamp": "<ISO 8601>",
  "user_id": "<user identifier or null if auth failure before identity established>",
  "client_id": "<MCP client/agent identifier>",
  "mtls_cert_fingerprint": "<client certificate fingerprint>",
  "auth_method": "<oauth2 | mtls | combined>",
  "success": true,
  "failure_reason": null,
  "source_ip": "<client IP>",
  "geolocation": "<derived from IP — country, region>",
  "device_trust_score": null,
  "token_issued_fingerprint": "<hash of issued session token>",
  "token_ttl_seconds": 0
}
```

**Retention:** 1 year minimum

---

### Event Type 3: Authorization Decisions

```json
{
  "event_type": "authorization_decision",
  "schema_version": "1.0",
  "timestamp": "<ISO 8601>",
  "user_id": "<user identifier>",
  "tool_requested": "<tool name>",
  "server_id": "<MCP server identifier>",
  "session_scope": ["<list of tools in authorized scope>"],
  "decision": "<allow | deny>",
  "denial_reason": null,
  "policy_version": "<version of policy applied>",
  "reason_code": "<RBAC_DENY | DENYLIST | SCOPE_EXCEEDED | RATE_LIMITED | other>",
  "user_role_at_decision_time": "<role>",
  "environment": "<development | staging | production>"
}
```

**Retention:** 1 year minimum

---

### Event Type 4: Configuration Changes

```json
{
  "event_type": "configuration_change",
  "schema_version": "1.0",
  "timestamp": "<ISO 8601>",
  "admin_id": "<identifier of administrator making the change>",
  "server_id": "<MCP server identifier>",
  "change_type": "<tool_added | tool_modified | tool_removed | auth_changed | schema_updated | version_upgraded | other>",
  "previous_config_hash": "<sha256 of config before change>",
  "new_config_hash": "<sha256 of config after change>",
  "approval_ticket": "<change management ticket reference — REQUIRED>",
  "deployment_pipeline_run_id": "<CI/CD run identifier>",
  "emergency_change": false,
  "retrospective_review_required": false,
  "retrospective_review_deadline": null
}
```

**Retention:** Permanent

---

### Event Type 5: Error and Exception Events

```json
{
  "event_type": "error_exception",
  "schema_version": "1.0",
  "timestamp": "<ISO 8601>",
  "error_type": "<auth_failure | schema_validation | rate_limit | backend_error | injection_pattern_detected | other>",
  "server_id": "<MCP server identifier>",
  "tool_name": "<tool name if applicable>",
  "user_id": "<user identifier if available>",
  "parameters_hash": "<sha256 of parameters if applicable>",
  "error_message": "<sanitized error message — no PII, no secrets>",
  "stack_trace_hash": "<hash of stack trace for correlation without exposing internals>",
  "recovery_action": "<none | rate_limited | circuit_breaker_triggered | session_terminated | server_suspended>"
}
```

**Retention:** 90 days minimum

---

### Event Type 6: Data Access Events

```json
{
  "event_type": "data_access",
  "schema_version": "1.0",
  "timestamp": "<ISO 8601>",
  "user_id": "<user identifier>",
  "data_classification": "<classification tier>",
  "record_count_accessed": 0,
  "record_types": ["<list of data types accessed>"],
  "purpose_code": "<legal basis / purpose code>",
  "downstream_system": "<system called by MCP tool>",
  "phi_record_ids": ["<patient record IDs if PHI — required for HIPAA breach notification>"],
  "data_subject_ids": ["<data subject IDs if personal data — required for GDPR SAR>"],
  "consent_reference": null,
  "dpia_reference": null
}
```

**Retention:** Per applicable regulation (see mappings below)

---

## Log Architecture Requirements

### Centralized Collection [REQUIRED]
All audit events must be forwarded to a centralized SIEM in real time. Supported platforms: Splunk, Datadog, Azure Sentinel, Elastic SIEM. Local storage on MCP server hosts is **insufficient** for compliance.

### Tamper-Evidence [REQUIRED]
Logs must be written to write-once, append-only storage with cryptographic chaining:
- Each log entry includes a hash of the previous entry
- Any deletion or modification is detectable
- Alternatively: logs signed with hardware-backed key before transmission
- The signing key must not be accessible to the MCP server process itself

### Structured Format [REQUIRED]
- All events in machine-readable JSON
- Consistent, versioned schema registered in organization's schema registry
- Schema version included in every event

### PII/Secret Redaction [REQUIRED]
- Sensitive parameter values (passwords, API tokens, PII fields) replaced with hash or redaction marker before logging
- Hash preserves correlation capability without storing plaintext
- Do NOT log: raw credentials, PHI in plaintext in parameters, PII in tool outputs

---

## Regulatory Retention Requirements

| Regulation | Minimum Retention | Relevant Event Types |
|---|---|---|
| HIPAA (164.312(b)) | 6 years from creation | All PHI-touching events (Tool Invocation, Data Access) |
| GDPR (Article 30) | Duration of processing + applicable SOL | Data Access events with data subject IDs |
| SOC 2 Type II | 1 year (audit period + 90 days buffer) | All event types |
| PCI-DSS (Req 10.7) | 12 months, 3 months immediately available | All events for CDE-touching servers |

---

## Regulatory Compliance Mapping

### GDPR (EU 2016/679)
- **Article 4(2):** Tool invocations processing personal data are data processing activities → must be in Record of Processing Activities (Article 30)
- **Article 15 (SAR):** Logs must be queryable by `data_subject_id` to respond to Subject Access Requests
- **Article 9:** DPIA required for MCP servers accessing special category data (health, financial, behavioral)
- **Article 33:** Breach notification within 72 hours — logs must support breach reconstruction

### HIPAA (45 CFR Parts 160 and 164)
- **164.312(b) Audit Controls:** PHI access logging satisfied by Tool Invocation + Data Access schema
- **164.312(a)(2)(i) Unique User ID:** Satisfied by `user_id` never null in all event types
- **PHI tagging:** `phi_record_ids` in Data Access events enables breach notification scope determination

### SOC 2 Type II
- **CC6.1 (Logical Access Controls):** RBAC framework (§7) + Authorization Decision logging
- **CC6.3 (Authorization Based on Job Responsibilities):** Role-based tool allowlists + decision reason codes
- **CC7.2 (Anomaly Monitoring):** Anomaly detection signals (see below) + SIEM integration
- **Log integrity:** Tamper-evidence architecture satisfies auditor requirement for log integrity controls

### PCI-DSS v4.0
- **Requirement 7:** Restrict access — satisfied by RBAC + Authorization Decision logging
- **Requirement 8:** Identify and authenticate — satisfied by Authentication Events + mTLS
- **Requirement 10.2.1:** Mandatory event types map directly to the 6 event categories above
- **Requirement 10.3 (Log protection):** Satisfied by tamper-evidence architecture
- **Scope:** All MCP servers that can reach cardholder data environment are in PCI scope

---

## Anomaly Detection Signals

The following alerts must be configured in your SIEM:

| Signal | Detection Logic | Severity |
|---|---|---|
| Tool flood | Invocation rate >3× 7-day rolling baseline per user/agent | HIGH |
| Schema anomaly | Tool invocations accessing data schemas outside user's historical access patterns | HIGH |
| Injection pattern | Parameter values containing SQL metacharacters, LLM instruction syntax, path traversal | HIGH |
| Role violation | Sequential read-then-write for user with read-only authorization | CRITICAL |
| Token reuse | Single session token from multiple source IPs within 5 minutes | CRITICAL |
| Unauthorized config change | Config change event without corresponding approval ticket | HIGH |

---

## References
- Paper §9.1–9.4
- HIPAA 45 CFR 164.312(b)
- GDPR Articles 4, 15, 30, 33
- SOC 2 CC6, CC7
- PCI-DSS v4.0 Requirements 7, 8, 10
