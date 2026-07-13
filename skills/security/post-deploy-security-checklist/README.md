# Post-Deploy Security Checklist

A pre- and post-deploy security checklist for the **Human Review & Ship** and **Post Release** phases of an agentic development workflow. Verifies no secrets leaked, dependencies are safe, auth is intact, and monitoring is live.

Dependency-scanning CI catches dependency CVEs — this skill catches everything else (secrets, misconfiguration, auth regressions, missing monitoring) and confirms that smoke tests and rollback plans are ready.

## Usage

Invoke the skill and provide deployment context:

- Service name and repository
- What changed (PR/MR link or summary)
- Target environment (staging, production)
- Deployment method (rolling, blue/green, canary)
- Links to deployed service or relevant code/config

The skill walks pre-deploy items (secrets, dependencies, auth, input validation, CORS/CSP, logging, migrations, IAM) and post-deploy items (monitoring, smoke tests, rollback readiness, vulnerability re-scan), assigning **PASS / FAIL / NEEDS REVIEW** with evidence and concluding with a **GREEN / YELLOW / RED** risk rating.

## Installation

Via skills CLI (HTTPS):

```bash
npx skills add https://github.com/mmpashby/thedevsecopsexpert.git --skill security/post-deploy-security-checklist
```

Via skills CLI (SSH):

```bash
npx skills add git@github.com:mmpashby/thedevsecopsexpert.git --skill security/post-deploy-security-checklist
```

Manual:

```bash
cp -r skills/security/post-deploy-security-checklist ~/.claude/skills/post-deploy-security-checklist
```
