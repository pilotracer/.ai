# plan-foundation - reference

Supplement to `skill.md`. **How to invoke**, **verify**, and **certify plan-master-ready**.

**Anti-drift reminder:** `{PLANS_ROOT}/foundation/` (docs 01–04) = **foundation inputs**. `*-full-plan.md` = **master plan** (plan-master skill only). Never call doc 04 "the full plan" - say **architecture foundation** or **foundation doc 04**. Doc 01 heading: `Architecture directions (non-prescriptive - architecture foundation in doc 04)`. See [Terminology](skill.md#terminology-required--prevents-confusion-with-plan-master).

---

## Readiness states (do not confuse)

| State | Question | Who certifies | Unlocks |
|-------|----------|---------------|---------|
| **foundation-complete** | Do P0–P6 artifacts and gates pass? | plan-foundation **status** | Continue fixing foundation |
| **plan-master-ready** | Is foundation semantically mature enough for a master plan? | plan-foundation **certify** + `plan-master integrity` | `@plan-master greenfield` / `continue` |
| **implementation-ready** | Is the Approved master plan safe to execute broadly? | **plan-master** **status** only | Multi-milestone implementation |

```text
P0–P6 → foundation-complete → plan-master-ready → plan-master → Approved master plan → implementation-ready → code
     └─ plan-foundation ─────────────┘              └─ plan-master ─────────────────────────────┘
```

---

## How to invoke (quick reference)

| Goal | Prompt (Cursor) | Mode |
|------|-----------------|------|
| Snapshot (foundation only) | `@plan-foundation` status | status |
| Ready for master plan? | `@plan-foundation` status - plan-master-ready? | status |
| Certify for plan-master | `@plan-foundation` certify plan-master-ready | certify |
| Resume incomplete gate | `@plan-foundation` continue | continue |
| New project from scratch | `@plan-foundation` greenfield - \<idea\> | greenfield |
| Ready to code? (broad) | `@plan-master` status - implementation-ready? | **plan-master** (not plan-foundation) |

**Explicit file path (any agent):**

```
Read .ai/skills/plan-foundation/skill.md - run status mode. Read-only.
```

```
Read .ai/skills/plan-foundation/skill.md - run certify mode. Update HANDOFF if plan-master-ready passes.
```

### Claude Code / opencode / Codex

```
Follow .ai/skills/plan-foundation/skill.md in status mode.
Report foundation-complete and plan-master-ready with evidence. Redirect implementation-ready questions to plan-master.
Do not create files unless certify mode and HANDOFF update requested.
```

```
Follow .ai/skills/plan-foundation/skill.md in certify mode.
Run plan-master integrity on foundation artifacts, then evaluate S4 criteria.
```

---

## Verification playbook

### Step 1 - Foundation status (always start here)

```
@plan-foundation status
```

**Agent must:**

1. Read `skill.md` Step 0 → **status**
2. Read S2 artifacts: HANDOFF, NEXT, foundation doc 04, registries, `*-full-plan.md` if present
3. Evaluate P0–P6 per gate completion model (not file-exists-only)
4. Report: **foundation-complete**, **plan-master-ready** only (if user asked implementation-ready → redirect to plan-master)

**Pass:** Report lists evidence paths per phase.  
**Fail:** Missing reads or only glob counts without integrity.

---

### Step 2 - Semantic integrity (before plan-master-ready)

When foundation-complete is **yes** but plan-master-ready is **not evaluated** or **no**:

```
@plan-master integrity
```

Or inline during certify:

```
@plan-foundation certify plan-master-ready
```

(plan-foundation orchestrates; plan-master supplies integrity rules.)

**Pass:** `plan-master integrity` → **pass** or **pass with waivers** in HANDOFF.  
**Fail:** Contradictions ADR ↔ SPEC ↔ foundation doc 04 → fix via `@plan-foundation continue`.

---

### Step 3 - Certify plan-master-ready

```
@plan-foundation certify plan-master-ready
```

**Agent must:**

1. Confirm all [S4 criteria](skill.md#s4--plan-master-readiness) (10 rows) with pass/fail + evidence
2. Require `plan-master integrity` result on **foundation** artifacts
3. On **yes**: write `Plan-master-ready: <date>` in HANDOFF §Repository state
4. Recommend next: `@plan-master greenfield` (no master plan yet) or `@plan-master continue` (draft exists)

**Do not** run `@plan-master greenfield` if certification is **no**.

---

### Step 4 - Master implementation plan (plan-master skill)

After **plan-master-ready: yes**:

```
@plan-master greenfield
```

or

```
@plan-master continue
```

**Produces:** `{PLANS_ROOT}/full/YYYYMMDD-full-plan.md` ([master plan artifact](skill.md#master-plan-artifact))

**Pass:** Plan file exists with 25 mandatory sections; registries linked, not duplicated.  
**Not yet implementation-ready** until Status: **Approved**.

---

### Step 5 - Implementation-ready check (plan-master skill - not plan-foundation)

After master plan exists:

```
@plan-master status - implementation-ready?
```

**Agent must:** Evaluate master plan Approved + validation gates (plan-master skill). plan-foundation does **not** score this.

**Pass:** plan-master reports implementation-ready **yes** with evidence.

---

## Example - post-foundation sequence

| Check | Typical next step |
|-------|-------------------|
| foundation-complete | **no** → `@plan-foundation greenfield` or `continue` |
| plan-master-ready | **no** → `@plan-foundation certify plan-master-ready` |
| Master plan missing | `@plan-master greenfield` |
| implementation-ready | `@plan-master status` (authoritative) |

```
1. @plan-foundation status
2. @plan-foundation certify plan-master-ready
3. @plan-master greenfield
4. (owner reviews) → master plan Approved
5. @plan-master status
6. @session-control start → @code-implementation plan - M1
```

---

## Mode comparison

| Mode | Writes files | Runs plan-master integrity | Updates HANDOFF |
|------|--------------|--------------------------|-----------------|
| status | no | reports last result | no |
| certify | HANDOFF only if pass | **yes** (required) | yes on pass |
| continue | yes (artifacts) | at P3+ gates | yes on gate |
| greenfield | yes (P0→P6) | at P3+ gates | yes on gate |

---

## Integration with plan-master

| plan-foundation event | plan-master action |
|----------------------|------------------|
| GATE p3, p4, p5 | integrity subset |
| **certify** / GATE p6 | **integrity** on foundation (required) |
| plan-master-ready **yes** | **greenfield** or **continue** |
| Master plan Approved | **plan-master** **status** → implementation-ready |
| ADR ↔ SPEC contradiction | **integrity** |

---

## Planning registry templates

Create at **P0** (empty) or use seeded files in mature repos.

### ASSUMPTIONS.md

```markdown
# ASSUMPTIONS - planning registry
**Updated:** YYYY-MM-DD

| ID | Assumption | Label | Source | Notes |
|----|------------|-------|--------|-------|
| A1 | … | Confirmed \| Inference \| Unverified | path/ADR | |

## Rejected
| ID | Assumption | Reason |
```

### RISK_REGISTRY.md

```markdown
# RISK_REGISTRY - planning registry
| ID | Risk | Category | Likelihood | Impact | Mitigation | Status | Owner |
```

### UNKNOWNS.md

```markdown
# UNKNOWNS - planning registry
| ID | Question / blocker | Blocks | Owner | Status |
```

---

## P0 initial scope (product-intent capture)

**Owner skill:** `@plan-foundation` **greenfield** (Phase 0 - Capture). There is **no** `code-foundation` skill.

**Canonical path:** `{PLANS_ROOT}/foundation/YYYYMMDD-01-<project-slug>-initial-scope.md` (foundation doc 01).

**Not the seed:** `{PROMPTS_ROOT}/initial.md` - user-owned scratch; skills **must not** read or create unless the user explicitly names that path.

### Greenfield creates (minimum)

```markdown
# <Project Name> - Initial exploration and scope

## Audience and document purpose
…

## Assumption ledger

### Founder intent (verbatim - P0 capture)
<paste raw product idea from user>

### Confirmed facts (repository evidence)
…

## Architecture directions (non-prescriptive - architecture foundation in doc 04)
…
```

P1 expands doc 01; P1 also produces docs 02–04. Doc 01 is the **mini/preliminary plan** - not a prompt file.

---

## Glob patterns (artifact detection only)

Use for **foundation-complete** artifact presence - **not** for plan-master-ready (requires semantic checks).

| Phase | Glob | Min count |
|-------|------|-----------|
| P1 scope | `{PLANS_ROOT}/foundation/*-01-*.md` | 1 |
| P1 architecture | `{PLANS_ROOT}/foundation/*-04-*.md` | 1 |
| P2 ADRs | `{DECISIONS_ROOT}/20*.md` | ≥4 excluding README |
| P3 conventions | `.ai/standards/*CONVENTIONS*.md` | 1 |
| P3 features | `{FEATURE_SPEC_ROOT}/*/20*-SPEC.md` | ≥1 |
| P4 stack | `REPLACE:TECH_STACK_DOC` | 1 |
| P5 compose | `docker-compose.yml` OR `*docker-compose-proposal*` | 1 |
| P6 ops | `README.md`, `HANDOFF.md`, `NEXT.md` | 3 |
| Registries | `ASSUMPTIONS.md`, `RISK_REGISTRY.md`, `UNKNOWNS.md` | 3 |
| Master plan | `{PLANS_ROOT}/full/*-full-plan.md` | 0 until plan-master runs |

---

## Common blockers

| Blocker | Affects | Where |
|---------|---------|-------|
| Gate passed on file exists only | plan-master-ready | Re-run gate + shared integrity |
| No `plan-master integrity` run | plan-master-ready | `@plan-master integrity` |
| Master plan missing | implementation-ready | `@plan-master greenfield` |
| Master plan Draft | implementation-ready | Owner review → Approved |
| Open compliance / legal ADR | implementation-ready only (optional waiver for M1) | HANDOFF, UNKNOWNS |
| Docker not approved | foundation P5 | continue P5 |

---

## Wrong prompts

| Prompt | Problem | Use instead |
|--------|---------|-------------|
| `@plan-master greenfield` before certify | Master plan on weak foundation | `@plan-foundation certify` first |
| "@plan-foundation implementation-ready?" | Wrong skill | `@plan-foundation status` then `@plan-master status` |
| "foundation status" + write domain SPEC | Mixed modes | status, then continue |
| "start foundation" with full HANDOFF | Re-runs P0 | continue |
| Glob-only check | False pass | status + certify |

---

## Optional slash commands

| Command | Mode |
|---------|------|
| `/foundation status` | status |
| `/foundation certify` | certify |
| `/foundation continue` | continue |
| `/foundation start` | greenfield |

---

## Traceability quick check (P3+)

- [ ] Purpose → foundation scope (doc 01)
- [ ] ADRs referenced in SPEC
- [ ] R1… in test plan
- [ ] Row in plan-master trace matrix (when master plan exists)

---

## Gate pass vs semantic pass

| Wrong | Right |
|-------|-------|
| Files on disk → plan-master-ready | certify + S4 + integrity |
| HANDOFF complete, empty UNKNOWNS | Deferred ADRs in UNKNOWNS |
| P6 done → implementation-ready | P6 → plan-master-ready → plan-master → Approved → implementation-ready |
