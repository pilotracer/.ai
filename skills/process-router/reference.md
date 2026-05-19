# process-router — routing index

Update this table when registering a new skill or guide. Keep **Run next** commands copy-pasteable.

---

## Routing table

| Bucket | Example questions | Run next | Canonical sources |
|--------|-------------------|----------|-------------------|
| **orient** | "Where am I?", "What's next?", "Lost" | `@session-control status` then `.work/context/HANDOFF.md` → `.work/plans/NEXT.md` | `.ai/START_HERE.md` §2 · [`PROCESS_ROUTER.md`](../PROCESS_ROUTER.md) |
| **session** | "Start/close session", "Bookend work" | `@session-control start` · `@session-control close` | `.ai/skills/session-control/skill.md` |
| **plan-foundation** | "Is foundation done?", "New project greenfield" | `@plan-foundation status` · `@plan-foundation greenfield` | `.ai/skills/plan-foundation/skill.md` · `20260518-guide-plan-foundation.md` |
| **plan-master** | "Master plan", "Implementation-ready?" | `@plan-master status` · `@plan-master greenfield` | `.ai/skills/plan-master/skill.md` · `20260518-guide-plan-master-full.md` |
| **implement** | "Start coding M1", "Next task", "Verify iteration" | `@code-implementation status` → `start` / `continue` / `complete`; `@code-verify` for audits | `.ai/skills/code-implementation/skill.md` · `.ai/skills/code-verify/skill.md` |
| **iteration-block** | "Fix broken NEXT", "New iteration for M2" | `@code-implementation plan-iteration — M{N}` | `20260518-tutorial-next-sub-plan-iteration.md` · `20260518-tutorial-next-fix.md` |
| **spec** | "New feature SPEC", "Review SPEC", "Amend SPEC" | `@feature-spec create — <slug>` · `@feature-spec review — <path>` | `.ai/skills/feature-spec/skill.md` · `20260517-FEATURE_STANDARD.md` §3 |
| **concept** | "Which MOD prompt?", "Run coupling audit" | `@concept-run list` · `@concept-run — MOD-06` | `.ai/skills/concept-run/skill.md` · `.ai/concepts/README.md` § Trigger table |
| **schema** | "Add table/column", "Migration status" | `@db-migration create — <desc>` · `@db-migration status` | `.ai/skills/db-migration/skill.md` |
| **stack** | "Start Docker stack", "Dev environment" | `@dev-stack` (generate/update `bin/start.sh`) · `docker compose up` | `.ai/skills/dev-stack/skill.md` · `.cursorrules` § Docker |
| **test-request** | "Add a test", "Tests for module X" | Read tutorial; then `@code-implementation continue` or new task in NEXT | `20260518-tutorial-request-new-test.md` · `20260518-tutorial-request-test-feature-module.md` |
| **verify-fail** | "Tests failed", "Lint/type failed" | `.ai/START_HERE.md` §6 · fix · re-run task gate | `code-implementation/skill.md` § Task gate |
| **conventions** | "Naming", "pyright", "PR format" | Read section only | `.ai/standards/20260517-CONVENTIONS.md` |
| **security** | "Threat model", "New column classification" | Read section only | `20260517-threat-model.md` · `20260517-data-classification.md` |
| **observability** | "Metrics/traces for feature" | SPEC §9 + standard | `20260517-observability-spec.md` · `20260518-guide-observability-traceability-in-workflow.md` |
| **fiscal** | "Hacienda integration", "Signing" | Read fiscal SPEC + integration mirror on demand | `{FEATURE_SPEC_ROOT}/fiscal-pipeline/20260517-SPEC.md` · `.ai/docs/integration/MANIFEST.txt` |
| **learn** | "Understand the system", "Reading order" | `.ai/START_HERE.md` §7 | `.ai/README.md` · `20260518-guide-workflows-index.md` |

---

## Example invocations

```text
@process-router — how do I add a database migration?
@process-router ask — which concept prompt for an AI-assisted PR?
@process-router route — I'm ready to code M1, what do I run first?
@process-router — what's the difference between plan-foundation and plan-master?
@process-router help
```

---

## Readiness states (quick reference)

```text
foundation-complete  →  plan-master-ready  →  implementation-ready
   @plan-foundation       @plan-foundation       @plan-master status
                          certify
```

Only `@plan-master status` certifies **implementation-ready**.
