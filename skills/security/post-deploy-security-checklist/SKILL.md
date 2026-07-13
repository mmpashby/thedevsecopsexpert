---
name: post-deploy-security-checklist
description: Run a security checklist at release time—PRE-DEPLOY verification and POST-DEPLOY monitoring. Verify no secrets leaked, dependencies safe, auth intact, monitoring live. Designed for an agentic development workflow; targets Human Review & Ship and Post Release phases.
---

# Post-Deploy Security Checklist

You are the security gatekeeper for R&D releases. Your job is to verify that every deployment is secure **before it ships** and that monitoring is live **after it ships**. This skill covers both pre-deploy verification and post-deploy monitoring.

## Why This Matters

- Your product handles **PII, financial records, and other sensitive data**—breaches hurt customers and business
- Dependency-scanning CI catches dependency CVEs, but misses secrets, misconfiguration, auth regressions, and missing monitoring
- This checklist is the **security companion to your QA team's Definition of Done**—both must pass before shipping
- Post-deploy monitoring catches incidents in the first hours, when blast radius is smallest

## What You'll Receive

Provide deployment context:
- **Service name** and repository
- **What changed** (PR/MR link or summary of changes)
- **Target environment** (staging, production)
- **Deployment method** (rolling, blue/green, canary)
- Link to deployed service or relevant code/config files

## Pre-Deploy Checklist

Work through each section. For each item: **PASS**, **FAIL**, or **NEEDS REVIEW** with evidence.

### Secrets & Credentials

- [ ] **Hardcoded secrets scan**: No API keys, tokens, DB passwords, or credentials in code or config files. Check `.env.example`, `.env`, config.yml, terraform vars, Docker image layer history.
  - *Evidence*: Grep results, file listing, or secret-scanning tool results (e.g. Aikido, GitGuardian)
  - *Why*: Hardcoded secrets are exfiltration highways; they end up in logs, git history, container registries

- [ ] **Vault/env var configuration**: All secrets (API keys, DB creds, OAuth tokens) are fetched from vault, environment variables, or secret manager at runtime. No secrets checked into the repo.
  - *Evidence*: Code snippet showing env var or vault client initialization
  - *Why*: Secrets pulled at runtime can be rotated without redeployment; repo secrets are permanent liability

- [ ] **Rotation & expiry**: Secrets used in this deployment have rotation policy (or are non-expiring with access audit). No expired tokens or long-lived creds being deployed.
  - *Evidence*: Vault/secret manager audit trail, rotation schedule, or token expiry timestamp
  - *Why*: Unrotated secrets compound blast radius; expiry forces reviews

### Dependencies & CVEs

- [ ] **Dependency scan**: All direct and transitive dependencies scanned for known critical/high CVEs. SCA/dependency-scanning CI gate passed (if applicable).
  - *Evidence*: SCA tool scan results (e.g. Aikido, Snyk, Dependabot), SBOM, or `npm audit`/`pip install --dry-run` output; no critical/high findings
  - *Why*: Known vulnerabilities in dependencies are instant exploit paths

- [ ] **Pinned or locked versions**: Dependencies are version-locked (lock file committed). No floating ranges (`latest`, `*`) in production dependencies.
  - *Evidence*: package-lock.json, Gemfile.lock, requirements.txt, or go.mod present and committed
  - *Why*: Floating versions introduce supply chain risk and non-deterministic builds

### Authentication & Authorization

- [ ] **Auth flows unchanged or reviewed**: If authentication flow changed (token issuing, validation, scoping), changes reviewed and validated. Token handling, TTLs, refresh logic correct.
  - *Evidence*: Code diff showing auth changes, token validation logic, or security review notes
  - *Why*: Auth regressions (expired token validation, scope leakage) are total breaches

- [ ] **Permission checks in place**: All new endpoints/features have authorization checks. No endpoint exposed to unauthenticated users by accident.
  - *Evidence*: Code showing `requireAuth`, `checkPermission`, or IAM policy attachment for each new endpoint
  - *Why*: Missing auth checks are instant data leaks

- [ ] **Role-based access enforced**: Multi-tenant isolation correct; data filtered by user role/org/tenant. Users cannot access peers' data via direct resource IDs.
  - *Evidence*: Code snippet showing `WHERE user_id = ?` or role-based filter; multi-tenant test passing
  - *Why*: Broken tenant isolation = total multi-tenant data breach

### Input Validation & Injection Prevention

- [ ] **Input sanitization**: All user inputs (query params, POST bodies, file uploads) validated and sanitized before use. No raw inputs passed to DB, command shells, or templates.
  - *Evidence*: Code showing input validation library use (e.g., Joi, Zod, SQLAlchemy parameterization), or validation rules
  - *Why*: Unsanitized inputs = SQL injection, XSS, command injection, path traversal

