# Agentic Permissions Audit

Audit agent configurations, MCP server definitions, and system prompts for over-permissioning, data leakage risks, prompt-injection vectors, and missing guardrails.

Designed for the **Spec Validation** and **Agent Execution** phases of an agentic development workflow — surface critical permission gaps before agents reach production.

## Usage

Invoke the skill and paste (or upload) any of:

- Agent configuration files
- MCP server definitions (JSON / YAML)
- System prompts
- Tool definitions
- `CLAUDE.md` / `SKILL.md` files
- Architecture descriptions of an agentic setup

The skill walks seven audit dimensions (least privilege, data boundaries, tool scope, injection resistance, destructive guardrails, context leakage, audit trail) and produces a per-finding report plus a permissions scorecard.

## Installation

Via skills CLI (HTTPS):

```bash
npx skills add https://github.com/mmpashby/thedevsecopsexpert.git --skill security/agentic-permissions-audit
```

Via skills CLI (SSH):

```bash
npx skills add git@github.com:mmpashby/thedevsecopsexpert.git --skill security/agentic-permissions-audit
```

Manual:

```bash
cp -r skills/security/agentic-permissions-audit ~/.claude/skills/agentic-permissions-audit
```
