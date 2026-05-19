# Tutorial — generate a new `{ITERATION_CARRIER}` (or first `## Current iteration`)

**Doc type:** Authoring tutorial (portable).  
**Prerequisites:** Approved `{MASTER_PLAN}` (or documented waiver in `{HANDOFF}`).  
**Skill:** `@code-implementation plan-iteration - M{N}` (preferred) — reads milestone from plan and writes the block into `{ITERATION_CARRIER}`.

---

## 1. What “NEXT” means in this workflow

`{ITERATION_CARRIER}` (often `NEXT.md`) is **not** the full roadmap. It holds:

- **Backlog / Done / Recommended next** (tactical queue).  
- **Exactly one** active `## Current iteration — M{N}: …` block — the **sub-plan** for the current milestone.

The **master plan** owns M1…Mn **definitions**; the carrier owns **now**.

---

## 2. Greenfield: new repository file

1. Create `{ITERATION_CARRIER}` with top matter:  
   - Title, **Updated** date.  
   - `## Done` (empty table).  
   - `## Blocked on owner` (optional).  
   - `## Recommended next` (one row: run plan-foundation / plan-master / plan-iteration as appropriate).  
2. Do **not** write `## Current iteration` by hand until the master plan exists and is Approved.  
3. Run **`@code-implementation plan-iteration - M1`** (or first milestone).  
4. Skill inserts the iteration block **after** `## Recommended next` (per `code-implementation` skill — confirm order in your skill if forked).

---

## 3. Valid `## Current iteration` block (copy this checklist)

An iteration block is **valid** when **all** are true (mirror of `code-implementation` skill):

1. Milestone ref present and traces to a **task row** in the master plan milestone section.  
2. **In scope** / **Out of scope** explicit (non-empty).  
3. At least one **task row** with at least one **file path** (or `TBD` + owner blocker).  
4. **Acceptance criteria** with at least one item.  
5. **Validation steps** include at least one **runnable** test command (adapt to your stack: Docker, `npm test`, etc.).  
6. **Concept / NFR registry** subsection present (or explicit `N/A — no pack` if your process allows).

---

## 4. After `plan-iteration` completes

1. Read the generated block as a human — fix typos only; do not renumber task ids.  
2. Set first task to `pending`, others `pending`, none `done` yet.  
3. Update `## Recommended next` top row to **`@code-implementation start`** (or your verb).  
4. Commit or session-close per team habit.

---

## 5. Starting the next milestone (new sub-plan)

When all tasks show `done`:

1. Run **`@code-implementation complete`** (updates Done, clears iteration guidance, sets Recommended next).  
2. Run **`@code-implementation plan-iteration - M{N+1}`** to append a **fresh** `## Current iteration` for the next milestone.  
3. Optionally **archive** the previous iteration block into `## Done` or a `.work/plans/archives/` snippet if your governance requires long retention on one page.

---

## 6. Agent prompt (optional)

```text
Read {MASTER_PLAN} milestone M{N}. Run code-implementation plan-iteration - M{N}.
Do not invent task ids. Preserve FR/NFR references from the plan.
```

Replace paths per [Path bootstrap](20260518-tutorial-path-bootstrap.md).
