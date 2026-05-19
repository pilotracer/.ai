# `.ai/` — **Agent OS**

Portable process for teams that ship with coding agents: **skills** run the playbook, **standards** keep output honest, **`.work/`** remembers what you decided.

You get less re-prompting, fewer “where were we?” threads, and a loop you can run **start → ship → hand off** every session.

**Project artifacts** (plans, SPECs, HANDOFF) → **[`.work/`](../.work/README.md)** · **Lost mid-flight?** → [`START_HERE.md`](START_HERE.md) · **Deep dive** → [deepwiki.com/PiloTracer/.ai](https://deepwiki.com/PiloTracer/.ai)

---

## Mini-tutorial — full lifecycle (agent chat)

One straight line: **foundation → master plan → session → milestone → hand off**.  
Replace `M1` with the milestone in `.work/plans/NEXT.md`.  
**Already past planning?** Start at **§3**.

### 1 · Foundation (once per project)

```text
@plan-foundation greenfield
@plan-foundation status
@plan-foundation certify plan-master-ready
```

Produces `.work/plans/foundation/` (docs 01–04), registries, SPECs/ADRs — **no application code yet**.

### 2 · Master plan (once per project)

```text
@plan-master greenfield
@plan-master status
```

Stop until **Approved** and **implementation-ready: yes**. Only `plan-master` certifies that.

### 3 · Open a coding session (every day)

```text
@session-control start
@code-implementation status
```

### 4 · Plan and run one milestone (repeat per M{N})

```text
@code-implementation plan-iteration — M1
@code-implementation start
@code-implementation continue
```

Schema change only:

```text
@db-migration create — <short description>
```

First time this milestone needs a running API/DB (usually M1+), on the **host**:

```bash
./bin/start.sh
```

The agent runs **pytest / ruff / pyright inside Docker** at each task gate — you do not paste those unless debugging.

### 5 · Close the milestone

```text
@code-verify milestone
@code-implementation complete
```

### 6 · End the session

```text
@session-control close
@session-control close commit
@session-control close commit push
```

Next chat: **`@session-control start`** → read `.work/plans/NEXT.md` for the next `M{N}`.

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
- **`.cursorrules`** — hard rules (Docker-only tests, no secret commits, verify before “done”).

---

## What’s inside `.ai/`

| Folder | Role |
|--------|------|
| [`skills/`](skills/README.md) | Executable playbooks — full registry |
| [`standards/`](standards/) | Binding engineering contracts |
| [`concepts/`](concepts/README.md) | MOD-01…06 architecture prompts |
| [`docs/guides/workflows/`](docs/guides/workflows/README.md) | Tutorials + artifact matrix |
| [`docs/integration/`](docs/integration/) | Vendor artifacts + `MANIFEST.txt` |
| `plans/`, `features/`, … | **Pointers only** → `.work/` |

---

## Required reads (when work is active)

1. [`START_HERE.md`](START_HERE.md)
2. `.cursorrules`
3. `.work/context/HANDOFF.md`
4. `.work/plans/NEXT.md`
5. `.work/plans/foundation/*-01-*-initial-scope.md` when present
6. For code: `standards/20260517-CONVENTIONS.md`, `20260517-FEATURE_STANDARD.md`, `20260517-DIRECTORY_MAP.md`

Agent rules file: **`.cursorrules` only** — do not add `AGENTS.md` without owner approval.

---

## Copy to another project

1. Copy `.ai/`.
2. Add a tuned `.cursorrules`.
3. Create `.work/` with `HANDOFF.md` and `NEXT.md`.
4. New project: **`@plan-foundation greenfield`** first; daily coding: **`@session-control start`**.
