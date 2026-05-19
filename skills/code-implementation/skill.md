---
name: code-implementation
description: >-
  Execute an approved implementation iteration: validate or generate the
  NEXT.md iteration block from the plan-master milestone, implement tasks per
  CONVENTIONS and FEATURE_STANDARD, gate each task on tests/lint, and finalize
  the iteration. Verification modes live in the **code-verify** skill. Use when the
  user says code-implementation start, continue, complete, plan-iteration, or status.
  Requires implementation-ready (plan-master Approved) or explicit HANDOFF waiver.
---

# code-implementation

Execute implementation iterations derived from an **Approved master plan** (`{PLANS_ROOT}/full/YYYYMMDD-full-plan.md`). Each iteration is scoped by a `## Current iteration` block in `NEXT.md` — validated before the first line of code, gated per task on tests/lint, and cross-verified before completion.

**Tool-agnostic** (Cursor, Claude Code, opencode, Codex). **Requires:** `implementation-ready: yes` from `@plan-master status`, or an explicit HANDOFF waiver noting which milestones may proceed early.

**Pairs with:** `session-control` (bookends), `plan-master` (milestone source and revisions), `code-verify` (milestone / uncommitted / last audits), `db-migration` (all schema changes), `.ai/standards/CONVENTIONS`, `.ai/standards/FEATURE_STANDARD`.

**Canonical path:** `.ai/skills/code-implementation/skill.md` · **Invocation examples:** `reference.md`

**Hard rules:**

- **No implementation without a valid iteration block.** If `NEXT.md` lacks one, run `plan-iteration` first.
- **No code without reading the relevant SPEC(s) first.** Evidence-first: read before writing.
- **No task is `done` until its gate passes.** Tests + lint + type-check must exit 0 before advancing.
- **Scope discipline.** Do not modify any file not declared in the task's file list. Undo and document any accidental out-of-scope change.
- **Schema changes go through `db-migration`.** Stop the task, create the migration script, resume. No inline DDL in application code.
- **All commands run in Docker containers.** Never bare `python`, `pytest`, `npm`, `pyright`, or `ruff` on the host.
- **Protected files** (`package.json`, `docker-compose*.yml`, `Dockerfile.*`, `dashboard/next.config.js`, `dashboard/tsconfig.json`, `.env*`) require explicit user permission before modification. Stop and ask.
- **No secrets in code, tests, or comments.** Use `.env` variables or KMS references.
- **Completion Gate is non-negotiable.** Per `.cursorrules`: code changed → checks run → output reviewed → residual risks listed. Cannot be skipped.
- **AI-assisted default:** Cursor/agent sessions are **AI-assisted: yes** for MOD-06 unless the human explicitly declares **`human-only`** in the same message. Agents must not skip MOD-06 by self-classifying.
- **MOD-06 before complete:** `@concept-run — MOD-06` (or documented output path) is **required** before `@code-implementation complete` when any task in the iteration touched `apis/src/` or `apis/tests/`.
- **Every mode ends with a Completion checklist** — each item `pass` | `fail` | `skip` with evidence.

---

## Parse invocation

Normalize the user message to **verb** + optional **target**.

| User says | Verb | Action |
|-----------|------|--------|
| `@code-implementation` **status** | status | Read-only: task matrix, progress snapshot |
| `@code-implementation` **plan-iteration** — M1 | plan-iteration | Generate/validate `## Current iteration` block from plan-master milestone |
| `code-implementation` **start** | start | Load iteration block, read SPECs/CONVENTIONS, begin first task |
| `code-implementation` **continue** | continue | Resume: find first incomplete task, implement, gate, advance |
| `code-implementation` **complete** | complete | Finalize iteration: CO2 `@code-verify milestone` + CO1 gates + update HANDOFF/NEXT |
| `code-implementation` **verify** [uncommitted \| last] | — | **Legacy** — use `@code-verify` (see `code-verify` skill) |
| `code-implementation` **task** T3 | task | Execute a single task by shorthand ID (active iteration context) |
| `code-implementation` **task** M1-T3 | task | Execute a single task by globally unique ID; gate immediately |

