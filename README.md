# `.ai/` — **Agent OS**

Portable process for teams that ship with coding agents: **skills** run the playbook, **standards** keep output honest, **`.work/`** remembers what you decided.

You get less re-prompting, fewer “where were we?” threads, and a loop you can run **start → ship → hand off** every session.

**Project artifacts** (plans, SPECs, HANDOFF) → **[`.work/`](../.work/README.md)** · **Lost mid-flight?** → [`START_HERE.md`](START_HERE.md) · **Deep dive** → [deepwiki.com/PiloTracer/.ai](https://deepwiki.com/PiloTracer/.ai)

---

## Mini-tutorial — one day, start to finish

Copy this block into a new session. Replace `M4` with your milestone from `.work/plans/NEXT.md`.

### 1 · Boot the stack (host)

```bash
./bin/start.sh
# Pick: start / up — brings up pg, redis, api, workers, etc.
# Health check (optional):
curl -s "http://localhost:${ACB_HOST_PORT_API:-8000}/health"
```

All Python/test commands below run **inside Docker** — not on the host.

### 2 · Open session (agent chat)

```text
@session-control start
@code-implementation status
```

If there is no active iteration yet:

```text
@code-implementation plan-iteration — M4
```

### 3 · Build the milestone (agent chat — repeat until done)

```text
@code-implementation start       # first task in NEXT.md
@code-implementation continue    # every task after that
```

Schema change? Stop and run:

```text
@db-migration create — <short description>
```

### 4 · Verify (terminal — after each task or before complete)

```bash
docker compose exec api bash -c "cd /code/apis && python -m pytest tests/ -m 'not sandbox'"
docker compose exec api bash -c "cd /code/apis && ruff check src tests"
docker compose exec api bash -c "cd /code/apis && pyright --strict src tests"
```

### 5 · Sign off the milestone (agent chat)

```text
@code-verify milestone
@code-implementation complete
```

### 6 · Close and hand off (agent chat)

```text
@session-control close
# optional — agent runs git for you:
@session-control close commit
@session-control close commit push
```

**Done.** Next person (or next chat) runs `@session-control start` and reads `.work/plans/NEXT.md`.

---

### Greenfield? (planning before code)

Only when the repo has no approved master plan yet:

```text
@plan-foundation greenfield
@plan-foundation certify plan-master-ready
@plan-master greenfield
@plan-master status                    # must show implementation-ready
@code-implementation plan-iteration — M1
```

Then jump to **§3** above.

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
4. Teach the team: **`@session-control start`** on day one.
