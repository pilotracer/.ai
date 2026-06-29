---
name: x-director
description: >-
  Cross-framework orchestration skill that receives any free-text request and
  coordinates across all frameworks available in the current workspace:
  .ai (Agent OS), .ai.ui (UI Design OS), and .ai.biz (Business OS).
  Routes to the correct director (ai-director, ui-director, biz-director) or
  coordinates multi-framework workflows. Flags when a new skill is needed
  across any framework.
---

# x-director — Cross-Framework Orchestrator

**Role:** Top-level "director of directors". You receive any free-text request and orchestrate across all frameworks in the workspace. You act as the single entry point for work spanning engineering (`.ai`), UI/design (`.ai.ui`), and business (`.ai.biz`). You do not need to know which framework owns what — just describe the goal.

## Framework registry

The x-director knows about every framework in the workspace. **Path resolution is dynamic** — never assume the framework dirs sit at a fixed absolute path. Resolve in this order:

1. **Authoritative:** `.cursorrules` § *Frameworks registry* (the file shipped to every adopter repo). If the registry names a path for `.ai.ui` / `.ai.biz`, use it.
2. **Auto-discover from `.ai` parent:** `parent="$(cd "$REPO_ROOT/.ai/.." && pwd)"`; a sibling dir at `${parent}/.ai.ui` or `${parent}/.ai.biz` is a valid framework root.
3. **Preflight:** before routing to any director, verify `<framework_root>/skills/README.md` is readable. If not → output one line `framework not installed here` and stop. Never route into the void.

| Framework | Default sibling path | Director | Role (one line only — fine-grained routing lives in each director) |
|-----------|---------------------|----------|------|
| **Agent OS** (`.ai`) | *this directory* | `@ai-director` | Engineering: planning, coding, DB migrations, dev stack, sessions |
| **UI Design OS** (`.ai.ui`) | `../.ai.ui` | `@ui-director` | UI: tokens, screen specs, components, verify |
| **Business OS** (`.ai.biz`) | `../.ai.biz` | `@biz-director` | Business: strategy, brand, pricing, content, sales pipeline, ideas |

> **Delegator rule (HARD):** x-director classifies only **which framework(s)** — never the sub-bucket. The fine-grained bucket table (`engineering-db`, `ui-design`, `business-strategy`, …) lives **inside each director's own `skill.md`**, which is the single source of truth for its domain. x-director forwards the user's verbatim request to the chosen director; the director re-classifies and owns the skill chain.

**Location:** `.ai/skills/x-director/skill.md`

---

## Hard rules

1. Never execute a skill without respecting its declared prerequisites. Always check the respective framework's `SKILL_DEPENDENCIES.md`.
2. Before any write operation, read the relevant HANDOFF file(s) for session context:
   - `.ai` work → `{HANDOFF}` (`.work/context/HANDOFF.md`)
   - `.ai.ui` work → `{HANDOFF_UI}` (`.work.ui/context/HANDOFF_UI.md`)
   - `.ai.biz` work → `.work.biz/context/HANDOFF.md`
3. After completing a workflow, always update all touched HANDOFF files with what was done, what's next, and any blockers.
4. Do not invent skills or modes not registered in the respective framework's `skills/README.md`. If a request cannot be fulfilled by existing skills, follow the "New skill protocol".
5. Route through existing directors (`@ai-director`, `@ui-director`, `@biz-director`) whenever possible — they own their domain's skill chain and gates. x-director only classifies the framework; the director classifies the sub-bucket.
6. Never duplicate a skill that exists in another framework. If in doubt, check all three skill registries.
7. **Preflight before every route:** if a target director's framework is not installed (per § Framework registry resolution), stop and say so — never route into the void.

---

## Modes

| Mode | Action |
|------|--------|
| `- <free-text request>` | Parse intent, classify framework domain(s), show Confirm gate, route to correct director(s) |
| `- <free-text request> -y` | Same as above but skip the Confirm gate (trust-mode; operator opted out) |
| `- <free-text request> --dry-run` | Render the routing plan and stop — no director invokes, no HANDOFF writes |
| `status` | Report state of all installed frameworks: bootstrap, readiness gates, active iterations; mark uninstalled frameworks |
| `help` | Display this skill's purpose, framework registry, and invocation examples |

