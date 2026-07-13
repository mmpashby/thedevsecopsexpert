# Security Spec Review

Scan user stories, acceptance criteria, and feature specs for missing security requirements **before** development — before Jira tickets are filed, before code is written, before agents execute.

Catches gaps across nine dimensions: auth/authz, input validation, PII handling, secrets, rate limiting, audit logging, error handling, third-party integrations, and agentic-specific risks. Designed for the **Spec Validation** phase of an agentic development workflow.

## Usage

Invoke the skill and paste (or attach) the spec to review:

- A user story
- Acceptance criteria
- A feature spec or design document

The skill walks all nine dimensions and outputs prioritized gaps (**Critical / High / Medium**), each with a concrete, copy-pasteable acceptance criterion you can add to the spec or convert to a Jira ticket.

## Installation

Via skills CLI (HTTPS):

```bash
npx skills add https://github.com/mmpashby/thedevsecopsexpert.git --skill security/security-spec-review
```

Via skills CLI (SSH):

```bash
npx skills add git@github.com:mmpashby/thedevsecopsexpert.git --skill security/security-spec-review
```

Manual:

```bash
cp -r skills/security/security-spec-review ~/.claude/skills/security-spec-review
```