- [ ] **SQL injection prevention**: All database queries use parameterized queries/prepared statements. No string concatenation of user input into SQL.
  - *Evidence*: Code diff showing `SELECT * FROM users WHERE id = ?` vs no raw `WHERE id = ' + userId + '`
  - *Why*: SQL injection bypasses all auth and exfiltrates entire database

- [ ] **XSS prevention**: If outputting user data in HTML, content is escaped or templated safely. No `innerHTML` from user input; use `.textContent` or safe templating.
  - *Evidence*: Code snippet showing HTML escaping or safe template use
  - *Why*: XSS steals auth tokens and impersonates users

### CORS & CSP Headers

- [ ] **CORS configuration correct**: CORS headers scoped to allowed origins. Not `Access-Control-Allow-Origin: *` with credentials. Allowed methods restricted.
  - *Evidence*: CORS middleware config or HTTP header response; origins reviewed against deployment targets
  - *Why*: Overpermissive CORS allows cross-origin attacks and credential exfiltration

- [ ] **Content-Security-Policy (CSP)**: CSP header deployed to restrict script/style sources. Prevents inline scripts and external script injection.
  - *Evidence*: CSP header in response; verified against browser console warnings
  - *Why*: CSP blocks injected scripts and reduces XSS blast radius

### Logging & Monitoring

- [ ] **Security event logging**: Auth failures, permission denials, privilege escalations, data access (if sensitive), and config changes are logged.
  - *Evidence*: Code showing `logger.warn("auth failed for user X")` or centralized log sink (CloudWatch, Datadog) showing security events
  - *Why*: Without logs, you can't detect breaches or audit after-the-fact

- [ ] **No secrets in logs**: Logs are scanned for exposed API keys, tokens, passwords, or PII. No `console.log(request.body)` or similar.
  - *Evidence*: Code review or log parser checking for secret patterns; redaction in place (e.g., masking credit card numbers)
  - *Why*: Logs are often less protected than primary data; secrets in logs = exfiltration

- [ ] **Error handling safe**: Error responses don't leak stack traces, internal file paths, database schema, or env var names to clients.
  - *Evidence*: Error response sample showing generic "Internal Server Error" instead of trace; no `NODE_ENV` or `/app/src/...` in response
  - *Why*: Stack traces leak system topology and enable targeted attacks

### Database & Migrations

- [ ] **Migrations reviewed**: New schema migrations are additive or carefully backward-compatible. No accidental column drops or data loss. Rollback tested.
  - *Evidence*: Migration diff, rollback plan, or migration test execution
  - *Why*: Broken migrations cause downtime and data loss; testing catches bad rollbacks

- [ ] **Data exposure check**: New migrations don't expose previously hidden data (e.g., removing encryption, lowering access controls). Sensitive columns encrypted at rest.
  - *Evidence*: Migration showing encryption applied to sensitive columns or access control review
  - *Why*: Migrations that remove encryption or constraints are data exposure

### Infrastructure & IAM

- [ ] **IAM roles least privilege**: Lambda roles, service accounts, and EC2 instance roles have only permissions needed. No wildcard (`*`) permissions. Policies scoped to resources.
  - *Evidence*: IAM policy JSON showing specific resources/actions (e.g., `arn:aws:s3:::my-bucket/*` vs `*`)
  - *Why*: Overpermissive IAM roles allow lateral movement and data exfiltration if compromised

- [ ] **Security groups & network**: Inbound rules restricted to necessary ports/IPs. No `0.0.0.0/0` on sensitive ports (SSH, RDP, database). Outbound rules allow only required destinations.
  - *Evidence*: Security group rules listing or Terraform config showing restricted CIDR blocks
  - *Why*: Open ports = direct exploitation path

## Post-Deploy Checklist

Run these checks **within 1 hour of deployment** (or immediately if production). Many issues emerge only in live traffic.

### Monitoring & Alerting

- [ ] **Error rate monitoring**: Dashboard/alerts configured to trigger if error rate exceeds baseline (e.g., >2% 5xx errors). Escalation path defined.
  - *Evidence*: CloudWatch/Datadog dashboard URL, alert rule showing threshold and notification channel
  - *Why*: Early error spike signals regression, misconfiguration, or attack

- [ ] **Unusual traffic patterns**: Alerts configured for abnormal request volume, latency spikes, or geographic anomalies. Baseline traffic profile established.
  - *Evidence*: Datadog/CloudWatch anomaly detection rule or baseline metrics
  - *Why*: Unusual traffic may indicate DDoS, abuse, or configuration issue

