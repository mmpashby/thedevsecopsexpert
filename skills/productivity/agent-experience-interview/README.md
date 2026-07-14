# Agent Experience (AX) Interview

A developer-experience interview, but the subject is the **AI agent** and the source material is
its **claude-mem memory**. A Platform Engineer (DevEx & DevOps expert) plays the interviewer; the
Agent is the interviewee, answering from recorded observations and citing them as evidence.

The goal is the same as a human DevEx interview: surface **friction, flow, decisions, and
lessons** from past work on a project so the next run goes better — clearer specs, better docs,
the right tooling, less context lost between sessions.

Scope is selectable: interview the agent about **one project**, **all projects** in the
claude-mem store, or a **filtered subset** (by name pattern and/or inferred category). The single
project is the default; the cross-project modes surface friction and decisions that recur across
repos as a *trend* rather than a one-off.

## Prerequisites

- The **claude-mem** plugin installed and active, with its worker running.
- At least one project in scope must have recorded observations (claude-mem records from session
  2 onward). With no history there is nothing to interview — in `all`/`filtered` mode the resolved
  project set must be non-empty.

## Usage

Invoke the skill (e.g. "interview the agent about this project", "run an AX interview across all
my projects", "interview the agent on my backend-* repos"). It will:

1. **Scope** — enumerate the projects claude-mem knows about, then offer an interactive menu:
   **Single** (current, the default), **All**, or **Filtered** (by name pattern and/or inferred
   category). It echoes the resolved project set and each project's memory horizon before
   interviewing. An optional date window still applies as an overlay.
2. **Gather evidence** — pull the agent's history with claude-mem: `timeline-report` for the
   arc (single-project), `mem-search` (`search` → `timeline` → `get_observations`) for friction
   (`bugfix`) and decisions (`decision`), session `next_steps` for unfinished work, and optionally
   a `knowledge-agent` corpus for a deeper expertise round. In multi-project scope every
   observation is tagged with its project so answers attribute, never conflate.
3. **Interview** — themed rounds (Reflection, Friction & flow, Decisions, Learning, Continuity,
   Effort/ROI) in two voices, every Agent answer grounded in cited observations. Multi-project
   scope adds a **Cross-project patterns** round.
4. **Output** — a markdown interview transcript (with a scope + horizon header) plus a prioritised
   **DevEx findings** list (specs, docs, tooling, continuity); in multi-project scope each finding
   is tagged *project-specific* or *recurring across N projects*. Optionally rendered as a styled
   HTML doc via an HTML-authoring subagent or tool, if one is available.

## Related

- See [`SKILL.md`](./SKILL.md) for the full procedure and personas.

## Installation

Via skills CLI (HTTPS):

```bash
npx skills add https://github.com/mmpashby/thedevsecopsexpert.git --skill productivity/agent-experience-interview
```

Via skills CLI (SSH):

```bash
npx skills add git@github.com:mmpashby/thedevsecopsexpert.git --skill productivity/agent-experience-interview
```

Manual:

```bash
cp -r skills/productivity/agent-experience-interview ~/.claude/skills/agent-experience-interview
```
