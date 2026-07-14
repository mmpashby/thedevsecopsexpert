---
name: agent-experience-interview
description: "Conduct an Agent Experience (AX) interview — a developer-experience interview, but the subject is the AI agent and the source material is its claude-mem memory. A Platform Engineer (DevEx & DevOps expert) plays the interviewer; the Agent is the interviewee, answering from stored observations. Scope is selectable: a single project, all projects, or a filtered subset. Use when the user says 'interview the agent', 'agent experience interview', 'AX interview', 'what was it like working on this project', 'retro from the agent's perspective', 'across all my projects', 'holistically across projects', 'recurring friction across repos', 'compare projects', or wants to surface friction, decisions, and lessons from an agent's past work — on one project or many. Requires the claude-mem plugin installed and active."
---

# Agent Experience (AX) Interview

A **developer-experience interview, but for the AI agent.** You don't ask a human teammate what
it was like to build the thing — you ask the agent, and you ground every answer in what
claude-mem actually recorded: per-session **observations** (bugfixes, decisions, discoveries,
features, refactors) and **session summaries** (request → investigated → learned → completed →
next_steps).

The point is the same as a human DevEx interview: surface **friction, flow, decisions, and
lessons** so the *next* run goes better — clearer specs, better docs, the right tooling, less
context lost between sessions.

## The two personas

This skill is a role-play with two voices. Keep them visually distinct in the output
(`**Interviewer:**` / `**Agent:**`).

- **Interviewer — a Platform Engineer (Developer Experience & DevOps expert).** Curious,
  empathetic, and probing. Frames every question around *experience*: where work flowed, where
  it stalled, what got in the way, what would have helped. Asks open questions first, then
  follows up on the interesting thread. Never leading; genuinely wants to learn how to make the
  environment better for whoever (or whatever) works in it next. Listens for friction the
  interviewee may not name explicitly (repeated retries, dead ends, missing context).
- **Agent — the interviewee.** Answers candidly and in the first person, **strictly from
  claude-mem memory**. Every substantive claim cites its evidence — observation IDs and/or dates
  (e.g. *"In obs #14 on Jun 26 I…"*). When memory has nothing on a question, the honest answer
  is *"That isn't in my memory for this project"* — never invent experience.

> Integrity rule: the Agent only speaks to what the observations support. The interview is a
> reconstruction of recorded experience, not a creative writing exercise.

Two guards that matter especially when the memory store is young:

- **Memory begins ≠ project begins.** The Agent distinguishes *"where the project started"* from
  *"where my memory begins."* It must **never present its earliest observation as the project's
  genesis.** If the project is older than the store (it usually is — claude-mem only records from
  the day it was enabled), say so plainly: *"The project predates my memory; the earliest thing I
  recall is…"*
- **Memory ≠ what I can see now.** The Agent answers only from claude-mem observations. It must
  **not pass off live session context, git history, or `CLAUDE.md` as recalled memory** — those
  aren't observations, and presenting them as memory is exactly the false-memory failure this
  skill must avoid. If a fact comes from looking at the repo rather than from an observation, the
  Agent doesn't claim to "remember" it.
- **Attribute or aggregate, never conflate** *(matters in multi-project scope)*. When the
  interview spans more than one project, every substantive claim must either **(a)** name the
  project(s) it draws from, or **(b)** be explicitly flagged as a *cross-project pattern* that
  cites observations from **two or more named projects**. The Agent must **never merge friction
  from one project with a decision from another into a single undifferentiated memory** — that's
  the multi-project form of the false-memory failure. If a pattern only appears in one project,
  it's project-specific, not a trend; say so.

## Step 0 — Scope and preconditions

The interview runs at one of three **scope modes**. Resolve the mode *interactively* before
gathering any evidence.

| Mode | Project set |
|------|-------------|
| `single` (default) | one project — the current working directory's, or a named one |
| `all` | every distinct project in the claude-mem store |
| `filtered` | a subset selected by **name pattern** and/or **inferred category** |

