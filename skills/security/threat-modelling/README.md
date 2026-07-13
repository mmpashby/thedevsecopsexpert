# STRIDE Threat Modelling

Run a structured STRIDE threat model on an architecture, data-flow diagram, or feature spec. Identifies trust boundaries, enumerates threats across **S**poofing, **T**ampering, **R**epudiation, **I**nformation Disclosure, **D**enial of Service, and **E**levation of Privilege — plus agentic-specific risks (prompt injection, tool misuse, context exfiltration).

Each threat is rated `Likelihood × Impact` and paired with a concrete mitigation. Output is a Jira-ready threat report.

## Usage

Invoke the skill and provide one of:

- An architecture description (prose)
- A data-flow diagram (text or Mermaid)
- A feature spec, user story, or agent task definition

Include trust boundaries (where ownership or privilege changes — internet edge, internal services, authenticated user, admin role).

Use it during **Spec Validation** (pre-implementation) or **Agent Execution** (pre-deploy) to surface security gaps before they ship.

## Installation

Via skills CLI (HTTPS):

```bash
npx skills add https://github.com/mmpashby/thedevsecopsexpert.git --skill security/threat-modelling
```

Via skills CLI (SSH):

```bash
npx skills add git@github.com:mmpashby/thedevsecopsexpert.git --skill security/threat-modelling
```

Manual:

```bash
cp -r skills/security/threat-modelling ~/.claude/skills/threat-modelling
```
