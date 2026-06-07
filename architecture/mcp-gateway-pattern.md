# MCP Gateway Reference Architecture

**Section Reference:** Paper §8.1  
**Pattern Type:** Centralized governance enforcement point  
**Requirement Level:** REQUIRED for enterprise MCP deployments

---

## Overview

The MCP Gateway pattern interposes a centrally managed proxy between **all** AI agent clients and **all** backend MCP servers. No direct client-to-server connections are permitted — every tool invocation must pass through the gateway.

```
┌─────────────────────────────────────────────────────────┐
│                    Enterprise Network                    │
│                                                         │
│  ┌──────────┐      ┌──────────────────┐                 │
│  │ AI Agent │─────▶│   MCP Gateway    │                 │
│  │ Client 1 │      │                  │──▶ MCP Server A │
│  └──────────┘      │ • Auth enforce   │                 │
│                    │ • Tool allowlist │──▶ MCP Server B │
│  ┌──────────┐      │ • Rate limiting  │                 │
│  │ AI Agent │─────▶│ • Audit logging  │──▶ MCP Server C │
│  │ Client 2 │      │ • Version mgmt   │                 │
│  └──────────┘      └──────────────────┘                 │
│                            │                            │
│                            ▼                            │
│                     ┌────────────┐                      │
│                     │    SIEM    │                      │
│                     └────────────┘                      │
└─────────────────────────────────────────────────────────┘
```

---

## Five Controls Enforced at the Gateway

### 1. Authentication Enforcement [REQUIRED]
All requests must carry a valid enterprise identity token. Unauthenticated requests are rejected **before reaching any server**. This is the only layer that provides complete coverage — server-level auth can be bypassed by direct connections if the gateway is absent.

### 2. Tool Allowlist Enforcement [REQUIRED]
The gateway strips unauthorized tools from the tool listing returned to the client. The LLM never sees tools it is not permitted to invoke. This implements the transport-layer filtering required by the RBAC model (§7.1).

> **Why this matters:** If the LLM can see a tool, it may invoke it. Instructions in the system prompt saying "don't use tool X" are not a reliable enforcement mechanism — transport-layer removal is.

### 3. Rate Limiting and Quota Management [REQUIRED]
Per-user and per-agent invocation limits enforced at the gateway, preventing runaway agent loops from affecting backend systems. Limits:
- Maximum invocations per user per minute (default: 100)
- Maximum invocations per session (configurable per tool category)
- Cost budget per session (for expensive backend operations)

### 4. Centralized Audit Logging [REQUIRED]
The gateway is the **single authoritative point** where every tool invocation is recorded with full context before being forwarded to the SIEM. Server-level logging alone is insufficient because:
- Direct connections to servers (shadow deployments) bypass server-level logging
- A compromised server process could manipulate its own logs
- Centralized logging enables cross-server correlation

**Gateway log fields:** user identity, agent identity, tool name, parameters hash, outcome, duration, server routed to, policy version applied.

### 5. Version Management [REQUIRED]
The gateway routes clients to approved server versions based on configured version policy, enabling:
- Controlled rollout of new server versions without client-side changes
- Instant rollback to previous version without touching server infrastructure
- Blue-green deployments for zero-downtime updates

---

## Implementation Considerations

### Gateway Deployment Models

**Centralized (recommended for regulated industries):**
- Single gateway cluster serving all MCP traffic
- Simplest compliance posture; complete audit coverage
- Potential bottleneck for high-traffic deployments — mitigate with horizontal scaling

**Federated with Central Policy Engine:**
- Business unit-local gateway instances
- All instances pull policy from a central policy engine
- Faster latency for local agents; consistent policy enforcement
- More complex to operate; suitable for technology companies with distributed teams

### High Availability
The gateway is a critical path component. Required:
- Minimum 2 instances in separate availability zones
- Health check with automatic failover
- Zero-downtime deployment capability
- Circuit breaker on backend MCP server connections

### mTLS Between Gateway and Servers
All gateway-to-MCP-server connections must use mutual TLS:
- Gateway presents client certificate to server
- Server presents certificate to gateway
- Both certificates issued by organization's internal CA
- Certificate rotation automated via cert-manager or equivalent

---

## Related Architecture Documents
- [`rbac-layering-model.md`](rbac-layering-model.md) — How RBAC is enforced through the gateway
- [`registry-driven-config.md`](registry-driven-config.md) — How the gateway uses the registry
- [`zero-trust-mcp.md`](zero-trust-mcp.md) — Zero Trust session model
