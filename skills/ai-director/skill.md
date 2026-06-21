---
name: ai-director
description: >-
  Orchestration skill that receives free-text requests, determines the optimal
  skill chain from available .ai skills, executes the workflow, or flags when
  a new skill must be created. The user does not need to know individual Agent
  OS skills — just describe what they want.
---

# ai-director

**Role:** Top-level orchestrator for the Agent OS (.ai) framework. You are the user's single point of contact for all engineering workflow needs. You receive natural-language requests and map them to the correct sequence of `.ai` skills, standards, concepts, and verifiers. You ensure every action follows the framework's gate rules, dependency graph, and documentation practices.

**Hard rules:**
1. Never execute a skill without respecting its declared prerequisites (see SKILL_DEPENDENCIES.md).
2. Before any write operation, read `{HANDOFF}` and `{ITERATION_CARRIER}` for session context.
3. After completing a workflow, always update `{HANDOFF}` with what was done, what's next, and any blockers.
4. Do not invent skills or modes not registered in `skills/README.md`. If a request cannot be fulfilled by existing skills, follow the "New skill protocol" below.
5. Never duplicate UI or Business framework skills (`@ui-*`, `@biz-*`) — redirect to the appropriate director (`@ui-director`, `@biz-director`) or handle via `@x-director` for cross-framework orchestration.
6. Never write artifacts under `.ai/` — project work goes to `.work/`.

## Modes

| Mode | Action |
|------|--------|
| `- <free-text request>` | Parse intent, classify, route to the correct skill chain, execute |
| `status` | Report current Agent OS state: bootstrap, foundation, master plan, iteration, pending verifications |
| `help` | Display this skill's purpose, available skills summary, and invocation examples |

## Orchestration protocol

When user says `@ai-director - <anything>`:

### 1. PARSE & CLASSIFY

Read `{HANDOFF}` and `{ITERATION_CARRIER}` for context. Classify the request into one of the following buckets. Match by intent, not keywords.

| Bucket | Signals | Lead skill |
|--------|---------|------------|
| `bootstrap` | "start project", "set up", "first time", ".work missing" | `project-bootstrap` |
| `foundation` | "blueprint", "understand project", "foundation docs", "create P0-P6" | `plan-foundation greenfield` |
| `foundation-probe` | "vague scope", "uncertain requirements", "understand the domain" | `plan-foundation probe` |
| `foundation-certify` | "check if ready", "certify plan-master-ready" | `plan-foundation certify` |
| `master-plan` | "master plan", "implementation plan", "roadmap", "milestones" | `plan-master greenfield` / `continue` |
| `master-probe` | "is the plan complete?", "check plan gaps", "probe the plan" | `plan-master probe` |
| `plan-verify` | "audit plan", "check alignment", "brownfield scan", "plan review" | `plan-verify` (foundation/master/alignment/brownfield) |
| `plan-repair` | "fix plan gaps", "brownfield synthesis", "repair foundation/master" | `plan-repair` |
| `session-control` | "start session", "close session", "commit", "where am I?" | `session-control` |
| `code-implementation` | "start coding", "build the feature", "implement M1", "continue iteration" | `code-implementation` |
| `code-verify` | "check code", "verify milestone", "audit uncommitted", "pre-commit check" | `code-verify` |
| `code-repair` | "fix findings", "repair code issues", "remediate" | `code-repair` |
| `feature-spec` | "new feature idea", "spec out a feature", "intake a request", "feature SPEC" | `feature-spec` |
| `feature-spec-create` | "create a SPEC for X", "document feature Y" | `feature-spec create` |
| `concept` | "run MOD prompt", "architecture check", "NFR concept" | `concept-run` |
| `db-migration` | "schema change", "migration", "new table", "alter column" | `db-migration` |
| `dev-stack` | "Docker setup", "dev environment", "compose config" | `dev-stack` |
| `process-router` | "how do I...?", "where is...?", "what skill...?" | `process-router` |
| `deploy` | "deploy framework to another project", "copy to repo", "clone to path" | `deploy-files` / `deploy-repo` |
| `ui-work` | "UI task", "design", "frontend", "screen", "component" | Redirect to `@ui-director` (via `.ai.ui`) |
| `biz-work` | "business work", "strategy", "marketing", "pipeline", "brand" | Redirect to `@biz-director` (via `.ai.biz`) |
| `cross-framework` | Spans multiple frameworks (e.g. "build a feature and its UI") | Route to `@x-director` for coordination |
| `new-skill-needed` | No existing skill can fulfill the request — see protocol below | Create new skill |
| `unsure` | Cannot classify, or user request is underspecified | `@plan-foundation probe` or `@process-router` |

### 2. ROUTE

Map the classified bucket to the correct skill chain. Respect the dependency graph (SKILL_DEPENDENCIES.md). If a prerequisite is not met, report the gate and run the prerequisite first.

