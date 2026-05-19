# plan-master — reference

Supplement to `skill.md`. Invocation examples, AC Billing paths, and edge cases.

---

## Invocation examples

| Action | Prompt |
|--------|--------|
| Status | `@plan-master` **status** |
| Continue | `plan-master` **continue** |
| New plan | `@plan-master` **greenfield** |
| Integrity only | `plan-master` **integrity** |
| Revise | `plan-master` **revise** — add fiscal sandbox milestone |
| With mode | `plan-master greenfield` — **enterprise** |
| Look up a task | `plan-master` **task** M1-T3 |
| All tasks for a milestone | `plan-master` **task** M4 |

### Cursor

```
@plan-master status
@plan-master continue
@plan-master greenfield — ai-native
Read .ai/skills/plan-master/skill.md and run integrity mode.
```

### Claude Code / opencode / Codex

```
Follow .ai/skills/plan-master/skill.md — status. Read-only.
Follow .ai/skills/plan-master/skill.md — continue. Update `{PLANS_ROOT}/full/YYYYMMDD-full-plan.md`.
```

---

## Mode comparison

| | status | continue | greenfield | integrity | revise |
|---|--------|----------|------------|-----------|--------|
| Read plan | yes | yes | create/update | yes | yes |
| Write plan | no | yes | yes | optional fixes | yes |
| Update NEXT | no | if Approved | if Approved | no | if Approved |
| User questionnaires | no | per phase | per phase | no | targeted |
| P5 integrity | report | on phase complete | before Approved | full | delta |

---

## AC Billing System — foundation inputs

When planning for this repo, **prefer cite over rewrite**:

| Topic | Primary evidence |
|-------|------------------|
| Scope | `{PLANS_ROOT}/foundation/20260517-01-*.md` |
| Hacienda integration | `20260517-02-*.md`, `.ai/docs/integration/MANIFEST.txt` |
| ERP lanes | `20260517-03-*.md` |
| Architecture | `20260517-04-foundation-architecture.md` |
| Stack | `DOCS_TECH_STACK.md` |
| Layout | `.ai/standards/20260517-DIRECTORY_MAP.md` |
| Fiscal | `{FEATURE_SPEC_ROOT}/fiscal-pipeline/20260517-SPEC.md` |
| Commercial / MD | `commercial-documents`, `master-data` SPECs |
| Interaction | ADR 012 + archived `decision_012_*.md` (read-only) |
| Threats / data | `threat-model`, `data-classification` |
| Local infra | `docker-compose.yml`, `20260517-docker-compose-proposal.md` |
| Personas | `{PLANS_ROOT}/20260517-personas-v1.md` |

**Suggested milestone order (v1 — validate in Phase 5):**

1. M1 — `apis/` platform skeleton (health, settings, migration runner)
2. M2 — Synthetic fixtures F1–F6
3. M3 — `master_data` + `commercial` API stubs
4. M4 — Fiscal pipeline worker shell (no production keys)
5. M5 — Hacienda sandbox E2E (runbook steps)
6. M6 — Dashboard shell + i18n scaffold
7. M7 — Counter profile vertical slice (ADR 012)

Adjust after integrity review; do not treat this list as authoritative without repo evidence.

---

## Traceability matrix (minimal example)

Task IDs use the globally unique **`M{N}-T{N}`** format. Shorthand `T{N}` is acceptable only when the milestone context is explicit.

| Goal | FR/NFR | Component | Task ID | Description | Test | Acceptance |
|------|--------|-----------|---------|-------------|------|------------|
| Issue valid FE | FR-12 | `fiscal` | M4-T1 | implement state machine R1–R5 | `test_fiscal_sm_*.py` | Sandbox 201 + Accepted |
| Multi-tenant isolation | NFR-03 | `platform` | M3-T11 | tenant middleware | `test_tenant_isolation` | Cross-tenant read fails |

---

## YAML input example (AC Billing)

```yaml
project: AC Billing System
description: Multi-business invoicing with Costa Rica Hacienda electronic documents
requirements:
  - Multi-tenant SaaS
  - FE issuance v4.4
  - Counter + desk profiles (ADR 012)
  - EN/ES/ZH/RU UI
non_functional:
  - Fiscal correctness non-negotiable
  - Tenant crypto isolation
foundation_docs:
  - {HANDOFF}
  - {PLANS_ROOT}/foundation/20260517-04-foundation-architecture.md
  - DOCS_TECH_STACK.md
constraints:
  - Docker-only dev per .cursorrules
  - No secrets in repo
target_users:
  - Counter clerk, desk user, business owner
additional_notes:
  - CPA ADRs 009/010 open; do not block M1 skeleton
advanced_mode: enterprise
```

---

## Integration with session-control

| Session event | plan-master action |
|---------------|------------------|
| **start** | Optional: `plan-master status` if master plan exists |
| **close** | If plan phase completed, note in HANDOFF artifact table |
| Planning-only session | No commit unless `close commit` |

---

## Integration with plan-foundation

```
plan-foundation (P0–P6)  →  plan-master-ready
        ↓
Approved master plan (`*-full-plan.md`)
        ↓
@plan-master status        →  implementation-ready
        ↓
feature SPECs + code     →  per FEATURE_STANDARD / code-implementation
```

Run `plan-foundation status` before `plan-master greenfield` on mature repos.

---

## Edge cases

| Situation | Behavior |
|-----------|----------|
| No `*-full-plan.md` | **continue** → suggest **greenfield** |
| Foundation not ready | Draft plan; waivers in Assumptions; list blockers |
| Plan contradicts ADR | **fail** P5; do not Approve until ADR amended |
| User wants code in same message | Stop plan mode; switch to implementation with SPEC refs |
| Huge traceability matrix | Split to `*-full-plan-trace.md` |
| Only `.ai/` changed | Commit type `docs` on close |

---

## Optional companion skills (future)

| Skill | Purpose |
|-------|---------|
| `plan-foundation` | Domain/foundation (exists) |
| `session-control` | Session bookends (exists) |
| `integrity-review` | Standalone P5 deep dive |
| `execution-orchestrator` | Task batching for agents |

plan-master **includes** P5/P6; companions are optional splits if skills grow too large.

---

## Wrong prompts

| Prompt | Problem | Use instead |
|--------|---------|-------------|
| `plan-master` write apis code | Wrong skill | Implementation + SPEC |
| Skip foundation on AC Billing | Reinvents ADRs | Read `{PLANS_ROOT}/foundation/` + ADRs |
| Approve with open U1 fiscal | Unsafe | Waivers explicit or resolve |
| `greenfield` during status request | Mode violation | `status` only |

---

## Optional slash commands (team convention)

| Command | Maps to |
|---------|---------|
| `/fp status` | status |
| `/fp continue` | continue |
| `/fp greenfield` | greenfield |
| `/fp integrity` | integrity |

Document in project README if adopted.