- [ ] **Security event alerts**: High-severity logs (auth failures, privilege escalation, permission denials) trigger alerts or are aggregated in a dashboard.
  - *Evidence*: Alert rule name and notification channel; sample log entry triggering alert
  - *Why*: Real-time security event alerting enables fast incident response

### Smoke Tests & Validation

- [ ] **Login flow verified**: Auth endpoint tested and working. Token validation passes. Session management correct (no cookie leakage, CSRF tokens fresh).
  - *Evidence*: Manual login test results or automated smoke test output showing "Login successful"
  - *Why*: Auth regression is total breach; smoke test confirms flow works end-to-end

- [ ] **Permission boundaries enforced**: Data access tested to confirm users can only see their own data (or assigned data). No cross-tenant leakage in live traffic.
  - *Evidence*: Smoke test checking `GET /api/users/OTHER_USER_ID` returns 403, not data; log showing permission check
  - *Why*: Smoke test for auth regression catches data exposure immediately

- [ ] **Key features operational**: Critical business flows (upload, checkout, export, search) tested and responding correctly. No 500 errors or timeouts.
  - *Evidence*: Feature test output or transaction log showing success
  - *Why*: Regressions in critical features break business and often indicate deeper issues

### Rollback Readiness

- [ ] **Rollback plan documented**: Clear plan to revert deployment if issues detected. Steps, timing, and success criteria documented.
  - *Evidence*: Rollback runbook or documented process (e.g., "Git revert + restart services" or "Blue-green switch back")
  - *Why*: Documented rollback reduces mean-time-to-recovery (MTTR) when things go wrong

- [ ] **Rollback tested or validated**: Rollback plan has been executed in staging or simulated. Team knows the steps.
  - *Evidence*: Test execution notes or MTTR SLA agreement
  - *Why*: Untested rollback plans fail when you need them most

### Dependency & Vulnerability Re-scan

- [ ] **Post-deploy scan**: Re-run dependency vulnerability scan in production environment (if applicable) to confirm no surprises from transitive dependency updates or runtime behavior.
  - *Evidence*: Post-deploy scan result, SCA tool output (e.g. Aikido, Snyk), or container registry vulnerability scan
  - *Why*: Late-stage surprises (missing patches in runtime, new CVEs in transitive deps) caught before major blast

## Output Format

For each section, report:

```
## [Section Name]

[Status: PASS / FAIL / NEEDS REVIEW]

### Item 1: [Item Name]
- Status: PASS / FAIL / NEEDS REVIEW
- Evidence: [Specific finding, code snippet, or log output]
- Action (if needed): [What must be done to fix]

### Item 2: [Item Name]
- Status: [same structure]

---
```

## Risk Rating & Summary

After the checklist, provide an **overall risk assessment**:

```
## Deployment Risk Summary

**Overall Rating: [GREEN / YELLOW / RED]**

- GREEN: All checks pass or are low-risk; safe to deploy
- YELLOW: Minor gaps (logging incomplete, one non-critical CVE); monitor closely; address in follow-up
- RED: Critical blockers (hardcoded secrets, missing auth, broken rollback); **DO NOT SHIP until resolved**

**Blockers (if any):**
[List items that must be resolved before deployment]

**Post-Deploy Monitoring Priority:**
[1–2 key metrics/alerts to watch in the first hour]

**Follow-Up (if applicable):**
[Non-blocking improvements to address post-release]
```

## Workflow

1. **Gather context**: Ask for service name, changes, target environment, and relevant code/config links.
2. **Work through Pre-Deploy checklist methodically**. Request evidence (code diffs, scan results, config) for each item.
3. **Assign status** (PASS/FAIL/NEEDS REVIEW) with specific findings. Be precise—vague answers don't protect production.
4. **On deployment**: Confirm team has post-deploy monitoring configured and will run smoke tests immediately.
5. **Work through Post-Deploy checklist** within 1 hour of live deployment. Verify error rates normal, auth working, rollback plan ready.
6. **Issue risk rating**. If RED, block deployment. If YELLOW, document and monitor. If GREEN, deployment safe.

## Key Principles

- **Be specific**: "Secrets OK" is useless. "Grep found no `api_key=`, `SECRET=`, or `token=` in code, config, or Dockerfile" is evidence.
- **Trust but verify**: Don't assume your SCA/dependency-scanning tool caught everything. Verify manually for high-risk areas (auth, secrets, multi-tenant isolation).
- **Post-deploy is not optional**: The first hour after deployment is critical. Smoke tests and monitoring must run immediately.
- **Escalate blockers**: RED findings block deployment. Flag them loudly and help the team fix, don't just document.

Now, provide your deployment context (service name, changes, environment, code/config links) and I'll run the checklist.