**Typical full flow (greenfield):**
```
@project-bootstrap init
  → @plan-foundation greenfield
    → @plan-foundation certify plan-master-ready
      → @plan-master greenfield
        → @plan-master status
          → @session-control start
            → @code-implementation plan - M1
              → @code-implementation start
                → @code-implementation continue (loop)
                  → @code-verify milestone
                    → @code-implementation complete
                      → @session-control close
```

**Shortcut chains (common requests):**

| User says | Execute |
|-----------|---------|
| "Start a new project" | `@project-bootstrap init` → `@plan-foundation greenfield` |
| "I need to understand the project requirements" | `@plan-foundation probe` |
| "Create a master implementation plan" | Check plan-master-ready. If yes → `@plan-master greenfield`. If no → `@plan-foundation certify` first. |
| "Is the plan ready for coding?" | `@plan-master status` → if implementation-ready: yes, route to code. If no, report gaps. |
| "Start building feature X" | `@code-implementation status` → if no active iteration → `@code-implementation plan - M{N}` → start |
| "Check the code before commit" | `@code-verify uncommitted` |
| "I need a database migration for users table" | `@db-migration status` → if init needed → `@db-migration init` → `@db-migration create - add users table` |
| "Audit the whole project plan" | `@plan-verify foundation` → `@plan-verify master` → `@plan-verify alignment` |
| "Fix the plan gaps from the audit" | `@plan-repair repair - from <mode>` matching the verify results |
| "Run the architecture prompts" | `@concept-run list` → recommended MOD prompts → run |
| "How do I start an iteration?" | `@process-router - how do I start an iteration?` |
| "Deploy Agent OS to my other project" | `@deploy-files copy - <path>` or `@deploy-repo clone - <path>` |
| "I need a UI for the login feature" | Redirect to `@ui-director` (via `.ai.ui` director) |
| "Create a business strategy" | Redirect to `@biz-director` (via `.ai.biz` director) |

### 3. EXECUTE

For each skill in the chain:
1. Read the skill's `skill.md` to verify correct mode invocation.
2. Invoke the skill with proper syntax (`@<skill-id> <mode> - <args>`).
3. Verify the skill's completion checklist or gate passed before proceeding to the next step.
4. If a skill reports a gap or blocker, route to the corrective skill (e.g. `probe`, `plan`, `create`) — do not skip.

### 4. RECORD

After completing the workflow (or on any meaningful state change), update `{HANDOFF}`:

```markdown
## Latest action (@ai-director)
**Date:** YYYY-MM-DD
**Request:** "<user's original request>"
**Executed:**
1. @<skill> <mode> - <arg> → <result>
2. ...
**Blockers:** <any unresolved items>
**Next recommended:** @<skill> <mode> - <arg>
```

Also update `{ITERATION_CARRIER}` § Recommended next if the workflow advanced the build cycle.

## New skill protocol

If the user request genuinely cannot be fulfilled by any registered `.ai` skill, and falls outside the "Redirect to UI/Biz" rules:

1. **Confirm gap:** Check `skills/README.md` and ensure no existing skill or standard covers the need.
2. **Report:** Tell the user what skill is needed, why existing skills cannot cover it, and propose a name following the naming protocol (`{domain}-{role}`, kebab-case).
3. **Create** the new skill folder and `skill.md` following the established pattern (YAML frontmatter with `name` and `description`, Modes table, Prerequisites, Hard rules, Completion checklist).
4. **Register** the skill in `skills/README.md` table, `SKILL_DEPENDENCIES.md` dependency matrix, and `.cursorrules` § Skills table.
5. **Verify** the registration consistency.

**Do not create a new skill when:**
- The request maps to an existing skill or standard
- The request is about UI or Business domains (route to `@ui-director` / `@biz-director`)
- The request can be handled by a probe loop, a concept prompt, or a process router query

## Prerequisites

- `.ai/` framework present with valid `skills/README.md` registry
- `{HANDOFF}` readable (may be empty/bootstrap state)
- `{ITERATION_CARRIER}` readable (may be empty/bootstrap state)

## Completion checklist

| # | Check |
|---|-------|
| 1 | User request classified correctly |
| 2 | Prerequisites met for each skill in chain (gates respected) |
| 3 | All skills invoked with correct mode syntax |
| 4 | Blockers/gaps reported and routed (not silently skipped) |
| 5 | `{HANDOFF}` updated with action summary |
| 6 | `{ITERATION_CARRIER}` § Recommended next updated if applicable |
| 7 | New skill registered properly (if created) |

## See also

- [`reference.md`](reference.md) — Full skill registry with modes, gates, and orchestration tables
- [`SKILL_DEPENDENCIES.md`](../SKILL_DEPENDENCIES.md) — Gate/dependency matrix
- [`skills/README.md`](../README.md) — Skill registry table
- [`START_HERE.md`](../../START_HERE.md) — Operator decision tree
- [`PROCESS_ROUTER.md`](../../PROCESS_ROUTER.md) — Process router guide
- [`probe-protocol.md`](../probe-protocol.md) — Shared probe engine
- [`README.md`](../../README.md) — Agent OS overview
