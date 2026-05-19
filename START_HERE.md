# START HERE — operator decision tree

**Purpose:** Answer one question fast: *"What do I do right now?"*

**Read this file when you sit down, are interrupted, or feel lost.** It is intentionally short. Every other doc in `.ai/` is reference material; **this** is the entry point.

**Rule:** If something below contradicts a `skill.md` or a binding standard, the **skill / standard wins**. Open a PR to fix this file.

---

## 0. Two things to know about this project

1. **Truth before speed.** The agent rules in `.cursorrules` (Core Principles 1–7) are non-negotiable. Never claim "tests pass" without running them and reading the output. If you're not sure, label your statement **Unverified**.
2. **Skills do the orchestration. Standards bind the code. Concepts gate the architecture.** You almost never need to read all three at once — pick what your task needs.
3. **`.ai/` = agnostic process** (skills, standards, guides). **`.work/` = this project's plans, SPECs, ADRs, HANDOFF.** Process how-to: [`.ai/PROCESS_ROUTER.md`](PROCESS_ROUTER.md) or `@process-router help`.

---

## 1. Decision tree (start here)

```text
┌──────────────────────────────────────────┐
│  Where am I right now?                    │
└──────────────────────────────────────────┘
       │
       ├── "Bootstrap Agent OS / empty .work"     ─────────► `@project-bootstrap init` · `bash .ai/templates/bootstrap.sh`
       │
       ├── "I just opened the project / I'm lost" ─────────► §2  Resume / orient
       │
       ├── "I'm lost / how do I…?"                  ─────────► §2  Resume / orient · `@process-router`
       │
       ├── "I want to start a coding task"          ─────────► §3  Implement
       │
       ├── "I need to plan something new"           ─────────► §4  Plan
       │
       ├── "I'm closing for the day"                ─────────► §5  Close
       │
       ├── "Tests / lint / type check failed"        ─────────► §6  Failure recovery
       │
       └── "I want to understand the system"        ─────────► §7  Reading order
```

---

## 2. Resume / orient (≤5 minutes)

You forgot where you were? Run **one** of these — pick the lightest that answers your question.

| Need | Command |
|---|---|
| **Process question (signpost only)** | `@process-router — <question>` · [`PROCESS_ROUTER.md`](PROCESS_ROUTER.md) · `@process-router help` |
| **Where am I / what's next?** | `@session-control status` + `.work/context/HANDOFF.md` + `.work/plans/NEXT.md` |
| One-paragraph status (no writes) | `@session-control status` |
| Where the iteration is (read-only) | `@code-implementation status` |
| Are we still planning or coding? | `@plan-foundation status` (planning) **or** `@plan-master status` (master plan / implementation-ready) |

If those don't help, read **in this order, no more**: `.work/context/HANDOFF.md` → `.work/plans/NEXT.md` (top section: *Recommended next*). That is the ground truth for "what's next."

Then run `@session-control start` to formally open a session.

---

## 3. Implement a coding task

**Before any code:**

```text
@session-control start            # bookend; loads required reads
@code-implementation status        # confirms valid iteration block
```

**If `code-implementation status` says the iteration block is invalid or missing:**

```text
@code-implementation plan-iteration — M{N}
```

Replace `M{N}` with the milestone you're working on (see `NEXT.md ## Recommended next` or `.work/plans/full/*-full-plan.md §19`).

**Then loop:**

```text
@code-implementation start          # first task; reads SPECs, CONVENTIONS, FEATURE_STANDARD
@code-implementation continue        # repeat until all tasks done
@code-verify milestone               # before complete; plan + SPEC check matrix
@code-verify uncommitted             # before commit (dirty tree)
@code-verify last                    # after commit or push (whichever was last)
@code-implementation complete        # finalizes; updates HANDOFF + NEXT
```

**Per-task obligations you must not skip** (these are what you forget):

