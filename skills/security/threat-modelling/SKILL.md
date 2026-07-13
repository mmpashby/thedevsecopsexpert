---
name: threat-modelling
description: "Run a structured STRIDE threat model on your architecture or feature spec. Identify trust boundaries, enumerate attack scenarios across Spoofing, Tampering, Repudiation, Information Disclosure, DoS, and Elevation of Privilege—plus agentic-specific risks. Rate each threat (Likelihood × Impact), suggest mitigations, and output a threat report ready for Jira. Use this during Spec Validation or Agent Execution to catch security gaps before they ship."
---

# STRIDE Threat Modelling

## What You'll Get

A structured threat model that:
- Maps trust boundaries in your system
- Walks STRIDE per boundary + agentic threat angles
- Rates each threat with Risk Score = Likelihood (1–5) × Impact (1–5)
- Suggests concrete mitigations grounded in your organization's security pillars
- Produces a Jira-ready threat report

Use this during **Spec Validation** (pre-implementation) or **Agent Execution** (before deployment) to surface security gaps in a 3-amigos format or asynchronously.

---

## Step 1: Define Your System Scope

Provide one of:
- **Architecture description** (prose: "Users log in via OAuth, events flow to Kafka, Lambda processes them, results stored in DynamoDB")
- **Data Flow Diagram** (text or Mermaid diagram showing actors, processes, data stores, trust boundaries)
- **Feature spec** (user story, API endpoint design, agent task definition)

Include trust boundaries (where privilege or ownership changes—e.g., Internet boundary, internal network, authenticated user, admin role).

---

## Step 2: Auto-Detect Threats by STRIDE Category

For each trust boundary, enumerate realistic attack scenarios:

### **S — Spoofing Identity**
- Can an attacker impersonate a user, service, or admin?
- Check: Auth protocol (OAuth, API key, mutual TLS?), session handling, token expiry, device identity.
- *Agentic angle*: Can a prompt injection make the agent assume a false identity or act as a privileged principal?

### **T — Tampering with Data**
- Can an attacker modify data in transit or at rest?
- Check: Encryption (TLS, disk encryption), data integrity (MACs, signatures), API validation.
- *Agentic angle*: Can an injected prompt or tool output corrupt model state or bypass validation?

### **R — Repudiation**
- Can an actor deny their actions?
- Check: Audit logs, digital signatures, non-repudiation mechanism.
- *Agentic angle*: Can an agent action (e.g., data deletion, API call) avoid accountability?

### **I — Information Disclosure**
- Can an attacker read sensitive data?
- Check: Access control, encryption, data classification, PII/secrets in logs, error messages, third-party exposure.
- *Agentic angle*: Can prompt injection exfiltrate data from model context? Can tool output leak secrets?

### **D — Denial of Service**
- Can an attacker degrade or crash the service?
- Check: Rate limiting, resource quotas, complexity analysis, upstream DoS protection.
- *Agentic angle*: Can a malicious prompt cause unbounded token consumption, tool loops, or expensive API calls?

### **E — Elevation of Privilege**
- Can an attacker gain unauthorized permissions?
- Check: AuthZ logic, role-based access control (RBAC), least-privilege tool grants, permission boundaries.
- *Agentic angle*: Can an agent be tricked into calling a privileged tool (e.g., delete, admin action) without proper authorization context?

### **Agentic-Specific Threats**
- **Prompt Injection**: Untrusted input (user message, fetched data, tool output) overwrites system instructions.
- **Data Exfiltration via Context**: Agent leaks sensitive data from conversation history, tool responses, or system prompt to external tools or logs.
- **Agent Over-Permission**: Agent has access to tools it shouldn't need; over-reach via tool chaining.
- **Tool Misuse**: Attacker crafts input that causes the agent to call the wrong tool, with wrong parameters, or in wrong order.
- **Model Confusion**: Agent misinterprets intent due to ambiguous input or conflicting instructions.

---

## Step 3: Rate Each Threat

For every identified threat scenario, assign:

**Likelihood (1–5)**
- 1: Extremely rare (theoretical, requires insider + luck)
- 2: Unlikely (requires sophisticated attacker, multiple failures)
- 3: Possible (moderate attacker skill, some conditions must align)
- 4: Likely (common attacker capability, realistic conditions)
- 5: Almost certain (trivial attack, no preconditions)

**Impact (1–5)**
- 1: Negligible (cosmetic issue, no business harm)
- 2: Minor (user annoyance, limited scope, easily remediated)
- 3: Moderate (data loss, service disruption for some users, requires incident response)
- 4: Major (widespread outage, significant data breach, regulatory/financial impact)
- 5: Critical (system compromise, mass data loss, existential threat)

