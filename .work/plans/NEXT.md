# NEXT - planning backlog

> **This is a template file.** In your adopter repo it is maintained by **`@code-implementation`** (the `## Current iteration` block) and **`@session-control close`** (the `## Recommended next` row). In this framework repo it stays as a demo skeleton.

**Updated:** 2026-05-27

---

## Done

| Item | Artifact |
|------|----------|
| Agent OS bootstrap | `.work/` skeleton, `.cursorrules` from template |
| Code-to-registry coverage | `@plan-verify coverage`, `@plan-repair repair - from coverage`, FEATURE_STANDARD §14, reports template |

---

## Blocked on owner

| # | Item | Notes |
|---|------|-------|
| - | (none) | |

---

## Recommended next

| Priority | Item | Notes |
|----------|------|-------|
| **0** | Tag release `v0.2.1` (optional) | After merge; CHANGELOG Unreleased documents coverage work |
| **1** | `@plan-foundation greenfield` | For greenfield adopters only — not required for mature repos |
| **2** | Consumer repos: `@plan-verify coverage` | After pulling framework update (e.g. codeiva) |

---

## Current iteration

*(No active iteration - run `@code-implementation plan - M1` after master plan is **Approved** and `implementation-ready: yes`.)*

```markdown
## Current iteration - M{N}: <milestone name>

**Milestone ref:** M{N} · `{MASTER_PLAN}` §<task section>
**Status:** planning | in-progress | complete
**Started:** YYYY-MM-DD

### In scope
- …

### Out of scope (explicit)
- …

### Tasks
| ID | Description | Files | Status | Notes |
|----|-------------|-------|--------|-------|
| M{N}-T1 | … | … | pending | |

### Acceptance criteria
- [ ] …

### Validation steps
- [ ] Tests: `REPLACE:TEST_COMMAND` (per `.cursorrules`)
- [ ] Lint: `REPLACE:LINT_COMMAND`
- [ ] Type: `REPLACE:TYPECHECK_COMMAND`

### Owner blockers
- none

### Concept / NFR registry (this iteration)
| Concept id | Applies | Status | Evidence / trigger |
|------------|---------|--------|-------------------|
| MOD-06 | yes | pending | AI-assisted session |

### Cross-LLM verification
- Triggered: no

### Done this iteration
| Task | Completed | Notes |
|------|-----------|-------|
```