| Obligation | Source of truth |
|---|---|
| Read the relevant feature SPEC **before** editing | `code-implementation/skill.md § Start protocol ST1` |
| Run the **task gate** (tests + lint + type + secrets + scope) | `code-implementation/skill.md § Task gate` |
| Schema change → **stop and run** `@db-migration create` | `code-implementation/skill.md § Hard rules` |
| AI-assisted diff → run concept prompt | `@concept-run — MOD-06` |
| Diff crosses >1 hard boundary | `@concept-run — MOD-01` |
| Type-check / lint / test commands match `.cursorrules` | `REPLACE:TYPECHECK_COMMAND`, `REPLACE:LINT_COMMAND`, `REPLACE:TEST_COMMAND` |
| Run verification where `.cursorrules` says (container or host) | `.cursorrules` § Docker / local CI |

---

## 4. Plan something new

| You need… | Run |
|---|---|
| To start a brand-new project | `@plan-foundation greenfield` |
| To check if foundation work is done | `@plan-foundation status` then `@plan-foundation certify` |
| To author the master implementation plan | `@plan-master greenfield` (foundation must be `plan-master-ready` first; if a draft plan exists — `@plan-master continue`) |
| To check if you can start coding | `@plan-master status` (only this skill scores `implementation-ready`) |
| A new feature SPEC | `@feature-spec create — <slug>` (see `FEATURE_STANDARD` §3; **do not skip §15**) |
| Concept / NFR prompts (MOD-01…06) | `@concept-run list` · `@concept-run — MOD-06` |
| A new ADR | `.work/decisions/YYYYMMDD-NNN-<slug>.md` — see existing ADRs for shape |
| A schema migration | `@db-migration create — <description>` (idempotent; no Alembic) |

**Three readiness states (do not confuse them):**

```text
foundation-complete  →  plan-master-ready  →  implementation-ready
   (plan-foundation)     (plan-foundation)       (plan-master)
```

Only `plan-master status` can mark `implementation-ready: yes`.

**Skill prerequisite gates (which step blocks the next):** [`.ai/skills/SKILL_DEPENDENCIES.md`](skills/SKILL_DEPENDENCIES.md).

---

## 5. Close for the day

```text
@session-control close                      # safe default — drafts message, no commit
@session-control close commit               # commits with drafted message
@session-control close commit push          # commit + push (requires explicit `push` keyword)
```

The skill will refuse to close cleanly if **secrets are in the diff**, **tests were claimed pass without evidence**, or **HANDOFF/NEXT are stale**.

---

## 6. Failure recovery (tests / lint / type fail)

1. **Do not** mark the task `done`. Per `.cursorrules` Core Principle 2, you cannot claim success when output shows failure.
2. Paste the **actual exit code and last 20 lines** of the failure into the task `Notes` column.
3. Diagnose root cause. Fix. Re-run.
4. If you cannot fix in this session: mark task `blocked` in `NEXT.md ## Current iteration` and add a row to `.work/plans/UNKNOWNS.md` with owner + what's blocked.
5. Never silence a check (`# noqa`, `--no-verify`, baseline files) without an explicit comment citing the reason.

---

## 7. Reading order (when you actually need to learn the system)

In order. Stop when your question is answered.