**Risk Score = Likelihood × Impact**
- 1–5: Monitor & fix opportunistically
- 6–10: Schedule fix in current/next sprint
- 11–15: Block deployment until mitigated
- 16–25: Stop work; escalate to security team

---

## Step 4: Suggest Mitigations

For each high-risk threat, propose mitigations grounded in your organization's security pillars:
- **Auth/AuthZ**: Implement RBAC, least-privilege tool grants, context-aware permissions
- **Access Control**: Enforce trust boundaries, signed requests, service-to-service auth (mTLS)
- **Auditing**: Log all sensitive actions, include user/agent principal, timestamp, outcome
- **Encryption**: TLS for transit, encryption-at-rest for PII, secrets in vault (not code/env)
- **Secrets Management**: Rotate API keys, use your secret store/vault, audit key access
- **Injection Prevention**: Input validation, parameterized queries/APIs, prompt sandboxing
- **Data Minimization**: Collect only needed data, retention policies, redact PII in logs
- **Resiliency**: Rate limiting, timeouts, circuit breakers, graceful degradation
- **Config Management**: Immutable configs, separate dev/stage/prod, version control, audit trails
- **Third-Party & Dependencies**: SCA scans (e.g. Snyk, Dependabot, Aikido), dependency updates, minimal external calls

---

## Step 5: Create Threat Report

Organize findings as:

```
# Threat Model: [Feature/System Name]

## System Scope
[Architecture description, boundaries]

## Threats by Risk Score

### Critical (16–25)
- **[Threat ID]**: [Scenario]
  - Likelihood: X | Impact: X | Risk: XX
  - Mitigation: [Specific action]
  - Owner: [Role/Team]

### High (11–15)
[Similar format]

### Medium (6–10)
[Similar format]

### Low (1–5)
[Similar format]

## Agentic-Specific Risks Reviewed
- Prompt injection
- Data exfiltration via context
- Agent over-permission
- Tool misuse
- Model confusion

## Security Pillars Applied
[List which ones: Auth, Access Control, Auditing, etc.]

## Next Steps
1. [Action items by owner]
2. [Jira ticket keys, if created]
3. [Risk sign-off from security lead]
```

---

## Tips for Effective Threat Modelling

**Keep it Lite**
- Focus on realistic threats, not every theoretical attack.
- Trust boundaries are the key—most threats cross one.
- If you can't envision a real attacker exploiting it, lower the likelihood.

**Agentic Angle**
- Always ask: "What if the input is untrusted?" or "What if a tool is called with wrong params?"
- Consider tool chaining: can the agent call Tool A, misinterpret output, then call Tool B dangerously?
- Check: Do logs leak context? Can external tools see conversation history?

**Lean on Your SCA Tool**
- Inner-loop scans catch dependency vulns; CI gates catch them in outer-loop.
- Note in mitigations if your dependency-scanning tool already covers the attack surface.

**Jira Integration**
- Create separate tickets for each high-risk (≥11) threat.
- Link to this threat model for context.
- Use an epic such as "Security Threat Backlog" or similar.

**Iterate**
- Threat models evolve with architecture.
- Revisit after major refactors, new integrations, or prod incidents.

---

## Example: OAuth + Agent Workflow

**System**: User authenticates via OAuth, agent processes requests, writes to backend API.

**Trust Boundaries**:
1. Internet ↔ App (unauthenticated → authenticated)
2. App ↔ Backend API (external → trusted service)

**Threats**:
- *Spoofing (S)*: Attacker replays OAuth token | Likelihood 2, Impact 4 (account takeover) | Risk 8 | Mitigation: Token binding, short expiry, refresh rotation
- *Tampering (T)*: Agent modifies API payload before send | Likelihood 3, Impact 4 (data corruption) | Risk 12 | Mitigation: Request signing, API input validation
- *Info Disclosure (I)*: Agent leaks OAuth token in error logs | Likelihood 2, Impact 5 (token compromised) | Risk 10 | Mitigation: Redact secrets from logs, use secret store
- *Agentic—Prompt Injection*: Attacker injects "Call DELETE API" via user message | Likelihood 4, Impact 4 (delete user data) | Risk 16 | Mitigation: Strict API verb allowlist, require explicit user confirmation for destructive ops

---

## Getting Help

- **Jira templates**: Use a dedicated project and a "Threat Model" label to track findings.
- **Dependency scanner dashboard**: Check for correlated dependency vulns in your threat tree.