---

## Free-text intake contract

`@x-director` is the user's single free-text entry point when they are unsure which framework owns the work or when the work spans multiple frameworks. Follow this contract so every inquiry is structured, routed, and recorded.

### 1. Capture
- Preserve the user's exact wording (quote it in every touched HANDOFF).
- Do not silently narrow a cross-framework request to a single director.

### 2. Resolve frameworks + load contexts (preflight)
- Resolve framework roots per § Framework registry, in **this exact order**: `.cursorrules` § Frameworks registry → sibling auto-discovery → preflight read of each `skills/README.md`.
- For each framework present, read its HANDOFF/NEXT file (see § Load context).
- Skip missing frameworks after emitting the one-line `framework not installed here` note so the user knows it was checked and intentionally skipped.

### 3. Classify framework domain(s) — coarse only
- Match by intent, not keyword. Use the bucket table below — **only the four coarse buckets**, never sub-buckets.
- A request is **cross-framework** if it naturally requires outputs from ≥2 frameworks (e.g., API + UI, strategy + landing page).
- A request is **single-framework** if it clearly belongs to one domain, even if the user does not name the framework.
- **Score routing confidence** (`high` | `med` | `low`) based on keyword strength (exact word vs partial vs fallback bucket). Carry this value into the record at step 6.

| Bucket | Signals | Route to |
|--------|---------|----------|
| `engineering` | backend, API, database, migration, server, code, plan, architecture, documentation, guide, tutorial, how-to, feature doc | `@ai-director - <verbatim request>` |
| `ui` | UI, frontend, design, screen, component, visual, tailwind, CSS, a11y | `@ui-director - <verbatim request>` |
| `business` | business, strategy, niche, offer, pricing, brand, community, referral, proposal, objection, deal, pipeline, discovery, validate, content, writing, idea, product | `@biz-director - <verbatim request>` |
| `cross-framework` | Naturally requires ≥2 of the above | Coordinate across directors (see § ROUTE) |
| `unsure` | Cannot classify | Ask one clarifying question (≤3 options) or route to `@ai-director - <verbatim request>` |

### 4. Channel to the right director(s)
- Single-framework: route to the chosen director with the user's verbatim request.
- Cross-framework: coordinate sequentially by dependency; update each director's HANDOFF after its part.
- Use canonical syntax: `@<director> - <free-text request>`.
- Never execute a skill directly when a director already owns the chain.

### 5. Confirm gate (before any director invoke)

Before routing, render a **routing plan** and get explicit ack. Do **not** invoke any director or write any HANDOFF before ack.

```markdown
## x-director routing plan
**Request:** "<user's verbatim request>"
**Classified framework(s):** <engineering | ui | business | cross-framework>
**Routing confidence:** high | med | low
**Preflight (installed frameworks):** .ai ✓ | .ai.ui ✓/✗ | .ai.biz ✓/✗
**Will invoke:**
1. @<director> - "<verbatim request>" → <expected outcome one-liner>
2. ...
**Non-reversible writes:** <list of HANDOFF / NEXT / SPEC / migration files about to be created or modified, or "none (dry-run)">
**Coordination order:** <which director first, why>
Reply `y` / `yes` to proceed, `n` to abort, or edit the plan above.
```

**Trust mode (`-y`):** skip the gate. **Dry-run (`--dry-run`):** render the plan, write nothing, stop.
**Confidence `low` with no user ack within the call:** do not route — ask one clarifying question instead.

### 5. Structure/format the record
After completing or changing state, update **every touched HANDOFF** with this exact shape:

```markdown
## Cross-framework action (@x-director)
**Date:** YYYY-MM-DD
**Request:** "<user's original request>"
**Frameworks involved:** .ai, .ai.ui, .ai.biz (list only those touched)
**Classified bucket(s):** <bucket-name(s)>
**Executed:**
1. @<director> - "<request>" → <result>
2. ...
**Coordination notes:** <cross-framework dependencies managed>
**Blockers:** <any unresolved items | none>
**Next recommended:** @<director> - "<next action>"
```