| Step | File | Why |
|---|---|---|
| 1 | `.cursorrules` | Identity, core principles, protected files, Docker rules. Read **once** per project, then skim. |
| 2 | `.ai/README.md` | Canonical map of `.ai/` (which folder holds what). |
| 3 | `.work/context/HANDOFF.md` | Last session state, owner blockers, repo truth. |
| 4 | `.work/plans/NEXT.md` | Tactical next action + active iteration block. |
| 5 | `.work/plans/UNKNOWNS.md` | Open unknowns, blocked decisions, owner assignments — mandatory at session start. |
| 6 | `.ai/skills/README.md` | Registered skills + naming protocol + typical flow. |
| 7 | `.ai/concepts/README.md` | Concept pack + **trigger table** (when to run which prompt). |
| 8 | `.ai/docs/guides/workflows/README.md` | Artifact matrix (every file's phase + status). |
| 9 | `.ai/docs/guides/workflows/20260518-guide-workflows-index.md` | Curriculum order if you want a tutorial path. |
| 10 | The specific `skill.md` you're invoking | Source of truth for verbs, gates, output shape. |
| 11 | The specific standard (`CONVENTIONS`, `FEATURE_STANDARD`, `DIRECTORY_MAP`, `observability-spec`, `threat-model`, `data-classification`) | Read the **section** that applies; do not read the whole file. |

**Do not read all 18 workflow guides** unless onboarding a new operator. Skills + standards + this file cover ~95% of daily work.

---

## 8. Forgetfulness check (run when something feels off)

Before claiming a task is done, answer **all** of these out loud:

- [ ] Did I read the SPEC rule(s) **before** editing?
- [ ] Did I run the task gate (tests, lint, type-check per `.cursorrules`, secrets scan, scope check) and read the actual exit codes?
- [ ] Did I touch only files in the task's declared file list?
- [ ] If schema changed, did I create a numbered idempotent SQL script under the migrations dir from `.cursorrules`?
- [ ] **AI-assisted default:** Cursor/agent session → MOD-06 **required** unless human declared **`human-only`** in the same message — did I run `@concept-run — MOD-06` and attach output?
- [ ] If concept registry rows were `Applies=yes`, did I run those prompts and update status (none left `pending`)?
- [ ] Did I capture residual risks / deferred sub-work in task `Notes` or `UNKNOWNS.md` (not just the agent report)?
- [ ] Did I update `NEXT.md ## Done this iteration` and `HANDOFF` produced-artifacts?
- [ ] Is the commit message in the format from `.cursorrules` (no surrounding quotes, no AI attribution)?

If any answer is "no" or "I'm not sure" → **fix it before closing**.

---

## 9. Anti-patterns to refuse, even under pressure

These are non-negotiable per `.cursorrules`:

- Claiming PASS when test output shows failure.
- Modifying protected files (see `.cursorrules` §Protected Files) without explicit user approval **in the same message**.
- Running verification on the host when `.cursorrules` requires containers (or the reverse).
- Committing on a default `close` (only `close commit` or `close commit push` may commit).
- AI attribution markers in any artifact ("Generated by", "Created by", signatures).
- Logging full payloads with PII (names, emails, tax IDs, amounts).
- Inventing a resolution for an owner-decision blocker — pause and ask.

---

## 10. Common questions (FAQ)

Use **`@process-router — <question>`** for anything not listed — it routes to the right skill or doc without duplicating them.

**Example:**
**`@process-router — what is the next step`**

| Question | Answer |
|----------|--------|
| What is `process-router`? | Read-only **signpost** — [`PROCESS_ROUTER.md`](PROCESS_ROUTER.md); maps questions → skill + guide (no writes) |
| How do I use `process-router`? | `@process-router help` · `@process-router — how do I start M1?` · `@process-router — which MOD prompt for AI-assisted code?` |
| Where am I / what's next? | `@session-control status` + `.work/context/HANDOFF.md` + `.work/plans/NEXT.md` |
| What does each `@skill` do? | [`README.md`](README.md) § Skills at a glance |
| Ready to code? | `@plan-master status` (implementation-ready) → `@code-implementation start` |
| New feature SPEC? | `@feature-spec create — <slug>` |
| Which concept prompt (MOD)? | `@concept-run list` · `@concept-run — MOD-06` (required for agent/Cursor code sessions unless **`human-only`**) |
| Add a DB table/column? | `@db-migration create — <description>` |
| Fix broken `NEXT.md`? | `20260518-tutorial-next-fix.md` · `@code-implementation plan-iteration — M{N}` |
| Tests/lint/type-check failed? | §6 above · re-run task gate per `.cursorrules` |
| Close session safely? | `@session-control close` · `@session-control close commit` · `@session-control close commit push` |
| Foundation vs master plan? | `plan-foundation` = P0–P6 + **plan-master-ready** · `plan-master` = full plan + **implementation-ready** |
| Read everything? | Don't — §7 reading order; stop when answered |

---

**Maintenance:** This file is *deliberately* short. If a section grows past 20 lines, that's a sign the underlying skill/standard is missing structure — fix the skill, not this file.
