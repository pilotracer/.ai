# `.ai/` — make AI-assisted development repeatable

You opened this folder because you want **less friction** with coding agents: fewer repeated explanations, fewer “where were we?” moments, and a process that stays **consistent** from session to session.

**`.ai/` is the portable process layer** — skills, standards, concepts, and workflow guides you can reuse across projects. It does **not** hold this product’s plans, SPECs, or session notes; those live in **[`.work/`](../.work/README.md)** so process stays separate from project memory.

---

## What you get

| Pain | How `.ai/` helps |
|------|------------------|
| Re-explaining the same workflow every chat | **Skills** — named playbooks (`@session-control`, `@code-implementation`, …) any agent can follow |
| Inconsistent code and docs | **Standards** — binding conventions, feature SPEC shape, directory map, observability |
| Architecture drift under AI speed | **Concept pack** (MOD-01…06) — short prompts before risky splits or fiscal work |
| Lost context between sessions | Pair with **`.work/`** — `HANDOFF.md` + `NEXT.md` bookended by `@session-control` |
| “Which doc do I read?” | **START_HERE** decision tree + **process-router** signposts |

The goal is simple: **automate the boring orchestration**, **bind the important rules**, and **keep human + agent aligned** without re-inventing process every time.

---

## How it fits together

```text
  You + agent                         Project truth
       │                                    │
       ▼                                    ▼
  .ai/  skills · standards          .work/  plans · SPECs
        concepts · guides                   HANDOFF · NEXT
       │                                    │
       └────────── same session ──────────┘
              @session-control start / close
```

- **`.ai/`** — *how* we work (portable).
- **`.work/`** — *what* this project decided and *what’s next* (not portable).
- **`.cursorrules`** — non-negotiable engineering discipline for this repo.

---

## Start in under two minutes

| I want to… | Do this |
|------------|---------|
| Orient or resume work | Read [`START_HERE.md`](START_HERE.md) |
| Open a coding session | `@session-control start` |
| See status without changing files | `@session-control status` |
| Ask “how do I…?” (read-only) | `@process-router — <question>` or [`PROCESS_ROUTER.md`](PROCESS_ROUTER.md) |
| Run the next implementation slice | `@code-implementation continue` (after `NEXT.md` names a milestone) |
| Close and hand off cleanly | `@session-control close` |

**Deeper walkthroughs:** [workflow guides](docs/guides/workflows/README.md) (tutorials + artifact matrix).

**External overview:** [deepwiki.com/PiloTracer/.ai](https://deepwiki.com/PiloTracer/.ai)

---

## What’s inside `.ai/`

### Skills — repetitive tasks, spelled out

Portable workflows in [`.ai/skills/`](skills/README.md). Invoke by folder name (e.g. `@session-control`, `@code-implementation`).

| Area | Examples |
|------|----------|
| Session hygiene | `session-control` — load context, update HANDOFF/NEXT |
| Planning | `plan-foundation`, `plan-master` — gates before big implementation |
| Delivery | `code-implementation`, `code-verify` — iteration tasks and verification |
| Quality & architecture | `feature-spec`, `concept-run`, `db-migration` |
| Environment | `dev-stack` — isolated Docker Compose helper |
| Orientation | `process-router` — routes questions to the right skill or doc |

Skills are **tool-agnostic** (Cursor, Claude Code, Codex, etc.): if the agent can read markdown, it can follow the playbook.

### Standards — consistent output

[`.ai/standards/`](standards/) — naming, layout, API style, observability, threat model, data classification. Agents and humans cite the same contracts so generated code does not fight the repo.

### Concepts — guardrails before big moves

[`.ai/concepts/`](concepts/README.md) — MOD-01…MOD-06 (coupling, latency, cost, ops load, modularity, AI amplification). Run when architecture, coupling, or AI-generated blast radius matters.

### Workflow guides — learn the system once

[`.ai/docs/guides/workflows/`](docs/guides/workflows/README.md) — tutorials (step-by-step) and guides (reference). Includes the **planning vs implementation** artifact matrix.

### Integration mirror — official vendor artifacts

[`.ai/docs/integration/`](docs/integration/) — cached Hacienda/XSD/PDF material with `MANIFEST.txt` (no secrets).

### Pointers into `.work/`

These README stubs only redirect; content lives under `.work/`:

| Stub | Project path |
|------|----------------|
| [plans/README.md](plans/README.md) | `.work/plans/` |
| [features/README.md](features/README.md) | `.work/features/` |
| [decisions/README.md](decisions/README.md) | `.work/decisions/` |
| [prompts/README.md](prompts/README.md) | `.work/prompts/` |
| [context/README.md](context/README.md) | `.work/context/HANDOFF.md` |

---

## Typical journey (this repo)

1. **Resume** — `@session-control start` → read `.work/context/HANDOFF.md` + `.work/plans/NEXT.md`.
2. **Implement** — `@code-implementation plan-iteration — M{N}` then `continue` / task gates per skill.
3. **Verify** — tests in Docker per `.cursorrules`; `@code-verify milestone` before claiming done.
4. **Close** — `@session-control close` (add `commit` / `commit push` only if you want git run for you).

Foundation and master planning use `plan-foundation` and `plan-master` when you are still shaping scope — see [skills registry](skills/README.md).

---

## Required reads (agents and humans)

When work is active, baseline context is:

1. [`.ai/START_HERE.md`](START_HERE.md)
2. `.cursorrules`
3. `.work/context/HANDOFF.md`
4. `.work/plans/NEXT.md`
5. P0 scope when present: `.work/plans/foundation/*-01-*-initial-scope.md` (not `.work/prompts/initial.md` unless you explicitly point the agent there)
6. For code changes: `.ai/standards/20260517-CONVENTIONS.md`, `20260517-FEATURE_STANDARD.md`, `20260517-DIRECTORY_MAP.md`

Project markdown under `.work/` uses the `YYYYMMDD-` filename prefix unless noted otherwise (`NEXT.md`, `HANDOFF.md`, …).

---

## Agent rules in this repository

The binding agent rules file is **`.cursorrules` only**. Do not add `AGENTS.md` here without explicit owner approval.

---

## Copy this layer to another project

1. Copy `.ai/` (skills, standards, concepts, guides).
2. Add `.cursorrules` tuned to that stack.
3. Create `.work/` with `HANDOFF.md`, `NEXT.md`, and your plans/SPECs.
4. Register skills in `.cursorrules` and teach your team `@session-control start`.

Process stays portable; product memory stays in `.work/`.