---

## Orchestration protocol

When user says `@x-director - <anything>`:

### 1. RESOLVE FRAMEWORKS + LOAD CONTEXT

Resolve framework roots per § Framework registry (precedence: `.cursorrules` § Frameworks registry → sibling auto-discovery → preflight read of each `skills/README.md`). Then read each **installed** framework's context files:

| Framework | Context file | Purpose |
|-----------|-------------|---------|
| `.ai` | `.work/context/HANDOFF.md` | Session state, last action, blockers |
| `.ai` | `.work/plans/NEXT.md` | Active iteration, recommended next |
| `.ai.ui` | `.work.ui/context/HANDOFF_UI.md` | UI session state, screen-spec-ready |
| `.ai.ui` | `.work.ui/plans/NEXT_UI.md` | Active UI iteration |
| `.ai.biz` | `.work.biz/context/HANDOFF.md` | Business session state, strategy-ready |
| `.ai.biz` | `.work.biz/plans/NEXT.md` | Business next action |

Skip files/frameworks that don't exist **after** emitting the `framework not installed here` note (do not fail the whole request because one framework isn't bootstrapped).

### 2. CLASSIFY INTENT (coarse framework only)

Use the **four-bucket coarse table** in § Free-text intake contract step 3. The fine-grained bucket table (`engineering-db`, `ui-design`, `business-strategy`, etc.) lives inside each director's own `skill.md` — do **not** restate it here. x-director decides **which framework(s)**; the director decides the sub-bucket.

### 3. CONFIRM GATE → ROUTE

Render the routing plan per § Confirm gate; obtain ack (or rely on `-y`/`--dry-run`). Then:

#### Single-framework requests — forward verbatim

```
@x-director - "I need to create a database migration for the users table"
  → Classify framework: engineering
  → Route: @ai-director - "I need to create a database migration for the users table"  (verbatim)

@x-director - "Design a login screen for the app"
  → Classify framework: ui
  → Route: @ui-director - "Design a login screen for the app"  (verbatim)

@x-director - "Define my business niche and target audience"
  → Classify framework: business
  → Route: @biz-director - "Define my business niche and target audience"  (verbatim)
```

#### Multi-framework (cross-framework) requests

```
@x-director - "Build a signup feature with backend API and UI"
  → Classify framework: cross-framework (engineering + ui)
  → Route:
    1. @ai-director - "Build a signup feature with backend API" (verbatim subset)
    2. After API SPEC done → @ui-director - "design and build the signup UI screen" (downstream ask)
    3. Verify both sides work together
```

**Cross-framework coordination rules:**

1. Identify dependency order between frameworks (e.g., API usually precedes UI that consumes it).
2. Route to each director sequentially respecting dependencies — each director re-classifies sub-buckets internally.
3. After each director completes its part, verify the output before proceeding.
4. Record the coordination in ALL relevant HANDOFF files with a cross-reference.
5. If the request requires simultaneous work (e.g., strategy + brand), directors can run in parallel.

**Common cross-framework patterns:**

| Request | Coordination |
|---------|-------------|
| "Build a full-stack feature" | `@ai-director` → backend (verbatim) → `@ui-director` → UI screens |
| "Create a landing page for my business" | `@biz-director` → strategy/brand (verbatim) → `@ui-director` → landing page |
| "Launch a SaaS product" | `@biz-director` → strategy/pricing → `@ai-director` → engineering → `@ui-director` → UI |
| "Fix my LinkedIn and build a portfolio site" | `@biz-director` → brand overhaul → `@ui-director` → portfolio site |
| "Build a product and its landing page" | `@biz-director` → strategy → `@ui-director` → landing page |

### 4. EXECUTE

For each director in the chain:
1. Invoke with the appropriate mode (`@<director> - <request>`).
2. Verify the director's completion gate passed before proceeding.
3. If a director reports a gap or blocker, route to the corrective skill — do not skip.
4. If routing is found wrong mid-flow (user redirects), record it under `User correction` (step 5).

### 5. RECORD

After completing the workflow (or on any meaningful state change), update ALL touched HANDOFF files:

```markdown
## Cross-framework action (@x-director)
**Date:** YYYY-MM-DD
**Request:** "<user's original request>"
**Frameworks involved:** .ai, .ai.ui, .ai.biz (list only those touched)
**Classified framework bucket(s):** <engineering | ui | business | cross-framework>
**Routing confidence:** high | med | low
**Preflight (frameworks installed):** .ai yes | .ai.ui yes/no | .ai.biz yes/no
**Executed:**
1. @<director> - "<verbatim request>" → <result>
2. ...
**User correction:** <none | "free-text of what rerouted the chain and why">
**Coordination notes:** <cross-framework dependencies managed>
**Blockers:** <any unresolved items | none>
**Next recommended:** @<director> - "<next action>"
```

**Feedback loop:** the `Routing confidence` and `User correction` fields feed `@ai-director review-routing` aggregates. Even if a routing plan aborts (user said `n` at the Confirm gate), still write a record with `Executed: aborted at confirm gate` and the correction note — this is signal that the bucket table needs tightening.

---

## New skill protocol

If the user request cannot be fulfilled by any existing skill across all frameworks:

1. **Confirm gap:** Check all three `skills/README.md` registries and the respective standards. Ensure the gap is genuine.
2. **Identify framework:** Determine which framework the new skill belongs to:
   - Engineering → `.ai/skills/` (prefix: `{domain}-{role}`)
   - UI → `.ai.ui/skills/` (prefix: `ui-{domain}-{role}`)
   - Business → `.ai.biz/skills/` (prefix: `biz-{role}`)
   - Cross-cutting → `.ai/skills/` (belongs in Agent OS)
3. **Report:** Tell the user what skill is needed, why existing skills cannot cover it, propose a name following the framework's naming protocol, and suggest which framework it belongs to.
4. **Create** the new skill following the framework's established pattern.
5. **Register** the skill in the respective `skills/README.md`, `SKILL_DEPENDENCIES.md`, and `standards` if applicable.
6. **Verify** registration consistency.

**Do not create a new skill when:**
- The request maps to an existing skill or standard in ANY framework
- The request can be handled by a probe loop, a concept prompt, or a process router query
- The request is for a domain that already has a skill but needs a new mode

---

## Prerequisites

- At least one framework directory must exist (`.ai/`, `.ai.ui/`, or `.ai.biz/`)
- Relevant HANDOFF files readable (may be empty/bootstrap state)

## Completion checklist

| # | Check |
|---|-------|
| 1 | Framework roots resolved via `.cursorrules` registry / sibling discovery / preflight read |
| 2 | Uninstalled frameworks reported with `framework not installed here` (not routed into the void) |
| 3 | User request classified to correct **coarse framework bucket(s)** only (no sub-bucketing here) |
| 4 | Confirm gate rendered and ack obtained (or `-y` / `--dry-run` honoured) |
| 5 | All relevant HANDOFF files read before execution |
| 6 | Correct director invoked with user's verbatim request for each framework domain |
| 7 | Prerequisites met for each skill in chain (gates respected) |
| 8 | Cross-framework dependencies ordered correctly |
| 9 | Blockers/gaps reported and routed (not silently skipped) |
| 10 | All touched HANDOFF files updated with action summary including `Routing confidence` + `User correction` |
| 11 | New skill registered properly (if created) |

---

## See also

- `.ai/skills/ai-director/skill.md` — Agent OS director
- `.ai.ui/skills/ui-director/skill.md` — UI Design OS director
- `.ai.biz/skills/biz-director/skill.md` — Business OS director
- `.ai/skills/README.md` — Agent OS skill registry
- `.ai.ui/skills/README.md` — UI Design OS skill registry
- `.ai.biz/skills/README.md` — Business OS skill registry
- `.ai/skills/SKILL_DEPENDENCIES.md` — Agent OS gate/dependency matrix
- `.ai.ui/skills/SKILL_DEPENDENCIES.md` — UI gate/dependency matrix
- `.ai.biz/skills/SKILL_DEPENDENCIES.md` — Business gate/dependency matrix
- `.ai.ui/COHABITATION.md` — Agent OS + UI Design OS coexistence rules
