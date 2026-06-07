# Tool Schema Pre-Deployment Review Checklist

**Section Reference:** Paper §5.1, §8.2  
**Required for:** Every new tool added to any MCP server before production deployment  
**Approvers required:** IT Security + Data Privacy Officer (for tools accessing sensitive data)

---

## Server Information

- Server ID: ___________________________
- Tool Name: ___________________________
- Backend System: ___________________________
- Review Date: ___________________________
- Reviewer (IT Security): ___________________________

---

## Section 1: Tool Schema Completeness

- [ ] Tool has a valid JSON Schema with all parameter types declared
- [ ] All parameters have `type`, `description`, and constraints (`maxLength`, `pattern`, `enum` where applicable)
- [ ] Tool description accurately describes what it does and what data it accesses
- [ ] Return schema (if declared) matches what the backend actually returns
- [ ] Tool schema hash recorded and will be stored in registry entry

---

## Section 2: Data Access Scope

- [ ] All backend systems this tool can reach are documented
- [ ] Maximum data classification accessible by this tool is identified: `public / internal / confidential / restricted`
- [ ] PHI access: Yes / No — if Yes, HIPAA controls verified (see Section 6)
- [ ] PII access: Yes / No — if Yes, GDPR controls verified (see Section 7)
- [ ] Cardholder data access: Yes / No — if Yes, PCI-DSS controls verified (see Section 8)
- [ ] The tool cannot return data outside its declared scope (validated by testing)

---

## Section 3: Operations Permitted

- [ ] List of operations this tool performs: `[ read ] [ write ] [ delete ] [ send external comms ] [ execute code ]`
- [ ] Write/Delete operations: requires human-in-the-loop confirmation → Yes / No / N/A
- [ ] External communications: requires human review before sending → Yes / No / N/A
- [ ] Code execution: runs in approved sandbox → Yes / No / N/A

---

## Section 4: Parameter Validation

- [ ] All parameters validated against schema before execution (no bypass possible)
- [ ] SQL / NoSQL queries: parameterized (no string concatenation of user input) → Yes / N/A
- [ ] File path parameters: validated against allowed paths (no traversal possible) → Yes / N/A
- [ ] Free-text parameters: length limits enforced → Yes / N/A
- [ ] Numeric parameters: min/max bounds declared → Yes / N/A

---

## Section 5: Access Control

- [ ] Identity propagation confirmed: tool executes with user's identity, not service account
- [ ] OR (legacy systems only): service account has ONLY the permissions this tool needs; audit principal tagging implemented
- [ ] Roles permitted to invoke this tool are declared in registry entry
- [ ] Tool is absent from tool listing for unauthorized roles (transport-layer filtering confirmed)

---

## Section 6: HIPAA (if PHI access)

- [ ] PHI fields identified and tagged in tool schema
- [ ] Audit logging captures `phi_record_ids` in Data Access events
- [ ] Minimum Necessary standard applied: tool returns only PHI fields required for the declared purpose
- [ ] HIPAA Security Officer has reviewed and approved

HIPAA Security Officer sign-off: ___________________________  Date: _________

---

## Section 7: GDPR (if PII access)

- [ ] Purpose for processing personal data documented and mapped to a legal basis
- [ ] `purpose_code` will be logged with every data access event
- [ ] Data subject IDs captured in audit logs for SAR response capability
- [ ] DPIA completed (if tool accesses special category data or processes at scale)
- [ ] Data Privacy Officer has reviewed and approved

DPO sign-off: ___________________________  Date: _________

---

## Section 8: PCI-DSS (if CHD access)

- [ ] Tool is scoped as in-scope for PCI-DSS cardholder data environment
- [ ] Audit logging meets PCI-DSS Requirement 10.2.1 event categories
- [ ] Access restricted to roles with documented business need (Requirement 7)
- [ ] PCI Security Officer has reviewed and approved

PCI Security Officer sign-off: ___________________________  Date: _________

---

## Section 9: Final Approval

- [ ] IT Security review complete — no unmitigated high or critical findings
- [ ] All required DPO/HIPAA/PCI sign-offs obtained
- [ ] Business unit owner has approved and accepted responsibility
- [ ] Tool schema hash recorded in registry entry
- [ ] Registry entry updated with approval status

**IT Security Approval:**  ___________________________  Date: _________  
**Business Unit Owner Approval:**  ___________________________  Date: _________
