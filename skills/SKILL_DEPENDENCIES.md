# Skill dependency graph

**Purpose:** Single source of truth for **which skill may run before which**. Skills implement these rules in their own `skill.md` § Prerequisite gates; this file is the registry operators and maintainers read first.

**Readiness states** (do not conflate):

```text
project-bootstrap (scaffold)
        ↓
foundation-complete  →  plan-master-ready  →  implementation-ready
   plan-foundation        plan-foundation       plan-master status
                          certify
```

| State | Certified by | Unlocks |
|-------|--------------|---------|
| *(scaffold only)* | `@project-bootstrap init` | `@plan-foundation greenfield`, `@session-control` (minimal) |
| **foundation-complete** | `@plan-foundation status` (P0–P6 gates) | `@plan-foundation certify`, foundation **continue** |
| **plan-master-ready** | `@plan-foundation certify` | `@plan-master greenfield` / **continue** / **revise** |
| **implementation-ready** | `@plan-master status` | `@code-implementation start` / **continue** (broad); **plan-iteration** with Approved plan |

**M1 early start:** `plan-foundation` may authorize M1 skeleton when **plan-master-ready: yes** and HANDOFF/NEXT document a waiver — that is **not** implementation-ready. `code-implementation` honors HANDOFF milestone waivers per its skill.

---

## Dependency matrix

**Legend:** **Required** = stop and redirect if unmet. **Recommended** = warn, proceed only if user confirms in the same message. **—** = no gate. **Read-only** modes never mutate artifacts.

| Skill / mode | Depends on | Gate |
|--------------|------------|------|
| **project-bootstrap** `init` | Repo contains `.ai/` (or is Agent OS root) | — |
| **project-bootstrap** `status` | — | Read-only |
| **session-control** `start` | `{HANDOFF}` (offer bootstrap if missing) | Recommended: `@project-bootstrap init` |
| **session-control** `close` | Prior `start` or dirty tree | — |
| **session-control** `status` | — | Read-only |
| **plan-foundation** `greenfield` | `.cursorrules`, `{HANDOFF}` | Recommended: `@project-bootstrap init` |
| **plan-foundation** `continue` | Prior foundation work started | — |
| **plan-foundation** `certify` | **foundation-complete: yes** | **Required** |
| **plan-foundation** `status` | — | Read-only |
| **plan-master** `greenfield` | **plan-master-ready: yes** | **Required** (see exceptions below) |
| **plan-master** `continue` | **plan-master-ready: yes**; draft or partial `*-full-plan.md` | **Required** |
| **plan-master** `revise` | Existing `*-full-plan.md`; **plan-master-ready** still valid | **Required** |
| **plan-master** `integrity` | Target artifacts exist (foundation set **or** master plan for P5) | Invoked by plan-foundation certify **or** standalone |
| **plan-master** `status` / `task` | — | Read-only |
| **code-implementation** `plan-iteration` | Approved `*-full-plan.md` **or** HANDOFF M{N} waiver | **Required** |
| **code-implementation** `start` / `continue` | Valid `NEXT.md` iteration block; **implementation-ready** or HANDOFF waiver | **Required** |
| **code-implementation** `complete` | Active iteration; `@code-verify milestone` pass | **Required** |
| **code-implementation** `status` | — | Read-only |
| **code-verify** `milestone` | Valid `NEXT.md` § Current iteration | **Required** |
| **code-verify** `uncommitted` / `last` | — | — |
| **feature-spec** `create` / `review` / `amend` | FEATURE_STANDARD | — (no plan-master gate) |
| **feature-spec** before **Approved** | §15 concept registry | **Required** per FEATURE_STANDARD |
| **concept-run** `run` | Applicable trigger (SPEC §15, iteration registry, diff scope) | Per `.ai/concepts/README.md` |
| **db-migration** `create` | Task or user request; CONVENTIONS | Typically during **code-implementation** |
| **dev-stack** | User request / docker-compose present | — |
| **process-router** | — | Read-only |

---

## Exceptions and waivers

| Situation | Rule |
|-----------|------|
| **plan-master greenfield** without prior certify | **Forbidden** unless HANDOFF already records `Plan-master-ready: <date>` from a prior certify, or user supplies structured YAML with complete `foundation_docs:` paths **and** confirms foundation was completed out-of-band in the same message. |
| **plan-master reference edge case** | Do **not** draft a master plan when foundation is not ready — **stop** and list blockers (see `plan-master/skill.md` § Prerequisite gate). |
| **code-implementation** before **implementation-ready** | **Stop** unless HANDOFF explicitly waives a named milestone (e.g. M1 platform skeleton). |
| **plan-master integrity** on foundation only | Does **not** require an existing `*-full-plan.md`; plan-foundation **certify** invokes this. |
| **feature-spec** during plan-foundation P3 | Expected; SPECs need not wait for plan-master. |
| **db-migration** | Does not require plan-master; requires an implementation task or explicit user request. |

---

## Redirect cheat sheet

| User tried | Blocked because | Run next |
|------------|-----------------|----------|
| `@plan-master greenfield` | Not plan-master-ready | `@plan-foundation status` → `@plan-foundation certify` |
| `@plan-foundation certify` | Not foundation-complete | `@plan-foundation continue` |
| `@plan-foundation greenfield` | No HANDOFF / `.cursorrules` | `@project-bootstrap init` |
| `@code-implementation start` | No iteration block | `@code-implementation plan-iteration — M{N}` |
| `@code-implementation plan-iteration` | No Approved plan | `@plan-master status` → approve or waiver in HANDOFF |
| `@code-implementation start` | Not implementation-ready | `@plan-master status` or document HANDOFF waiver |
| "Ready to code?" in plan-foundation | Wrong skill | `@plan-master status` |

---

## Maintenance

When adding or changing a skill:

1. Update this matrix.
2. Add or update § **Prerequisite gate** in that skill's `skill.md`.
3. Add a row to `process-router/reference.md` if operators commonly hit the gate.
4. Do **not** duplicate normative gate text in START_HERE beyond the readiness diagram — link here or to the skill.