**Aliases:** `impl`, `code`, `implement` → map to **continue** if iteration block exists, else **start**.

**Ambiguous:** if `NEXT.md` has an iteration block but status is unknown → run abbreviated **status** and ask once.

---

## Step 0 — Pick a mode

| Mode | Condition | Action |
|------|-----------|--------|
| **status** | progress/matrix/snapshot requested | [Status protocol](#status-protocol) — read-only |
| **plan-iteration** | iteration block missing or invalid; user names a milestone | [Plan-iteration protocol](#plan-iteration-protocol) |
| **start** | valid iteration block exists; no task started | [Start protocol](#start-protocol) |
| **continue** | iteration in-progress; tasks pending or one in-progress | [Continue protocol](#continue-protocol) |
| **complete** | all tasks done or user signals completion | [Complete protocol](#complete-protocol) |
| **task** | user names `T{N}` or `M{N}-T{N}` | Execute that task; run gate; report |

Do not run `plan-iteration` when the user asked for **status** only. For any **verify** request, use **`@code-verify`** (`code-verify` skill).

**Suggested cadence:** `@code-verify uncommitted` before commit · `@code-verify last` after commit/push · `@code-verify milestone` before **complete**.

---

## NEXT.md iteration block format

The `## Current iteration` section is owned by this skill. `session-control` and `plan-foundation` manage other sections; do not delete or rewrite theirs.

```markdown
## Current iteration — M{N}: {milestone name}

**Milestone ref:** M{N} · `{PLANS_ROOT}/full/YYYYMMDD-full-plan.md §19`
**Status:** planning | in-progress | blocked | complete
**Started:** YYYY-MM-DD · **Target:** YYYY-MM-DD (optional)

### In scope
- {explicit list — derived from milestone §Scope in plan-master}

### Out of scope (explicit)
- {anything callers might assume is in scope but is not}

### Tasks
| ID | Description | Files | Status | Notes |
|----|-------------|-------|--------|-------|
| M{N}-T1 | … | `apis/src/…` | pending | |
| M{N}-T2 | … | `apis/src/…` | in-progress | |
| M{N}-T3 | … | `apis/src/…` | done 2026-05-18 | |

### Acceptance criteria
- [ ] …

### Validation steps
- [ ] `docker compose exec api bash -c "cd /code/apis && python -m pytest tests/ -m 'not sandbox'"`
- [ ] Lint: `docker compose exec api bash -c "cd /code/apis && ruff check src/ tests/"`
- [ ] Type: `docker compose exec api bash -c "cd /code/apis && pyright src/ tests/"` (strict, per CONVENTIONS §1)
- [ ] Manual: …

### Owner blockers
- {none | list — each with owner and blocks (task ID or ADR)}

### Cross-LLM verification
- Triggered: no | yes · Date: … · Result: pass | fail | pending · Notes: …

### Done this iteration
| Task | Completed | Notes |
|------|-----------|-------|
```

### Task ID convention

Task IDs use the globally unique **`M{N}-T{N}`** format inherited from the approved plan-master (e.g. `M1-T3`). The shorthand `T{N}` is acceptable within this iteration block and in agent prompts when the active milestone is unambiguous. Always use the full `M{N}-T{N}` form in:

- Cross-milestone references
- HANDOFF and NEXT.md `## Done this iteration` table
- Traceability matrix in the plan-master

### Valid iteration block criteria

An iteration block is **valid** when all of the following are true:

1. Milestone ref present and traces to a task row in the approved plan-master §19.
2. In scope / out of scope sections are explicit (not empty).
3. At least one task row with at least one declared file path.
4. Acceptance criteria section present with at least one item.
5. Validation steps include at least one runnable Docker test command.
6. **`### Concept / NFR registry (this iteration)`** subsection is present with one row per architecture concept id **or** explicit `N/A` for each id with reason (if the repository has no concept pack, one row: `N/A — no pack`).

If any criterion fails → iteration block is **invalid** → run **plan-iteration** before **start**.

---

## Plan-iteration protocol

Generates or validates the `## Current iteration` block in `NEXT.md` from the next incomplete milestone in the approved plan-master.

### PI1 — Verify prerequisites

1. Read `{PLANS_ROOT}/full/YYYYMMDD-full-plan.md` (latest by date prefix). Confirm **Status: Approved** (or owner waiver recorded in HANDOFF for early M1 start).
2. If no approved plan → **stop**. Recommend `@plan-master status` then `@plan-master continue` or user approval.
3. Read `{HANDOFF}` — note owner blockers, waivers, M1-only authorizations.
4. Read `{ITERATION_CARRIER}` — find `## Done this iteration` and `## Recommended next` to determine which milestone is next.

### PI2 — Select target milestone

1. Identify the first incomplete milestone (M1…) in plan-master §19.
2. If user specified `plan-iteration — M{N}`, use that milestone directly; verify it exists in the plan.
3. Check dependencies: if the milestone declares prior milestones as dependencies, confirm they are done (or explicitly waived in HANDOFF).

### PI3 — Derive tasks

Copy the task rows from the plan-master §19 milestone verbatim into the iteration block. **Preserve the `M{N}-T{N}` IDs exactly as defined in the plan** — do not renumber or rename them. If the plan-master uses shorthand IDs, expand them to the full `M{N}-T{N}` form now.

For each task:

- Map to at least one file path (create or modify). If unknown, mark `TBD — owner needed` and add to Owner blockers.
- Link to FR or NFR from plan-master §3–4.
- Read the relevant SPEC(s) under `{FEATURE_SPEC_ROOT}/` for the bounded context. If a rule (R1…) is ambiguous → add to Owner blockers; do not invent a resolution.
- Estimate complexity: S / M / L. Flag L tasks as candidates for splitting.
- Detect schema changes: if any task requires DDL → add a migration sub-task (`db-migration create`) as T{N}a before the implementation task.

### PI4 — Write iteration block

Write the `## Current iteration` section into `NEXT.md` — insert after `## Recommended next`. Do not delete or overwrite other sections.

Include a **`### Concept / NFR registry (this iteration)`** subsection (table or explicit `N/A — no pack`). Rows must align with the **feature SPEC §15** (or equivalent) for the bounded contexts touched, or state why the iteration is **platform-only** with per-MOD `N/A` reasons.

### PI5 — Plan-iteration report

```markdown
## Plan-iteration — M{N}: {name}

**Milestone:** M{N} · {task count} tasks · **Source:** {plan-master path}
**Prerequisites:** pass | fail | waived

### Tasks derived
| ID | Description | Files | Complexity | FR/NFR |
|----|-------------|-------|------------|--------|
| T1 | … | … | S/M/L | FR-{N} |

### Ambiguities / blockers
{list or none — each with: description, blocks task, recommended owner action}

### Completion checklist
| # | Check | Result | Evidence |
|---|-------|--------|----------|
| 1 | Approved plan read | pass/fail | path |
| 2 | Milestone found; dependencies clear | pass/fail | M{N} |
| 3 | Tasks derived with file paths | pass/fail | count |
| 4 | SPEC(s) read for ambiguity check | pass/skip | paths |
| 5 | Migration sub-tasks added (if schema changes) | pass/skip | |
| 6 | Iteration block written to NEXT.md | pass/fail | |
| 7 | Valid iteration block criteria met | pass/fail | criteria |
| 8 | Concept / NFR registry subsection present | pass/fail | rows or N/A |
```

---

## Start protocol

### ST1 — Mandatory reads

| # | File | Pass criteria |
|---|------|---------------|
| 1 | `NEXT.md §Current iteration` | Valid per [criteria](#valid-iteration-block-criteria) |
| 2 | Relevant SPEC(s) for bounded context | Can state: R1… rules, data model, invariants, error codes |
| 3 | `.ai/standards/*CONVENTIONS*` | Can state: module structure, naming, import rules, logging |
| 4 | `.ai/standards/*FEATURE_STANDARD*` | Can state: service/repo/test layout for this context |
| 5 | `{HANDOFF}` | Know: owner blockers, waivers, previous session state |
| 6 | **Optional:** `.ai/docs/guides/workflows/README.md` (artifact matrix) and `20260518-guide-workflows-index.md` (curriculum), or repo equivalents | When concepts/workflows apply: know bootstrap, tutorials, and which artifacts touch plan vs implementation |

If any mandatory read **fails** → stop. Fix before implementing.

### ST2 — Environment snapshot

```bash
git status -sb
git log -1 --oneline
docker compose ps 2>/dev/null || true
```

Record: branch, clean/dirty, running services.

### ST3 — Assumption ledger

Before the first task, state the 3–5 most consequential assumptions for this iteration. Label each:

- **Confirmed** — cite SPEC rule, ADR, or **file path + quoted snippet** (or test command + exit code). Prose alone is not sufficient.
- **Inference** — likely but not proven; note partial evidence.
- **Unverified** — must check; append to `{PLANS_ROOT}/UNKNOWNS.md`.

Do not collapse Inference into Confirmed. A **Confirmed** label without a cite is treated as **Inference** in review.

### ST4 — Select first task

Pick T1 (or first pending task). Mark it `in-progress` in the task table. Announce the task, its file list, and which SPEC rules (R1…) govern it.

### ST5 — Start report

```markdown
## code-implementation start — M{N}: {name}

**Date:** {ISO date} · **Branch:** {branch} · **Tree:** clean | dirty

### Completion checklist
| # | Check | Result | Evidence |
|---|-------|--------|----------|
| 1 | Iteration block valid | pass/fail | criteria |
| 2 | SPEC(s) read | pass/fail | paths |
| 3 | CONVENTIONS read | pass/fail | |
| 4 | FEATURE_STANDARD read | pass/fail | |
| 5 | HANDOFF read | pass/fail | |
| 6 | Environment snapshot | pass/skip | branch, services |
| 7 | Assumption ledger | pass | count |
| 8 | First task selected | pass | T{N} |

### You are cleared to implement when
All mandatory checks (1–5, 7–8) are **pass**. If any **fail**, fix before proceeding.

### First task
**{T{N}}:** {description}
**Files:** {list}
**SPEC rules in scope:** {R1…}
```

---

## Continue protocol

0. **Unblock check:** Read `{PLANS_ROOT}/UNKNOWNS.md` and `{HANDOFF}`. Scan the iteration block for any task with status `blocked`:
   a. For each `blocked` task: find the blocker entry in `### Owner blockers` and/or `UNKNOWNS.md` (entries with `blocks: T{N}`).
   b. Check if the condition has changed: ADR decided, owner action completed, dependency landed, or HANDOFF lists the blocker as resolved.
   c. If resolved → flip task status from `blocked` to `pending`; annotate `unblocked YYYY-MM-DD — <reason>`. If the corresponding UNKNOWNS row is resolved, update its `Status` to `Resolved` with date.
   d. If unchanged → leave as `blocked`. If all tasks are `blocked` with no changes → do not advance; recommend `@session-control close` with blocker list.
1. Read `NEXT.md §Current iteration`. Find the first task with status `in-progress` or `pending`.
2. If `in-progress`: resume that task with file evidence (read the file before editing).
3. If all tasks `pending`: treat as a fresh start — run ST1–ST5 abbreviated.
4. Per task loop:
   a. Read every file to be modified **before** making any change.
   b. Implement per CONVENTIONS, FEATURE_STANDARD, and the SPEC rules (R1…) that govern this context.
   c. Do not modify files outside the task's declared file list.
   d. When implementation complete → run [Task gate](#task-gate).
   e. On gate **pass**: update task status to `done YYYY-MM-DD`; move to next pending task.
   f. On gate **fail**: report exact error; diagnose; fix; re-run gate. Do not advance until pass.
5. **Schema change detected mid-task:** stop the task, invoke `@db-migration create` for the required migration script, then resume.
6. **Blocked task:** see [Blocked task protocol](#blocked-task-protocol).
7. Every 3 tasks or on user request: output abbreviated [status](#status-protocol).
8. When all tasks are `done`: recommend **complete** mode. Do not auto-finalize.

---

## Task gate

Run after every task implementation before marking `done`. All checks must pass. **Mechanical only** — no master-plan reads. For pre-commit or post-push audits with a verdict report, use `@code-verify` (**uncommitted** / **last** / **milestone**).

| Check | Command | Pass criteria |
|-------|---------|---------------|
| Unit tests for changed context | `docker compose exec api bash -c "cd /code/apis && python -m pytest tests/<context>/ -m 'not sandbox' -x"` | Exit 0 |
| Full suite (smoke) | `docker compose exec api bash -c "cd /code/apis && python -m pytest tests/ -m 'not sandbox' -q"` | Exit 0 |
| Lint | `docker compose exec api bash -c "cd /code/apis && ruff check src/ tests/"` | Exit 0 |
| Type check | `docker compose exec api bash -c "cd /code/apis && pyright src/ tests/"` | Exit 0 (strict mode per CONVENTIONS §1) or documented baseline exceptions |
| No secrets in diff | `git diff --unified=0` reviewed | Same rules as `code-verify` S1 — no keys, tokens, passwords, PEM material |
| Protected files | `git diff --name-only` reviewed | No `.cursorrules` §Protected Files paths unless user explicitly approved |
| Scope discipline | `git diff --name-only` | All paths in declared task file list |
| MOD-06 (AI-assisted) | `@concept-run — MOD-06` when iteration touched code | Output attached to PR, task `Notes`, or iteration registry; **required** before **complete** (see CO1) |

**Also verify (manual — no single exit code):**

- **Residual risks / deferred sub-work:** Before marking a task `done`, any follow-up work, untested edge cases, or known limitations discovered during implementation must be captured in the task's `Notes` column or appended to `{PLANS_ROOT}/UNKNOWNS.md` with a new U* row. Do not let deferred sub-work exist only in the agent report — promote it to a tracked artifact.  
- **Observability:** If the task touches HTTP handlers, jobs, logging, or outbound calls: confirm fields and correlation/trace behavior match the feature SPEC §9 and the project observability standard; otherwise note `n/a — no observability surfaces touched`. Note: the project observability standard is **Draft** (`.ai/standards/20260517-observability-spec.md`); verify against observable dimensions listed there, not as a binding schema.  
- **Concept / MOD prompts:** Cursor/agent sessions are **AI-assisted: yes** by default (see `.ai/concepts/README.md` § Trigger table). Run MOD-06 per task or batch before **complete**; attach output skeleton to PR or iteration `Notes`. For multi-package edits, attach coupling-audit (MOD-01) when boundaries crossed. Rows in `### Concept / NFR registry` with `Applies=yes` must not remain `pending` before **complete** (CO1).

**On any gate failure:** report exact output. Diagnose root cause. Fix. Re-run. Do not mark task `done` or proceed to the next task until all checks pass.

**Scope violation** (file outside task list modified): stop immediately. Undo or stash the out-of-scope change. Document why in task Notes. If the out-of-scope change is genuinely necessary, update the task file list and note the expansion before proceeding.

---

## Status protocol

Read-only. No file writes.

1. Read `NEXT.md §Current iteration`.
2. Run `git diff --stat` and `git log --oneline -5`.
3. Count: pending / in-progress / done / blocked tasks.
4. Output:

```markdown
## code-implementation status — M{N}: {name}

**Status:** {iteration status} · **Date:** {ISO date}
**Branch:** {branch} · **Tree:** clean | dirty

### Task matrix
████░░░░░░ 40% (2 / 5 tasks done)

| ID | Description | Status | Files | Notes |
|----|-------------|--------|-------|-------|
| T1 | … | done 2026-05-18 | `…` | |
| T2 | … | done 2026-05-18 | `…` | |
| T3 | … | in-progress | `…` | |
| T4 | … | pending | | |
| T5 | … | blocked | | owner: CPA approval |

### Acceptance criteria
| Criterion | Status |
|-----------|--------|
| … | pass / pending / fail |

### Cross-LLM verification
{status from iteration block}

### Owner blockers
{list or none}

### Residual risks / unknowns
{from UNKNOWNS.md entries opened this iteration}
```

**Visual / datagrid output:** on user request for a visual summary, render the task table with alignment and a text progress bar. Do not use emojis unless the user requests them.

---

## Verification (delegated)

All verify modes moved to **`code-verify`** (`.ai/skills/code-verify/skill.md`):

| Mode | When |
|------|------|
| `@code-verify milestone` | Before **complete**, ≥80% tasks, or full iteration audit |
| `@code-verify uncommitted` | Dirty tree, pre-commit |
| `@code-verify last` | After commit or push — audits whichever event was **last** |

Legacy `@code-implementation verify` → run **`@code-verify milestone`**.

---

## Complete protocol

**Execution order:** **CO2 → CO1 → CO3 → CO4 → CO5 → CO6** (milestone verify before final gates — avoids duplicate full-suite runs).

### CO2 — Verify (mandatory before complete)

Run **`@code-verify milestone`** if not run since the last task was completed. Verdict must be **pass** or **pass with gaps** (waivers documented in HANDOFF). **fail** blocks completion.

### CO1 — Full iteration gate

Run remaining iteration validation steps (manual checks and any steps not covered by CO2 shared Docker gates).

**Skip duplicate suite:** If CO2 shared Docker gates (pytest, ruff, pyright on `src/ tests/`) already **passed** on the **current working tree** (no file changes since the milestone verify report), skip re-running them — record **skip — covered by CO2**. Otherwise run:

```bash
docker compose exec api bash -c "cd /code/apis && python -m pytest tests/ -m 'not sandbox'"
docker compose exec api bash -c "cd /code/apis && ruff check src/ tests/"
docker compose exec api bash -c "cd /code/apis && pyright src/ tests/"
```

All **manual** validation steps in the iteration block (e.g. `curl /health`) must still pass. Failures must be fixed or waived with a documented owner line in HANDOFF.

**Concept/NFR registry gate:** Before completing a milestone, inspect `NEXT.md` `### Concept / NFR registry (this iteration)`. Any row with `Applies=yes` and `Status=pending` must be resolved — either run `@concept-run` for that MOD id, or document a `gap — <reason>` waiver with owner. CO1 fails if unresolved applicable concept rows remain.

**MOD-06 gate (mandatory when code changed):** If any task in the iteration modified `apis/src/` or `apis/tests/`, CO1 **fails** unless MOD-06 output is attached (PR body, task `Notes`, or iteration registry with `Status=done` and evidence path). **`human-only`** opt-out requires explicit human declaration in the same message — agents cannot grant it retroactively.

### CO3 — Documentation updates

If the iteration surfaced documentation gaps:

- **SPEC amendment** → create `{FEATURE_SPEC_ROOT}/<context>/YYYYMMDD-SPEC-amendment-NN.md`. Do not edit a merged SPEC.
- **Full Plan gap** → note in HANDOFF; recommend `@plan-master revise — <reason>`.
- **Foundation gap** → note in HANDOFF; recommend `@plan-foundation continue` (rare — only for structural omissions).
- **New assumptions / risks / unknowns surfaced** → append to `{PLANS_ROOT}/ASSUMPTIONS.md`, `RISK_REGISTRY.md`, `UNKNOWNS.md` respectively.

Do not edit archived decision prompts.

### CO4 — NEXT.md update

1. Set iteration block `**Status:** complete`.
2. Move all task rows to `### Done this iteration` with completion dates.
3. Set `## Recommended next` to the next milestone: `@code-implementation plan-iteration — M{N+1}` or the first task of M{N+1} if already scoped.
4. Clear the `## Current iteration` body or replace with a one-line reference: `M{N} complete — see Done section.`

### CO5 — HANDOFF update

Append to `## What this cycle produced`:
- Row per new file created.
- Row per existing file significantly modified.

Refresh `## Repository state` with current truth: which milestones are done, what is next, any owner actions pending.

### CO6 — Close report

```markdown
## code-implementation complete — M{N}: {name}

**Date:** {ISO date} · **Branch:** {branch}

### Completion checklist
| # | Check | Result | Evidence |
|---|-------|--------|----------|
| 1 | All tasks done | pass/fail | T{list} |
| 2 | Full test suite pass (or skip — covered by CO2) | pass/fail/skip | exit 0 |
| 3 | Lint/type-check pass (or skip — covered by CO2) | pass/fail/skip | |
| 4 | Verify pass (CO2; or gaps waived) | pass/fail/waived | verdict |
| 5 | No secrets in diff | pass/fail | |
| 6 | Scope discipline enforced | pass/fail | git diff |
| 7 | Schema changes via migrations | pass/skip | |
| 8 | SPEC amendments written (if needed) | pass/skip | paths |
| 9 | NEXT.md updated | pass/fail | |
| 10 | HANDOFF updated | pass/fail | |

### Commit message (draft — always present)
{per .cursorrules format}

### Follow-ups before next iteration
{ordered list}

### Next
{from NEXT.md — Recommended next}
```

---

## Blocked task protocol

When a task cannot proceed mid-implementation:

1. Mark task status `blocked` in the iteration block.
2. Record blocker in `### Owner blockers` and in `{PLANS_ROOT}/UNKNOWNS.md` (owner = "human" or named role; `blocks: T{N}`).
3. If the blocker is an ambiguity in SPEC or Full Plan → add to UNKNOWNS; surface in next status report; ask the user once.
4. Move to the next non-blocked pending task (if one exists).
5. If all tasks are blocked → do not hallucinate a resolution. Recommend `@session-control close` with the blockers listed explicitly.
6. Do not invent owner-decision resolutions. Pause and ask.

---

## Integration with other skills

| Skill | Integration |
|-------|-------------|
| `session-control` | Run `@session-control start` before `code-implementation start`; run `@session-control close [commit]` after `code-implementation complete` |
| `plan-master` | Source of milestones; use `@plan-master status` to confirm implementation-ready; `@plan-master revise` if plan gaps surface |
| `code-verify` | **milestone** (CO2) before **complete**; **uncommitted** / **last** optional pre-commit / post-push cadence; task gate runs inline checks — does not delegate to verify |
| `plan-foundation` | Rarely invoked during implementation; only when **milestone** verify surfaces a structural foundation gap |
| `db-migration` | Mandatory for any schema change; stop task, run `@db-migration create`, resume after migration script exists |
| **Concept pack** | Run applicable `prompt.md` during **`@code-verify milestone`**; **MOD-06 required** before **complete** when code changed; attach outputs to PR or `NEXT.md` |

---

## Anti-patterns

- Starting implementation without a valid iteration block in NEXT.md.
- Writing code without reading the relevant SPEC(s) first.
- Marking a task `done` before the task gate passes.
- Reporting "tests pass" without running them and reviewing output.
- Modifying files outside the declared task file list.
- Inline `CREATE TABLE` / `ALTER TABLE` in application code.
- Touching protected files without explicit user permission.
- Running bare `pytest`, `python`, `npm`, `pyright`, or `ruff` on the host.
- Skipping the task gate because "it's a small change."
- Proceeding to complete without `@code-verify milestone`.
- Skipping the **Concept / NFR registry** subsection in the active iteration when a concept pack is documented in agent rules.
- Skipping **MOD-06** in a Cursor/agent session by self-classifying as non-AI.
- Passing **Observability** in `@code-verify milestone` without checking SPEC §9 fields on touched code paths.
- Inventing a resolution for an owner-decision blocker.
- Editing merged SPECs or archived decision prompts during implementation.
- Logging PII (emails, tax IDs, amounts) in structured log events.
- Adding attribution comments ("Generated by…", "Created by AI").
- Marking implementation-ready in this skill (that is `@plan-master status`).

---

## Completion checklist (all modes)

| # | Check | Result | Evidence |
|---|-------|--------|----------|
| 1 | Mode detected correctly | pass/fail | |
| 2 | NEXT.md iteration block valid | pass/fail | criteria |
| 3 | SPEC(s) read before implementation | pass/skip | paths |
| 4 | CONVENTIONS + FEATURE_STANDARD read | pass/skip | |
| 5 | Task gate passed per task | pass/fail | exit codes |
| 6 | No out-of-scope files modified | pass/fail | git diff |
| 7 | No secrets in output | pass/fail | |
| 8 | Schema changes via db-migration | pass/skip | |
| 9 | `@code-verify milestone` (complete mode) | pass/skip | verdict |
| 10 | NEXT.md + HANDOFF updated (complete mode) | pass/skip | |
