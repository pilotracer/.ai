# `.ai/` — **Agent OS**

Portable process for teams that ship with coding agents: **skills** run the playbook, **standards** keep output honest, **`.work/`** remembers what you decided.

You get less re-prompting, fewer “where were we?” threads, and a loop you can run **start → ship → hand off** every session.

**Project artifacts** (plans, SPECs, HANDOFF) → **[`.work/`](../.work/README.md)** · **Lost mid-flight?** → [`START_HERE.md`](START_HERE.md) · **Deep dive** → [deepwiki.com/PiloTracer/.ai](https://deepwiki.com/PiloTracer/.ai)

---

## First-time setup — install `.cursorrules` (human)

Cursor and compatible agents read agent rules from **`.cursorrules` in the repository root**, not from inside `.ai/`. Do this once per repo before your first `@session-control start`.

**From the repo root** (same directory as `.ai/` and `.work/`):

```bash
cp .ai/templates/cursorrules.template .cursorrules
```

Then:

1. Open **`.cursorrules`** and replace every `REPLACE:` token (project name, stack doc path, standards filenames, migrations dir, Docker services, test commands).
2. Commit **`.cursorrules`** at the root next to `.ai/`.
3. Keep a **single** rules file — do not add `AGENTS.md` unless your team explicitly standardizes on it.

| Situation | What to do |
|-----------|------------|
| **New repo** using Agent OS | Run the `cp` command above; customize; commit. |
| **This repo (AC Billing)** | A tailored `.cursorrules` may already exist at the root — **do not overwrite** unless you intend a governance reset. Use the template only as reference or for a fresh fork. |

More detail: [`templates/README.md`](templates/README.md) · source: [`templates/cursorrules.template`](templates/cursorrules.template).

---

## Mini-tutorial — full lifecycle (agent chat)

One straight line: **foundation → master plan → session → milestone → hand off**.  
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
| **`@code-implementation plan-iteration — M1`** | Builds or validates the **`## Current iteration`** section in `NEXT.md` from master-plan **M1** (task IDs, files, acceptance notes). Required before the first line of code. |
| **`@code-implementation start`** | Reads the relevant **SPECs** and **CONVENTIONS**, then implements the **first** task in the iteration. |
| **`@code-implementation continue`** | Picks up the next incomplete task; runs the **task gate** (pytest, ruff, pyright in Docker) before marking `done`. Repeat until all tasks are finished. |
| **`@db-migration create — …`** | *Only if the task changes schema.* Writes an idempotent numbered SQL script under `apis/migrations/` — never inline DDL in app code. |
| **`./bin/start.sh`** | *First time this milestone needs runtime.* Starts the isolated dev stack (Postgres, Redis, API, workers). The agent runs tests **inside** containers; you rarely paste pytest/ruff/pyright yourself. |

```text
@code-implementation plan-iteration — M1
@code-implementation start
@code-implementation continue
```

```text
@db-migration create — <short description>
```

```bash
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
| How do I…? | `@process-router — <question>` |
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
  You + agent                         Project truth
       │                                    │
       ▼                                    ▼
  .ai/  skills · standards          .work/  plans · SPECs
        concepts · guides                   HANDOFF · NEXT
       │                                    │
       └────────── @session-control ────────┘
```

- **`.ai/`** — *how* we work (copy to other repos).
- **`.work/`** — *what* this project decided (stay here).
- **`.cursorrules`** — hard rules at the **repo root** (copy from [`templates/cursorrules.template`](templates/cursorrules.template); see [First-time setup](#first-time-setup--install-cursorrules-human)).

---

## What’s inside `.ai/`

| Folder | Role |
|--------|------|
| [`skills/`](skills/README.md) | Executable playbooks — full registry |
| [`standards/`](standards/) | Binding engineering contracts |
| [`concepts/`](concepts/README.md) | MOD-01…06 architecture prompts |
| [`docs/guides/workflows/`](docs/guides/workflows/README.md) | Tutorials + artifact matrix |
| [`docs/integration/`](docs/integration/) | Vendor artifacts + `MANIFEST.txt` |
| [`templates/`](templates/README.md) | **`cursorrules.template`** — copy to repo root as `.cursorrules` |
| `plans/`, `features/`, … | **Pointers only** → `.work/` |

---

## Required reads (when work is active)

1. [`START_HERE.md`](START_HERE.md)
2. **`.cursorrules`** (repo root — install via [First-time setup](#first-time-setup--install-cursorrules-human) if missing)
3. `.work/context/HANDOFF.md`
4. `.work/plans/NEXT.md`
5. `.work/plans/foundation/*-01-*-initial-scope.md` when present
6. For code: `standards/20260517-CONVENTIONS.md`, `20260517-FEATURE_STANDARD.md`, `20260517-DIRECTORY_MAP.md`

Agent rules file: **`.cursorrules` only** — do not add `AGENTS.md` without owner approval.

---

## Copy to another project

1. Copy the whole **`.ai/`** tree (includes `templates/`).
2. At the **new repo root**, install agent rules (same as [First-time setup](#first-time-setup--install-cursorrules-human)):
   ```bash
   cp .ai/templates/cursorrules.template .cursorrules
   ```
   Edit `.cursorrules` — replace all `REPLACE:` tokens — then commit.
3. Create **`.work/`** with `HANDOFF.md` and `NEXT.md`.
4. Run **`@plan-foundation greenfield`**; daily coding: **`@session-control start`**.
