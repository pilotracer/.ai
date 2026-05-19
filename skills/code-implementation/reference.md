# code-implementation — reference

Supplement to `skill.md`. Invocation examples, NEXT.md templates, mode comparison, and edge cases.

---

## How to invoke

### Cursor

```
@code-implementation status
@code-implementation plan-iteration — M1
@code-implementation start
@code-implementation continue
@code-implementation complete
@code-verify milestone
@code-verify uncommitted
@code-verify last
```

Legacy (routes to `code-verify` skill):

```
@code-implementation verify
@code-implementation verify uncommitted
@code-implementation task T3
```

### Claude Code / opencode / Codex

```
Follow .ai/skills/code-implementation/skill.md — status. Read-only.
```

```
Follow .ai/skills/code-implementation/skill.md — plan-iteration — M1.
Derive iteration block from plan-master and write it to NEXT.md.
```

```
Follow .ai/skills/code-verify/skill.md — uncommitted.
Diff-only audit: secrets, scope, tests/lint/type on changed files.
```

---

## Mode comparison

| | status | plan-iteration | start | continue | complete | task |
|---|--------|----------------|-------|----------|----------|------|
| Read NEXT.md | yes | yes | yes | yes | yes | yes |
| Write NEXT.md | no | yes | no | task status | yes | task status |
| Write HANDOFF | no | no | no | no | yes | no |
| Run tests/lint | no | no | no | per task (task gate) | CO2 then CO1 if needed | yes (task gate) |
| Run verify | — | — | — | optional `@code-verify uncommitted` | `@code-verify milestone` (CO2) | — |

**Three check layers:** **task gate** (mechanical, every task) · **`@code-verify uncommitted` / `last`** (diff audit, optional cadence) · **`@code-verify milestone`** (plan + SPEC matrix, before **complete**). Use **`code-verify`** skill for all verify modes.

---

## Typical session flow

```
@session-control start — implement M1 platform skeleton
@code-implementation status               ← check if iteration block exists
@code-implementation plan-iteration — M1  ← if block missing or invalid
@code-implementation start                ← load context, begin T1
@code-implementation continue             ← resume after interruption
@code-implementation status               ← progress check (every 2–3 tasks)
@code-verify milestone                    ← approaching completion
@code-verify uncommitted                  ← before commit
@code-verify last                         ← after commit/push
@code-implementation complete             ← finalize + update HANDOFF/NEXT
@session-control close commit        ← commit and close session
```

---

## Quick invocation reference

| Goal | Prompt |
|------|--------|
| What tasks remain? | `@code-implementation status` |
| Generate iteration scope for M2 | `@code-implementation plan-iteration — M2` |
| Start fresh on current iteration | `@code-implementation start` |
| Resume after interruption | `@code-implementation continue` |
| Run a specific task (shorthand, active iteration) | `@code-implementation task T4` |
| Run a specific task (globally unique ID) | `@code-implementation task M1-T4` |
| Look up what a task requires (read-only) | `@plan-master task M1-T4` |
| See all tasks for a milestone | `@plan-master task M1` |
| Check work against Full Plan | `@code-verify milestone` |
| Audit uncommitted diff only | `@code-verify uncommitted` |
| Audit last commit or push | `@code-verify last` |
| Wrap up iteration | `@code-implementation complete` |
| Visual task matrix | `@code-implementation status` (datagrid/canvas) |

---

## AC Billing System — milestone map

Suggested milestone order (from plan-master reference.md — validate with `@plan-master status`):

| Milestone | Name | Key SPECs |
|-----------|------|-----------|
| M1 | `apis/` platform skeleton | platform CONVENTIONS, DIRECTORY_MAP |
| M2 | Synthetic fixtures F1–F6 | `synthetic-fixtures/SPEC` |
| M3 | `master_data` + `commercial` API stubs | `master-data`, `commercial-documents` SPECs |
| M4 | Fiscal pipeline worker shell | `fiscal-pipeline/SPEC` |
| M5 | Hacienda sandbox E2E | `fiscal-pipeline/SPEC` + integration mirror |
| M6 | Dashboard shell + i18n scaffold | CONVENTIONS §frontend |
| M7 | Counter profile vertical slice | ADR 012 + interaction SPEC |