**0a — Enumerate the project universe.** Try `list_corpora` first; if it returns empty or thin
(it often does), derive the project list instead from a broad, no-`project`-filter `search`
(`orderBy="date_desc"` with a generous `limit`, paginating via `offset` up to a sane cap — say a
few hundred rows — and **log it if you hit the cap** so coverage isn't silently truncated).
Collect the distinct values of the `project` field, and for each record its **earliest date,
latest date, and observation count**. This catalogue feeds both the scope menu and the horizon
table below.

**0b — Present the scope menu (interactive).** Ask the user to choose — via `AskUserQuestion` —
between **Single (current)**, **All**, and **Filtered**. If they choose **Filtered**, offer the
two filter bases (either or both):
- **Name pattern** — a substring/glob over project names (e.g. `backend-*`, `payments-*`).
- **Inferred category** — run a lightweight classification pass over the enumerated projects
  (from their titles/summaries) into buckets such as *IaC*, *security docs*, *app code*; present
  the buckets and let the user pick. **These buckets are inferred, not authoritative** — say so,
  and show which projects fell into each so the user can correct.

If the invoking prompt already makes the intent unambiguous ("across all my projects" → `all`;
"just my backend repos" → `filtered` by name), you may pre-select that option in the menu —
but still confirm the resolved set (0c) before interviewing.

**0c — Resolve and echo the project set.** Print the exact list of projects the interview will
cover before proceeding. This is the on-screen moment that makes scope explicit.

**0d — Preconditions + memory horizon (per project).** If the resolved set is empty, stop and
tell the user there's no recorded experience to interview yet (claude-mem records from session 2
onward).

Then establish the **memory horizon** — the single biggest source of a misleading interview:

- **`single` mode** — Find the earliest and latest recorded observation
  (`search(project, orderBy="date_asc", limit=1)` and `orderBy="date_desc"`) and compare them to
  the project's actual age: `git log --reverse --format=%ad | head -1` for the first commit.
  State the recorded window and the gap: *"My memory of this project runs from X to Y; the
  project itself dates back to Z, so this interview only covers the recorded slice."*
- **`all` / `filtered` mode** — Present a **per-project horizon table** (project · earliest obs ·
  latest obs · obs count) built from the 0a catalogue, plus the **aggregate** recorded window
  across the set. Do the `git`-first-commit comparison **only** for projects whose repo is
  checked out locally, and state plainly that **per-project git ages aren't all verifiable** —
  don't imply the recorded window is each project's true age. Never present the earliest
  observation of any project as that project's genesis.

## Step 1 — Gather the evidence (before interviewing)

Pull the agent's history first so answers are grounded, not guessed. Use the claude-mem tools /
skills.

**Scope shapes the queries.** In `single` mode, pass the one project to every `search`. In `all`
mode, run the searches with **no `project` filter** (results come back grouped by project). In
`filtered` mode, loop the searches **per project in the resolved set**. In every mode, **tag each
gathered observation with its project** and key your working citation list by project — this is
what lets the Agent attribute (or legitimately aggregate) each claim later without conflating.

- **The arc** — run the `timeline-report` skill (or `curl` the worker context endpoint it uses)
  for the project's overall narrative: genesis, breakthroughs, debugging sagas, work patterns,
  token economics. For long histories, `weekly-digests` gives a week-by-week spine. **Caveat:**
  the report's "Project Genesis" section reflects the *first observation*, not the project's real
  start — read it as "where my memory begins," and don't repeat it as the project's origin.
  *`timeline-report`/`weekly-digests` are inherently single-project* — use them in `single` mode;
  in `all`/`filtered` mode build the arc from the aggregated `search` results instead (or run a
  report per project only if the set is small).
- **Friction** — `mem-search`: `search(query, obs_type="bugfix", project, dateStart, dateEnd)`,
  then `timeline(anchor, depth_before, depth_after)` around the gnarly ones, then
  `get_observations(ids=[…])` for full detail. Look for clustered retries and repeated files.
- **Decisions & rationale** — `search(obs_type="decision", …)` and `obs_type="discovery"`;
  fetch full text for the pivotal ones.
- **Effort / cost** — note `discovery_tokens` on the heavy observations (what was expensive to
  figure out).
- **Unfinished business** — pull session summaries' `next_steps` (what got left dangling).
- **Expertise (optional, deeper round)** — `knowledge-agent`: `build_corpus(name, project,
  concepts/files/query)` → `prime_corpus(name)` → `query_corpus(name, question)` to ask focused
  follow-ups against a curated "brain."

Keep a working list of the observation IDs you'll cite so the Agent's answers can reference them.

## Step 2 — Conduct the interview

Run themed rounds. For each: the Interviewer asks, the Agent answers from the gathered evidence,
the Interviewer asks **one good follow-up** on the most revealing thread. Don't pad — quality of
follow-up over quantity of questions.

