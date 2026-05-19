# Agent OS — process framework for coding agents

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Last commit](https://img.shields.io/github/last-commit/PiloTracer/.ai)
![Stars](https://img.shields.io/github/stars/PiloTracer/.ai?style=social)

> Your AI coding agent forgets context between sessions, contradicts past decisions, claims tests pass when they fail, and starts coding before the plan exists. **Agent OS fixes that** with a small set of repeatable **skills**, binding **standards**, and a tiny **project-memory** folder.

**Works with:** Cursor, Claude Code, Codex, opencode, and any agent that reads project files.

```bash
bash .ai/templates/bootstrap.sh    # 30-second setup, any repo
```

Then in chat:

```text
@session-control start             # bookend the day
@plan-foundation greenfield        # blueprint → master plan → milestones → code
@session-control close             # update HANDOFF · draft commit message
```

**Lost?** → [`START_HERE.md`](START_HERE.md) · **All commands** → [Skills at a glance](#skills-at-a-glance) · **First time here?** → [How this compares](#how-this-compares)

---

## What you get

- **Skills** — `@session-control`, `@code-implementation`, `@plan-foundation`, … run the playbook (11 in total).
- **Standards** — binding contracts (CONVENTIONS, FEATURE_STANDARD, observability, security) keep agent output honest.
- **`.work/`** — the project's memory: plans, SPECs, ADRs, `HANDOFF.md`, `NEXT.md`. Survives session boundaries.
- **Gates** — `plan-master-ready`, `implementation-ready`, milestone verify; skip a step and the agent **stops** with a redirect.

Result: less re-prompting, fewer "where were we?" threads, a loop you can run **start → ship → hand off** every session.

---

## How this compares

| Tool | What it does | Where Agent OS adds value |
|------|--------------|---------------------------|
| `.cursorrules` / `AGENTS.md` | Static rules for one tool | Adds **gated workflow + memory + standards** on top |
| Plain prompts | Per-task instructions | Adds **repeatable skills** with prerequisite gates |
| Linear / Jira | Human-only planning | Plans live **next to code** as Markdown the agent reads |
| GitHub Copilot / inline LLM | Single-turn completion | Cross-session continuity via `.work/HANDOFF.md` + `NEXT.md` |
| Custom system prompts | Per-developer drift | One repo-wide, tool-agnostic contract checked into git |

---

## Bird's-eye — how to use Agent OS

Agent OS is a **gated pipeline**: each stage unlocks the next. Skills enforce the gates; if you skip a step, the agent should **stop** and tell you what to run instead ([`skills/SKILL_DEPENDENCIES.md`](skills/SKILL_DEPENDENCIES.md)).

### Readiness states (do not skip or confuse)

| State | You get there with | What it unlocks |
|-------|-------------------|-----------------|
| *(scaffold)* | `@project-bootstrap init` | Foundation planning, session files |
| **foundation-complete** | `@plan-foundation greenfield` (P0–P6 done) | `@plan-foundation certify` |
| **plan-master-ready** | `@plan-foundation certify plan-master-ready` | `@plan-master greenfield` / `continue` / `revise` |
| **implementation-ready** | `@plan-master status` (master plan **Approved**) | `@code-implementation start` / `continue` (broad coding) |

**plan-master-ready ≠ ready to code everything.** It means the blueprint is solid enough for a master roadmap. Broad implementation needs **implementation-ready** (or an explicit milestone waiver in `.work/context/HANDOFF.md`).

### Full flow (once per project → every day → per milestone)

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│  ONCE PER PROJECT — planning gates (no application code yet)                 │
└─────────────────────────────────────────────────────────────────────────────┘

  @project-bootstrap init
        │
        │  Creates: .cursorrules · DOCS_TECH_STACK.md · .work/ skeleton
        │  (scaffold only — fill REPLACE: tokens; no planning gates)
        ▼
  @plan-foundation greenfield
        │
        │  Produces: foundation docs 01–04 · ADRs · SPECs · registries
        │  State: foundation-complete
        ▼
  @plan-foundation certify plan-master-ready
        │
        │  Deep consistency check (+ plan-master integrity on foundation)
        │  State: plan-master-ready  (recorded in HANDOFF)
        ▼
  @plan-master greenfield  │  @plan-master continue
        │
        │  Produces: .work/plans/full/*-full-plan.md  (Draft → Approved)
        ▼
  @plan-master status
        │
        │  State: implementation-ready: yes  (only this skill scores it)
        ▼

┌─────────────────────────────────────────────────────────────────────────────┐
│  EVERY SESSION — bookends (planning or coding)                               │
└─────────────────────────────────────────────────────────────────────────────┘

  @session-control start          ← load HANDOFF · NEXT · UNKNOWNS · rules
        │
        │  … your work (planning skills above, or implementation below) …
        ▼
  @session-control close          ← refresh HANDOFF + NEXT; draft commit message
  @session-control close commit   ← optional: stage + commit
  @session-control close commit push

┌─────────────────────────────────────────────────────────────────────────────┐
│  PER MILESTONE M{N} — repeat until the master plan is done                  │
└─────────────────────────────────────────────────────────────────────────────┘

  @code-implementation plan-iteration - M{N}
        │  Writes ## Current iteration in .work/plans/NEXT.md from master plan
        ▼
  @code-implementation start
        │  Read SPECs + standards; first task in-progress
        ▼
  @code-implementation continue     ◄──┐
        │  Per task: implement → task gate (tests · lint · type)  │
        │  Schema change? → @db-migration create - …              │
        └────────────────────────────────────────────────────────┘
        ▼
  @code-verify milestone            ← plan + SPEC audit before you claim done
        ▼
  @code-implementation complete     ← archive iteration; update HANDOFF / NEXT
        │
        └──► next M{N+1} or @session-control close

┌─────────────────────────────────────────────────────────────────────────────┐
│  ANYTIME — supporting skills (invoke when the work needs them)               │
└─────────────────────────────────────────────────────────────────────────────┘

  @process-router - <question>      lost? read-only signpost (no file writes)
  @feature-spec create - <slug>     feature SPEC (see Skills at a glance)
  @concept-run list | - MOD-0N      architecture prompts MOD-01…06 (MOD-06 default for agent code)
  @dev-stack                        Docker dev helper script (bin/start.sh)
  @code-verify uncommitted | last   commit hygiene (milestone mode is in the loop above)
```

### One-line cheat path (copy into chat)

```text
@project-bootstrap init
@plan-foundation greenfield → @plan-foundation certify plan-master-ready
@plan-master greenfield → @plan-master status
@session-control start
@code-implementation plan-iteration - M1 → start → continue
@code-verify milestone → @code-implementation complete
@session-control close
```

**Jump ahead?** If planning is already done, open at [`§ 3 · Open a coding session`](#3--open-a-coding-session-every-day) below. If you only need setup files, see [First-time setup](#first-time-setup-human-or-agent).

### Skills at a glance

All **11** skills live under [`skills/`](skills/README.md). Invoke as `@<skill-id>` plus a mode (e.g. `@plan-foundation status`).

| Skill | One line | Typical invoke |
|-------|----------|----------------|
| **project-bootstrap** | Scaffold `.work/`, `.cursorrules`, stack doc from templates | `init` · `status` |
| **plan-foundation** | Foundation docs 01–04, ADRs, SPECs, registries; certifies **plan-master-ready** | `greenfield` · `status` · `certify plan-master-ready` |
| **plan-master** | Master plan with milestones; certifies **implementation-ready** | `greenfield` · `continue` · `status` · `revise` |
| **session-control** | Session bookends; updates HANDOFF + NEXT | `start` · `close` · `status` |
| **code-implementation** | Run one milestone from `NEXT.md`; per-task gates | `plan-iteration - M{N}` · `start` · `continue` · `complete` |
| **code-verify** | Audits (not implementation): milestone, dirty tree, last commit/push | `milestone` · `uncommitted` · `last` |
| **feature-spec** | Author, review, or amend feature SPECs | `create - <slug>` · `review - <path>` |
| **concept-run** | Run MOD-01…06 architecture/NFR prompts | `list` · `- MOD-06` (required for agent-assisted code) |
| **db-migration** | Idempotent numbered SQL scripts (no Alembic chain) | `create - <description>` |
| **dev-stack** | Generate or update isolated Docker `bin/start.sh` | `@dev-stack` (see skill for modes) |
| **process-router** | Read-only: “how do I…?” → right skill or guide | `— <question>` · `help` |

Gates between skills: [`skills/SKILL_DEPENDENCIES.md`](skills/SKILL_DEPENDENCIES.md).

---

## First-time setup (human or agent)

Cursor and compatible agents read **`.cursorrules` at the repository root** (sibling to `.ai/` in an application repo, or at this root when Agent OS is the git root).

**Recommended — one command from repo root:**

```bash
bash .ai/templates/bootstrap.sh
```

Or in chat: **`@project-bootstrap init`**

This creates (without overwriting existing files):

- `.cursorrules` from [`templates/cursorrules.template`](templates/cursorrules.template)
- `DOCS_TECH_STACK.md` from [`templates/DOCS_TECH_STACK.md.template`](templates/DOCS_TECH_STACK.md.template)
- `.work/` skeleton (HANDOFF, NEXT, registries, empty plan folders)

Then:

1. Replace every **`REPLACE:`** token in `.cursorrules` ([checklist](templates/README.md)).
2. Customize [`.ai/standards/`](standards/) templates and point `.cursorrules` at your dated filenames.
3. Run **`@plan-foundation greenfield`** (foundation docs 01–04).
4. Keep a **single** rules file — do not add `AGENTS.md` unless your team standardizes on it.

| Situation | What to do |
|-----------|------------|
| **New adoption** | `bootstrap.sh` or `@project-bootstrap init` |
| **Existing `.cursorrules`** | `@project-bootstrap status` — merge manually; do not blind overwrite |

More: [`templates/README.md`](templates/README.md) · skill: [`skills/project-bootstrap/`](skills/project-bootstrap/skill.md)

---

## Mini-tutorial — full lifecycle (agent chat)

Step-by-step detail for each phase. **Panoramic map:** [Bird's-eye — how to use Agent OS](#birds-eye--how-to-use-agent-os) above.

Replace `M1` with the milestone named in `.work/plans/NEXT.md`.  
**Already past planning?** Jump to **§3**.

---

### 1 · Foundation (once per project)

Turn an idea into a **documented, reviewable blueprint** — still no application code.

| Invoke | What happens |
|--------|----------------|
| **`@plan-foundation greenfield`** | Walks you through P0–P6: captures product intent and scope (**doc 01**), integration sources and evidence (**doc 02**), optional product lanes (**doc 03**), architecture and repo layout proposal (**doc 04**). Also seeds **ADRs**, **feature SPECs**, **ASSUMPTIONS / RISKS / UNKNOWNS**, and session files (`HANDOFF`, `NEXT`). |
| **`@plan-foundation status`** | Read-only snapshot: which gates passed, what is missing, whether you are safe to approach the master plan. |
| **`@plan-foundation certify plan-master-ready`** | Deep check that foundation artifacts are consistent and complete enough for **`@plan-master`** — certifies **plan-master-ready**, not implementation-ready. |

```text
@plan-foundation greenfield
@plan-foundation status
@plan-foundation certify plan-master-ready
```

**You should have:** `.work/plans/foundation/` (01–04), registries, SPECs, and a clear “what we are building” story.

---

### 2 · Master plan (once per project)

Turn the blueprint into an **approved execution roadmap** with milestones and tasks.

| Invoke | What happens |
|--------|----------------|
| **`@plan-master greenfield`** | Authors `{PLANS_ROOT}/full/*-full-plan.md`: FR/NFR traceability, milestones **M1…M9**, per-task file lists, acceptance criteria, and links back to foundation + SPECs. |
| **`@plan-master status`** | Read-only: plan exists, **Approved** or still Draft, integrity snapshot, and **`implementation-ready: yes/no`** — only this skill may mark implementation-ready. |

```text
@plan-master greenfield
@plan-master status
```

**Do not start broad coding until** the master plan is **Approved** and status shows **implementation-ready: yes**.

---

### 3 · Open a coding session (every day)

Bookend the day so the next chat does not start from zero.

| Invoke | What happens |
|--------|----------------|
| **`@session-control start`** | Loads `.cursorrules`, `HANDOFF`, `NEXT`, `UNKNOWNS`, and P0 scope; marks the session **Open** with your goal; surfaces owner blockers. |
| **`@code-implementation status`** | Shows the active **`## Current iteration`** block: which tasks are pending, in progress, done, or blocked. |

```text
@session-control start
@code-implementation status
```

---

### 4 · Plan and run one milestone (repeat per M{N})

Pick one milestone from the master plan and execute it task by task.

| Invoke | What happens |
|--------|----------------|
| **`@code-implementation plan-iteration - M1`** | Builds or validates the **`## Current iteration`** section in `NEXT.md` from master-plan **M1** (task IDs, files, acceptance notes). Required before the first line of code. |
| **`@code-implementation start`** | Reads the relevant **SPECs** and **CONVENTIONS**, then implements the **first** task in the iteration. |
| **`@code-implementation continue`** | Picks up the next incomplete task; runs the **task gate** (your project's test/lint/type commands from `.cursorrules`) before marking `done`. Repeat until all tasks are finished. |
| **`@db-migration create - …`** | *Only if the task changes schema.* Writes an idempotent numbered SQL script under your migrations dir (see `.cursorrules`) — never inline DDL in app code. |
| **Dev stack script** | *First time this milestone needs runtime.* Use your project's dev-stack entry (e.g. `bin/start.sh` from `@dev-stack`) to start the isolated compose stack. The agent runs checks **inside** containers when Docker is the canonical path. |

```text
@code-implementation plan-iteration - M1
@code-implementation start
@code-implementation continue
```

```text
@db-migration create - <short description>
```

```bash
# Example — use the path from .cursorrules (REPLACE:DEV_STACK_SCRIPT)
./bin/start.sh
```

---

### 5 · Close the milestone

Prove the milestone is really done, then freeze progress in project memory.

| Invoke | What happens |
|--------|----------------|
| **`@code-verify milestone`** | Audits the iteration against the master plan and SPECs: tests/lint/type evidence, scope, traceability gaps — **pass** before you claim the milestone. |
| **`@code-implementation complete`** | Finalizes the iteration: moves work to **Done** in `NEXT.md`, refreshes `HANDOFF`, archives the iteration block, promotes residual risks to `UNKNOWNS` when needed. |

```text
@code-verify milestone
@code-implementation complete
```

---

### 6 · End the session

Leave a clean handoff for your future self (or the next agent).

| Invoke | What happens |
|--------|----------------|
| **`@session-control close`** | Updates `HANDOFF` + `NEXT`, lists follow-ups, and **always** shows a draft commit message — **no git** unless you add `commit`. |
| **`@session-control close commit`** | Same as close, then stages and commits with that message (refuses if secrets are in the diff). |
| **`@session-control close commit push`** | Commit + push current branch. |

```text
@session-control close
@session-control close commit
@session-control close commit push
```

**Next session:** `@session-control start` → read **Recommended next** in `.work/plans/NEXT.md` → pick the next `M{N}`.

---

## Quick commands (cheat sheet)

| Moment | Command |
|--------|---------|
| Where am I? | `@session-control status` |
| How do I…? | `@process-router - <question>` |
| Stuck on process | [`START_HERE.md`](START_HERE.md) |
| Dirty tree before commit | `@code-verify uncommitted` |
| After you committed | `@code-verify last` |

---

## What you get

| Pain | Fix |
|------|-----|
| Re-explaining workflows every chat | **Skills** — `@session-control`, `@code-implementation`, … |
| Drifty code style | **Standards** — conventions, SPEC template, directory map |
| Architecture surprises at AI speed | **Concepts** MOD-01…06 — run before big splits |
| Context evaporates overnight | **`.work/`** + session bookends |

---

## How it fits together

```text
  You + agent (@skills)                 Project truth (.work/)
       │                                      │
       │   plan-foundation → plan-master      │  foundation/ · full/
       │   code-implementation → verify       │  features/ · HANDOFF · NEXT
       ▼                                      ▼
  .ai/  skills · standards            .work/  plans · SPECs · ADRs
        concepts · guides                     session + iteration state
       │                                      │
       └──────────── @session-control ────────┘
```

| Layer | Role |
|-------|------|
| **`.ai/`** | *How* we work — skills, standards, concepts, guides (copy to other repos). |
| **`.work/`** | *What* this project decided — plans, SPECs, HANDOFF, NEXT. |
| **`.cursorrules`** | Binding agent rules at the **repo root** ([template](templates/cursorrules.template)). |

Skill prerequisite gates: [`skills/SKILL_DEPENDENCIES.md`](skills/SKILL_DEPENDENCIES.md).

---

## What’s inside `.ai/`

| Folder | Role |
|--------|------|
| [`skills/`](skills/README.md) | Executable playbooks — full registry |
| [`standards/`](standards/) | Engineering contract **templates** (customize per project) |
| [`concepts/`](concepts/README.md) | MOD-01…06 architecture prompts |
| [`docs/guides/workflows/`](docs/guides/workflows/README.md) | Tutorials + artifact matrix |
| [`docs/integration/`](docs/integration/) | Vendor mirror layout + `MANIFEST` template (project adds artifacts) |
| [`templates/`](templates/README.md) | **`cursorrules.template`** — copy to repo root as `.cursorrules` |
| `plans/`, `features/`, … | **Pointers only** → `.work/` |

---

## Required reads (when work is active)

1. [`START_HERE.md`](START_HERE.md)
2. **`.cursorrules`** (repo root — install via [First-time setup](#first-time-setup--install-cursorrules-human) if missing)
3. `.work/context/HANDOFF.md`
4. `.work/plans/NEXT.md`
5. `.work/plans/foundation/*-01-*-initial-scope.md` when present
6. For code: customize then use your dated standards under `standards/` (templates ship as `20260517-*.md` — rename or copy after replacing `REPLACE:` tokens)

Agent rules file: **`.cursorrules` only** — do not add `AGENTS.md` without owner approval.

---

## Copy to another project

1. Copy the whole **`.ai/`** tree (includes `templates/`).
2. At the **new repo root**, run **`bash .ai/templates/bootstrap.sh`** (or `@project-bootstrap init`).
3. Fill **`REPLACE:`** tokens in `.cursorrules`; customize **standards** under `.ai/standards/`.
4. Follow the [bird's-eye flow](#birds-eye--how-to-use-agent-os): foundation → certify → master plan → status → daily session + milestones.

Template sources: [`templates/work/`](templates/work/). This repo includes a demo [`.work/`](.work/) skeleton when Agent OS is the git root.
