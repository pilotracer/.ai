# Tutorial — define the **NEXT sub-plan** (iteration block)

**Doc type:** Tactical planning tutorial (portable).  
**Synonym:** “Sub-plan” = **`## Current iteration`** in `{ITERATION_CARRIER}`.  
**Skill:** `@code-implementation plan - M{N}`   *(legacy alias: `plan-iteration - M{N}` — both still work; file kept for stability)*.

---

## 1. Relationship to other artifacts

```text
{MASTER_PLAN} section 19  ──copy task ids──►  ## Current iteration  (sub-plan)
        │                                              │
        └── FR/NFR ids ─────────────────────► task table FR/NFR column
```

The sub-plan **never** invents new task ids; it **mirrors** the master plan for one milestone.

---

## 2. One-minute decision

| Question | If yes |
|----------|--------|
| Is `{MASTER_PLAN}` Approved for this milestone? | Run **`@code-implementation plan - M{N}`**. |
| Is the milestone missing from the plan? | **plan-master revise** first. |
| Are you only fixing typos in an existing block? | [Fix NEXT tutorial](20260518-tutorial-next-fix.md). |

---

## 3. What `plan` (legacy `plan-iteration`) does (operator view)

1. Reads latest Approved `{MASTER_PLAN}` and milestone **M{N}**.  
2. Copies task rows → preserves **`M{N}-T{n}`** ids.  
3. Adds file paths (from plan or SPEC); flags `TBD` + **Owner blockers** when unknown.  
4. Writes **In / Out of scope** from milestone scope text.  
5. Emits **Acceptance** + **Validation** (must include runnable tests per valid-block rules).  
6. Inserts **`### Concept / NFR registry`** when your skill version requires it.

---

## 4. Manual alternative (not recommended)

If you cannot run the skill:

1. Open master plan milestone **M{N}**.  
2. Copy the task table **verbatim** into `{ITERATION_CARRIER}`.  
3. Add columns: Complexity, Status, Notes.  
4. Build acceptance from milestone **Acceptance criteria** only.  
5. Copy **Validation** commands from plan section **21** or milestone validation row.

Manual path has high **id drift** risk — prefer the skill.

---

## 5. Exit check

Pass every item in [Generate new NEXT — valid block](20260518-tutorial-next-generate-new.md) section 3.

---

## 6. Then start implementation

`@code-implementation start` — see [Multiple iterations](20260518-tutorial-implement-multiple-iterations.md).