Always derive actual task lists from the **approved** `{PLANS_ROOT}/full/YYYYMMDD-full-plan.md §19`; the above is illustrative.

---

## NEXT.md iteration block — quick template

```markdown
## Current iteration — M1: apis/ platform skeleton

**Milestone ref:** M1 · `{PLANS_ROOT}/full/20260517-full-plan.md §19`
**Status:** in-progress
**Started:** YYYY-MM-DD

### In scope
- `pyproject.toml`, `acb_platform/` layer, health route, idempotent migration runner, Docker entrypoint (plan-master M1 §19)

### Out of scope (explicit)
- Business domain tables (M3+)
- Tenant middleware (M3-T11)
- Dashboard (M6)

### Tasks
| ID | Description | Files | Status | Notes |
|----|-------------|-------|--------|-------|
| M1-T1 | Create `pyproject.toml` (no Alembic) | `apis/pyproject.toml` | pending | |
| M1-T2 | `acb_platform/settings.py` | `apis/src/acb_platform/settings.py` | pending | |
| M1-T3 | `acb_platform/database.py` | `apis/src/acb_platform/database.py` | pending | |
| M1-T4 | `acb_platform/logging.py` | `apis/src/acb_platform/logging.py` | pending | |
| M1-T5 | `acb_platform/time.py` | `apis/src/acb_platform/time.py` | pending | |
| M1-T6 | `acb_platform/errors.py` | `apis/src/acb_platform/errors.py` | pending | |
| M1-T7 | `migration_runner.py` + `001_init.sql` | `apis/src/acb_platform/migration_runner.py`, `apis/migrations/001_init.sql` | pending | |
| M1-T8 | `main.py` health + lifespan migrations | `apis/src/main.py` | pending | |
| M1-T9 | Docker entrypoint (PG/Redis wait) | `apis/docker/entrypoint.sh` | pending | |

### Acceptance criteria
- [ ] `docker compose up api` healthy; `curl /health` → 200
- [ ] Migration runner idempotent on restart
- [ ] `ruff check` and `pyright --strict` pass on `apis/src/`

### Validation steps
- [ ] `docker compose exec api bash -c "cd /code/apis && python -m pytest tests/unit/ -m 'not sandbox' -x"`
- [ ] `docker compose exec api bash -c "cd /code/apis && ruff check src/ tests/"`
- [ ] `docker compose exec api bash -c "cd /code/apis && pyright src/ tests/"` (strict; per CONVENTIONS §1)
- [ ] Manual: `curl http://localhost:${ACB_HOST_PORT_API:-8000}/health` → `{"status":"ok"}` (expand to `/health/ready` with DB check in M1-T8)

### Owner blockers
- none

### Concept / NFR registry (this iteration)
<!-- Required by skill.md § Valid iteration block criteria #6.
     One row per MOD-01…MOD-06 (or single `N/A — no pack` row if the repo has no concept pack). -->
| Concept id | Applies | Status | Evidence / trigger |
|------------|---------|--------|-------------------|
| MOD-01 | yes/no | pending/done/n-a | … |
| MOD-02 | yes/no | pending/done/n-a | … |
| MOD-03 | yes/no | pending/done/n-a | … |
| MOD-04 | yes/no | pending/done/n-a | … |
| MOD-05 | yes/no | pending/done/n-a | … |
| MOD-06 | **yes** | pending | Cursor/agent session — required before complete unless human-only |

### Cross-LLM verification
- Triggered: no

### Done this iteration
| Task | Completed | Notes |
|------|-----------|-------|
```

---

## Status output examples

### Text progress bar

```
████████░░ 80% (7 / 9 tasks done)   ← illustrative only
```

### Compact datagrid (illustrative — not live status)

```
## M1 — apis/ platform skeleton — in-progress

