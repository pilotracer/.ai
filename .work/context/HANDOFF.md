# HANDOFF - session boundary

> **This is a template file.** In your adopter repo it is rewritten by **`@session-control start`** / **`@session-control close`** every session. In this framework repo it stays as a demo skeleton so pointer links resolve.

## Session status

**Closed:** 2026-05-27 — Added `@plan-verify coverage` and `@plan-repair repair - from coverage` for code-to-SPEC registry parity; synced framework guidance into START_HERE, standards, and templates.

**Updated:** 2026-05-27

**Repository state:** Agent OS framework repo (self-hosted). Coverage/registry verify-repair path documented; `framework-verify.sh` passed this session. Codeiva consumer patch applied out-of-band at `/mnt/data/Projects/CODE/RECODE/codeiva` (not in this git tree).

**Recommended pick-up file:** `.work/plans/NEXT.md`

**Lost or new?** Read `START_HERE.md` (from repo root).

---

## Fresh start - what the next session should do first

1. Run **`@session-control start`** (or follow the manual list in `session-control` skill).
2. Read **`.cursorrules`**.
3. Read **P0 initial scope** when present: `.work/plans/foundation/*-01-*-initial-scope.md`.
4. Read **this file** through §Fresh start, then §Open owner actions.
5. Read `.work/plans/NEXT.md`.
6. Read `.work/plans/ASSUMPTIONS.md`, `RISK_REGISTRY.md`, `UNKNOWNS.md`.

End with **`@session-control close`** (add `commit` / `commit push` only when requested).

### Conditional reads (customize per project)

| If the task touches… | Read first |
|----------------------|------------|
| Product scope / foundation | `.work/plans/foundation/*-01-*.md` … `*-04-*.md` |
| Any code or new feature | `.ai/standards/*CONVENTIONS*`, `*FEATURE_STANDARD*` |
| External integration | `*-02-*.md`, `.ai/docs/integration/MANIFEST.txt` (if any) |
| Security | `.ai/standards/*threat-model*` |
| Stack / topology | `REPLACE:TECH_STACK_DOC` |
| Master plan / milestones | `.work/plans/full/*-full-plan.md` |
| High-risk feature | Relevant `.work/features/<slug>/*-SPEC.md` |
| Unmapped app surfaces / registry gaps | `@plan-verify coverage` |

---

## Open owner actions

| # | Action | Blocks | Owner |
|---|--------|--------|-------|
| - | (none) | | |

---

## What this cycle produced (audit history - skim last session only)

| Date | Session | Artifacts |
|------|---------|-----------|
| 2026-05-27 | Coverage/registry parity | `plan-verify` coverage mode; `plan-repair` from coverage; FEATURE_STANDARD §14; DIRECTORY_MAP gate; templates/work/reports |
| YYYY-MM-DD | | |

---

## Explicit unknowns (promoted from UNKNOWNS)

| ID | Summary | Blocks |
|----|---------|--------|
| - | | |

---

## Cross-LLM verification

- **Triggered:** no
- **Result:** -
- **Notes:** -
