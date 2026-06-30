# Director quick reference — free-text entry points

**When to open:** You have a goal in mind but don't know which Agent OS skill to invoke. Use a director to route for you.

---

## Which director?

| If your request is about… | Invoke |
|---------------------------|--------|
| Engineering, planning, coding, DB, dev stack | `@ai-director - <describe what you want>` |
| UI / design / frontend | `@x-director - <describe what you want>` (channels to `.ai.ui` via `@ui-director`) |
| Business / strategy / content / sales | `@x-director - <describe what you want>` (channels to `.ai.biz` via `@biz-director`) |
| Social / community / engagement | `@x-director - <describe what you want>` (channels to `.ai.soc` via `@soc-director`) |
| Spans multiple frameworks | `@x-director - <describe what you want>` (auto-resolves sibling frameworks; preflight-checks each before routing) |
| Unsure which framework | `@x-director - <describe what you want>` (classifies framework for you) |

**Flags (both directors):**
- `-y` / `--yes` — skip the Confirm gate (trust-mode).
- `--dry-run` — render the routing plan, write nothing, stop.

**Feedback:** `@ai-director review-routing` — read-only aggregate of recent `Routing confidence` / `User correction` HANDOFF entries; surfaces buckets whose signal tables need tightening.

---

## Common free-text requests

```text
@ai-director - I need to add a users table to the database
  → @db-migration status → @db-migration create - add users table

@ai-director - Start building the authentication feature
  → checks implementation-ready → @code-implementation plan - M{N} → start

@ai-director - Audit the project plan before we start coding
  → @plan-verify foundation → @plan-verify master → @plan-verify alignment

@ai-director - Fix the gaps from the plan audit
  → @plan-repair repair - from <mode>

@ai-director - I'm not sure what to build first
  → @plan-foundation probe

@x-director - Build a signup feature with backend API and UI
  → @ai-director - create backend signup API with database schema
  → @ui-director - design and build signup UI screen

@x-director - Create a landing page for my business
  → @biz-director - strategy and brand for landing page
  → @ui-director - design and build landing page

@x-director - Launch a community feature with backend and UI
  → @soc-director - community engagement strategy
  → @ai-director - build community backend API
  → @ui-director - build community UI screens
```

---

## What the director does

1. **Captures** your exact wording.
2. **Loads** `{HANDOFF}` and `{ITERATION_CARRIER}` for context. (x-director: resolves sibling frameworks from `.cursorrules` § Frameworks registry → sibling discovery → preflight each `skills/README.md`.)
3. **Classifies** intent into a coarse framework bucket (ai-director: fine `.ai` sub-bucket; x-director: framework only, never sub-buckets — directors own sub-bucketing).
4. **Confirm gate** — shows a routing plan (bucket, confidence, skills to invoke, non-reversible writes) and waits for `y`/`yes`. Skip with `-y`; render-only with `--dry-run`.
5. **Checks** prerequisite gates in `SKILL_DEPENDENCIES.md`.
6. **Invokes** the correct skill chain with canonical syntax. (x-director forwards the verbatim request to the chosen director; the director sub-classifies.)
7. **Records** the action in `{HANDOFF}` with `Routing confidence` + `User correction` fields, and updates `{ITERATION_CARRIER}` when the cycle advances.

---

## Syntax reminders

- ASCII hyphen `-` between verb and argument: `@code-implementation plan - M1`
- Free-text mode: `@ai-director - <anything>`
- Status mode: `@ai-director status`
- Help mode: `@ai-director help`

---

**Full protocol:** `skills/ai-director/skill.md` · `skills/x-director/skill.md`
