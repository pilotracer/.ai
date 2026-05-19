# Testing and test suite — role in the workflow

**Doc type:** Reference (portable) + **verification** that tests are first-class.  
**Conclusion (evidence):** Yes — **unit, suite, lint, and type** are mandatory in the **`code-implementation`** task gate and **complete** protocol; **`code-verify milestone`** matrix includes **FR coverage** and **test coverage** rows. Feature **SPECs** require a **Test plan** section.

---

## 1. Where tests appear in skills

| Location | What it enforces |
|----------|------------------|
| `code-implementation` — **Valid iteration block** | Validation steps must include ≥1 **runnable** test command. |
| `code-implementation` — **Task gate** | Per-task: context `pytest` (or equivalent), **full suite** smoke, lint, typecheck, secrets, protected files, scope. |
| `code-verify` — **Milestone matrix** | “Every FR linked to milestone has ≥1 test?”; “Happy path + key error paths have tests?” |
| `code-implementation` — **Complete protocol** | CO2 `@code-verify milestone` first; CO1 re-runs suite only if CO2 shared gates did not already pass on current tree. |
| `plan-master` (traceability rule) | Goal → requirement → **task → validation/test** → acceptance (stated in skill frontmatter). |

Paths in skills may show `docker compose exec … pytest` — **swap** for your stack; the **structure** remains.

---

## 2. Where tests appear in standards

| Artifact | Section |
|----------|---------|
| `FEATURE_STANDARD` | **§12 Test plan** in SPEC template; **§6 Observability hooks** tie metrics to behaviour. |
| `CONVENTIONS` | Test folder layout (`unit/`, `contract/`, `integration/`, `e2e/`), tooling (`pytest`, `hypothesis`, …). |
| `{ITERATION_CARRIER}` | **Validation steps** checklist mirrors what CI / humans run. |

---

## 3. Pyramid vs velocity

- **Task gate** optimizes **fast inner loop** (context tests + quick full suite).  
- **CI** (when configured) should run the same commands as validation steps.  
- **Sandbox / e2e** may be `-m 'not sandbox'` excluded locally — still must be documented.

---

## 4. Gaps to watch (honest)

| Gap | Mitigation |
|-----|------------|
| Type-checker drift between skill commands and project standard | Single source: align task-gate commands with **CONVENTIONS** and `.cursorrules` `REPLACE:TYPECHECK_COMMAND` in one PR. |
| Frontend tests not in same skill file | Add parallel task gate rows for `dashboard/` or split skill per surface. |
| “FR has test” unverifiable without coverage tool | Add coverage threshold in CI later; verify row stays **gap** until then. |

---

## 5. Interactive workflow summary

```text
SPEC §12 ──► plan-master (validation refs) ──► NEXT validation steps
                    │                                    │
                    └──────────────► code-implementation task gate (pytest)
                                           │
                                           └── code-verify milestone (FR + test coverage rows)
```

---

## 6. Tutorials for humans

- [Verify / audit / test in dev](20260518-tutorial-verify-audit-test-development.md)  
- [Request new test](20260518-tutorial-request-new-test.md)  
- [Request tests for feature/module](20260518-tutorial-request-test-feature-module.md)
