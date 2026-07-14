# thedevsecopsexpert
Security/DevOps Tools, AI Skills, and much more.

## Skills

Reusable [Claude Code skills](https://docs.claude.com/en/docs/claude-code/skills) live under [`skills/`](skills/), organized by category. Each skill is a self-contained directory with its own `README.md` (usage + install instructions) and `SKILL.md` (the skill definition).

### Security

Skills for weaving security review into an agentic development workflow, from spec to post-deploy:

| Skill | Use it during | What it does |
|---|---|---|
| [security-spec-review](skills/security/security-spec-review) | Before development | Scans user stories, acceptance criteria, and specs for missing security requirements across 9 dimensions |
| [agentic-permissions-audit](skills/security/agentic-permissions-audit) | Spec validation / before agents run | Audits agent configs, MCP permissions, and system prompts for over-permissioning, data leakage, and injection risk |
| [threat-modelling](skills/security/threat-modelling) | Spec validation / pre-deploy | Runs a structured STRIDE threat model against an architecture, data-flow diagram, or feature spec |
| [post-deploy-security-checklist](skills/security/post-deploy-security-checklist) | Release time | Pre- and post-deploy checklist covering secrets, dependencies, auth, monitoring, and rollback readiness |

### Productivity

Skills for improving how agents and humans work together over time:

| Skill | Use it during | What it does |
|---|---|---|
| [agent-experience-interview](skills/productivity/agent-experience-interview) | Retro / continuous improvement | Runs a developer-experience-style interview where the AI agent is the interviewee, grounding answers in its recorded memory to surface friction, decisions, and lessons from past work |

### Installation

Install an individual skill via the [skills CLI](https://docs.claude.com/en/docs/claude-code/skills):

```bash
npx skills add https://github.com/mmpashby/thedevsecopsexpert.git --skill security/<skill-name>
```

Or copy it manually:

```bash
cp -r skills/security/<skill-name> ~/.claude/skills/<skill-name>
```
