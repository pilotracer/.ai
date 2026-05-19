# `.ai/` — agnostic process layer

**Purpose:** Portable skills, standards, concepts, workflow guides, and integration mirrors — **reusable across projects**. Not project plans, SPECs, or session handoff.

> **First time here, or lost?** Read [`START_HERE.md`](START_HERE.md) — the operator decision tree.  
> **Process questions?** Read [`PROCESS_ROUTER.md`](PROCESS_ROUTER.md) or run `@process-router — <question>`.  
> **DeepWiki:** [deepwiki.com/PiloTracer/.ai](https://deepwiki.com/PiloTracer/.ai)

**Project working artifacts** (plans, SPECs, ADRs, prompts, HANDOFF) live under **[`.work/`](../.work/README.md)**.

Do **not** treat `.claude/` as authoritative.

---

## Layout (`.ai/` only)

| Path | Contents |
|------|----------|
| `.ai/START_HERE.md` | Operator decision tree — read first when lost |
| `.ai/PROCESS_ROUTER.md` | Human guide for `@process-router` skill |
| `.ai/skills/` | Portable agent skills — [registry](skills/README.md) |
| `.ai/standards/` | CONVENTIONS, FEATURE_STANDARD, DIRECTORY_MAP, observability, threat model, … |
| `.ai/concepts/` | Architecture NFR concept pack (MOD-01…MOD-06) |
| `.ai/docs/guides/workflows/` | Portable workflow tutorials and reference guides |
| `.ai/docs/integration/` | Vendor integration artifacts + `MANIFEST.txt` |
| `.ai/plans/README.md` | **Pointer** → `.work/plans/` |
| `.ai/features/README.md` | **Pointer** → `.work/features/` |
| `.ai/prompts/README.md` | **Pointer** → `.work/prompts/` (questionnaires; optional user scratch — not read by skills) |
| `.ai/context/README.md` | **Pointer** → `.work/context/HANDOFF.md` |
| `.ai/decisions/README.md` | **Pointer** → `.work/decisions/` |

---

## Project working tree (`.work/`)

| Path | Contents |
|------|----------|
| `.work/plans/` | Foundation, master plan, registries, `NEXT.md`, operations |
| `.work/features/` | Feature SPECs and amendments |
| `.work/decisions/` | ADRs |
| `.work/prompts/` | Decision questionnaires, archives; optional user scratch (`initial.md` — not read by skills) |
| `.work/context/` | `HANDOFF.md` (session-control) |

See [`.work/README.md`](../.work/README.md) for placeholder map.

---

## Required reads (humans + agents)

1. **`.ai/START_HERE.md`**
2. `.cursorrules`
3. `.work/context/HANDOFF.md`
4. `.work/plans/NEXT.md`
5. **If planning context needed:** `.work/plans/foundation/*-01-*-initial-scope.md` (P0 mini-plan). **Not** `.work/prompts/initial.md` (user scratch — skills ignore unless user requests).
6. For code: `.ai/standards/20260517-CONVENTIONS.md`, `20260517-FEATURE_STANDARD.md`, `20260517-DIRECTORY_MAP.md`

Generated **project** markdown under `.work/` uses the `YYYYMMDD-` filename prefix unless documented otherwise.

## Agent rules file

This repository's **only** agent-rules file is `.cursorrules`. Do **not** create `AGENTS.md` without explicit owner approval.