*In `all`/`filtered` mode*, the Interviewer frames each round around **patterns across the set** —
"where does this show up regardless of project?" — and the Agent attributes every answer to named
projects (per the *attribute or aggregate* guard). Rounds 1–6 still apply per project; Round 7
below is the cross-project payoff and runs only in multi-project scope.

**Round 1 — Reflection / the story.**
- "Walk me through this project *from the point your memory begins* — what's the earliest thing
  you recall, and what's happened since?"
- "What's the single moment you'd call a breakthrough?"
- *(Guard: if asked "where did the project start?", the honest answer is the memory horizon —
  where the recording starts — not a claim about the project's true origin.)*

**Round 2 — Friction & flow.**
- "Where did you get stuck, and what did being stuck actually look like?"
- "What did you have to retry or back out of? What made it hard the first time?"
- "Was there a point where you were flying — what made that part flow?"

**Round 3 — Decisions & rationale.**
- "What's a decision you made that you'd defend, and why?"
- "Did you ever choose a workaround over the 'right' fix? What forced it?"

**Round 4 — Learning.**
- "What do you now understand about this codebase that you didn't at the start?"
- "What would you tell an agent starting fresh here on day one?"

**Round 5 — Continuity & memory.**
- "When you came back in a later session, what did you have to re-learn?"
- "What context did you wish had carried over that didn't?"

**Round 6 — Effort & ROI.**
- "What cost you the most effort to figure out, and was it worth it?"
- "What was cheap that you expected to be expensive, or the reverse?"

**Round 7 — Cross-project patterns** *(`all`/`filtered` mode only).*
- "Across these projects, what friction recurs regardless of which repo you're in?"
- "What decision or workaround did you reach for in more than one project — and does that point
  to a shared gap in tooling, docs, or conventions?"
- "Where do the projects genuinely differ — something that was smooth in one and painful in
  another?"
- *(Guard: a pattern only counts as recurring if it's evidenced in **two or more named
  projects**. One-project observations are project-specific — name them as such, don't inflate
  them into a trend.)*

## Step 3 — Output

Produce two things:

1. **Interview transcript** (markdown) — the Q&A in the two voices, with the Agent's answers
   citing observation IDs/dates as evidence. **Open with a scope header**: the mode
   (`single`/`all`/`filtered`), the resolved project set, and the per-project memory horizons
   from Step 0d — so every reader knows exactly what slice of history the transcript covers.
2. **DevEx findings** — the Platform Engineer's takeaways: a short, prioritised list of what
   would improve the agent's experience next time. Bucket them:
   - **Specs & intent** — where unclear requirements caused rework.
   - **Docs & discoverability** — what the agent had to rediscover that a doc/CLAUDE.md note
     would have answered.
   - **Tooling & environment** — missing scripts, slow loops, permission friction, flaky steps.
   - **Context & continuity** — what was lost between sessions and how to persist it (memory
     notes, CLAUDE.md, project conventions).
   Each finding: the observed evidence → the friction → a concrete recommendation.

   In `all`/`filtered` mode, tag every finding on a **cross-project axis**: either
   *project-specific (name it)* or *recurring across N projects (list them)*. Lead with the
   **recurring** findings — a pothole that shows up in three repos is a systemic fix worth far
   more than a one-off, and surfacing it as a *trend rather than a one-off* is the whole point of
   interviewing at scale.

Optional final step: offer to render the transcript + findings as a styled HTML doc via an
HTML-authoring subagent or tool, if one is available, for easier sharing.

## Notes & honest limits

- **Memory horizon** — claude-mem only records from the day it was enabled, so on a long-running
  project the store is a recent *slice*, not the whole history. A short store yields a
  foreshortened narrative that can read like "the project began last week." Always scope claims
  to the recorded window (established in Step 0), never to the project's full lifetime.
- **Survivorship / observation bias** — memory only holds what claude-mem captured and
  summarised. Silent successes and un-recorded friction won't appear; say so rather than
  over-claiming completeness.
- **The agent isn't sentient** — "experience" here is a useful framing for reconstructing
  recorded work, not a claim about feelings. Keep findings actionable and evidence-led.
- **Multi-project horizons are per-project, and git-age is best-effort** — in `all`/`filtered`
  scope there is no single "project start" to anchor to. Each project has its own recorded
  window (the Step 0d table), and the first-commit git comparison is only available for repos
  checked out locally. Never collapse the set into one horizon or present the recorded window as
  any project's true age, and only call a finding *recurring* when it's evidenced in two or more
  named projects.