| ID    | Task                                | Status (example) |
|-------|-------------------------------------|------------------|
| M1-T1 | pyproject.toml                      | done             |
| M1-T2 | acb_platform/settings.py            | done             |
| M1-T7 | migration_runner + 001_init.sql     | pending          |
| M1-T8 | main.py + /health                   | pending          |
| M1-T9 | docker entrypoint                   | pending          |

Verify: not yet triggered
Owner blockers: none
```

### Canvas (when available)

Render the task matrix in a canvas with colour-coded status cells (done=green, in-progress=blue, blocked=red, pending=grey) and an animated progress bar. See the canvas skill for setup.

---

## Cross-LLM verification — prompt template

When triggering cross-LLM verification, use this compact prompt format:

```
Milestone: M{N} — {name}
Objective: {one paragraph from plan-master §19}

Tasks implemented:
- T1: {description} — Files: {list}
- T2: …

SPEC rules in scope (R1…):
{paste numbered rules from the relevant SPEC}

Review for:
1. {highest-risk gap from V2}
2. {second highest-risk gap}
3. {third}

Return: pass | fail | gaps-found, with specific file/rule citations.
```

---

## Integration with session-control

| Event | code-implementation action |
|-------|-----------------------|
| `session-control start` | Optional: run `@code-implementation status` to see current iteration before beginning |
| Interruption mid-task | Run `@session-control close` — HANDOFF notes "in-flight: T{N}" |
| `session-control close commit` | After `code-implementation complete`, provides the draft commit message |

---

## Edge cases

| Situation | Behavior |
|-----------|----------|
| NEXT.md has no `## Current iteration` | Run `plan-iteration` before start |
| Iteration block exists but milestone not in plan-master | Block is invalid; re-run `plan-iteration` with correct milestone |
| Master plan not Approved but HANDOFF has M1 waiver | Proceed with `plan-iteration — M1`; note waiver in start report |
| Schema change discovered mid-task | Stop task; run `@db-migration create`; resume after migration exists |
| Task T3 depends on T2 which is blocked | Mark T3 `blocked (depends on T2)`; surface both in status |
| All tasks done but test suite fails | Do not run complete; fix failing tests; re-run gate |
| Verify returns `fail` | Fix all high/med gaps; re-run verify; only skip low gaps with waiver |
| User asks for implementation-ready check | Redirect: `@plan-master status` — not code-implementation |
| Second model unavailable for cross-LLM (M1–M3) | Log `skipped — single-model session`; does not block complete |
| Second model unavailable for cross-LLM (**M4+ or fiscal/Hacienda milestone**) | **fail** unless owner records a **human architect review** waiver in `{HANDOFF}` (name + date) per `code-verify` § M3 |
| Cursor/agent session: MOD-06 skipped | **fail** at CO1; run `@concept-run — MOD-06` or attach output before complete. **`human-only`** opt-out requires explicit human declaration in the same message |
| Protected file change needed | Stop; explain why; ask explicit permission; only proceed after yes |

---

## Wrong prompts

| Prompt | Problem | Use instead |
|--------|---------|-------------|
| `@code-implementation start` with no NEXT.md iteration block | No scope | `plan-iteration — M{N}` first |
| `@code-implementation complete` with failing tests | Violates gate | Fix tests; rerun gate |
| Marking task done without running tests | Violates hard rule | Run task gate |
| `@code-implementation` for planning-only work | Wrong skill | `@plan-master continue` |
| Asking code-implementation if plan is "implementation-ready" | Wrong skill | `@plan-master status` |
| Using `ruff` / `pytest` without `docker compose exec` | Host environment | Always use Docker exec |

---

## Optional slash commands (team convention)

| Command | Maps to |
|---------|---------|
| `/impl status` | status |
| `/impl plan M1` | plan-iteration — M1 |
| `/impl start` | start |
| `/impl continue` | continue |
| `/impl done` | complete |
| `/impl verify` | `@code-verify milestone` (legacy: routes via `code-verify` skill) |

Document in project README if adopted.
