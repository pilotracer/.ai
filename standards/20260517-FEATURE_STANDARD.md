# Feature Standard — AC Billing System

**Status:** binding once code lands.
**Scope:** every code-shaped change large enough to be called a "feature". Bug fixes follow `.cursorrules` §"Verification & Quality Discipline" and do not need this lifecycle.

Pairs with `.ai/standards/20260517-CONVENTIONS.md` (code-shape rules) and the foundation plan §11/§12 (tests/CI).

---

## 1. What counts as a "feature"

A feature is any unit of work that:

- Adds, modifies, or removes a user-visible capability, **or**
- Changes a domain invariant, **or**
- Adds a new bounded-context module, public port, or persisted table, **or**
- Modifies fiscal-pipeline behaviour (signing, submission, callback, polling, contingency).

Anything else is a maintenance change.

## 2. Lifecycle

```
Proposal → Spec → Implementation → Release notes → Archive
   (1)      (2)         (3)             (4)           (5)
```

| Stage | Required artifact | Gate to next stage |
|-------|------------------|--------------------|
| 1. Proposal | One-paragraph problem statement in a Linear/Jira/GitHub issue or under `.work/plans/proposals/YYYYMMDD-<slug>.md` if the team is operating without a tracker yet. | Tech lead acks scope. |
| 2. Spec | `.work/features/<feature-slug>/YYYYMMDD-SPEC.md` (see §3 template). | Spec reviewed and merged by ≥1 reviewer (≥2 if fiscal-impacting). |
| 3. Implementation | Code, tests, migrations, observability hooks. | PRs reference SPEC and pass `.cursorrules` §"Verification & Quality Discipline". |
| 4. Release notes | Append to `.work/features/<feature-slug>/CHANGELOG.md`. Update `HANDOFF.md` if the feature changes session-relevant context. | Deployed to `staging`, then `prod`. |
| 5. Archive | The SPEC stays as-is; new amendments go in `YYYYMMDD-SPEC-amendment-NN.md` siblings. SPECs are never edited after merge. | Feature is in `prod`. |

## 3. SPEC template (mandatory sections)

Place at `.work/features/<feature-slug>/YYYYMMDD-SPEC.md`. Use the following H2 headings exactly; reviewers will check for them.

```markdown
# <Feature Name> — Feature SPEC

**Status:** Draft | Approved | Implemented | Superseded by <SPEC id>
**Owner:** <person/role>
**ADRs referenced:** <list>
**Related plans:** <list>

## 1. Purpose
Why this feature exists. One paragraph max.

## 2. In scope / Out of scope
Two bullet lists. Out of scope must be explicit.

## 3. Domain language
New or refined terms introduced by this feature (table: term → definition → owner).

## 4. Behavioural spec
The rules of the feature, as plain English or pseudocode. Diagrams welcome.
Every rule numbered (R1, R2, …) so tests and PRs can reference them.

## 5. Data model
Tables added/modified. Columns with types, nullability, indexes, constraints.
Migration order if multiple steps.

## 6. APIs
HTTP routes (if any) with method, path, request, response, error codes.
Event types emitted/consumed.

## 7. Invariants
What must always be true. Each invariant references the rule(s) that uphold it.

## 8. Error model
Domain exceptions, HTTP problem-detail mapping, user-facing messages.

## 9. Observability
Metrics, traces, log events. SLOs (link to observability spec when it exists).

## 10. Security and privacy
Tenant isolation impact. Data classification of any new column (link to data-classification doc).
Threat-model considerations.

## 11. i18n
Locale impact. Glossary changes. Template changes.

## 12. Test plan
Unit / contract / integration / sandbox tests required.
Fixtures introduced.

## 13. Open questions
Unknowns. Decisions deferred to follow-up ADRs.

## 14. Residual verification
What was verified, what was not.

## 15. Concept / NFR registry (architecture pack)
Which cross-cutting concept ids apply to **this** feature (repository-defined pack under the path documented in agent rules, e.g. coupling, network cost, cost model, ops load, modularity, AI blast radius). Each row: **Concept id** | **Applies (yes/no)** | **Owner** | **Evidence or N/A reason** | **Status**. If the repository has not adopted a concept pack, write **N/A — no pack** once and skip further rows.

```

Reviewers confirm **section 15** is filled (or explicitly N/A) before **Approved**.

**Grandfathering:** SPECs already **Approved** before this template change may omit section 15 until the next **amendment** or feature touch; note waiver in the iteration **Concept / NFR registry** with owner + date.

## 4. Code organization

For a feature owned by bounded context `C`:

```
apis/src/C/
  domain/              ← entities, value objects, invariants (no I/O)
  application/         ← use cases (commands, queries), depend on ports
  infrastructure/      ← adapters (db, http clients, queue, kms)
  http/                ← FastAPI routers (thin), translate to use cases
  events/              ← published event types other contexts may consume
  ports/               ← interfaces other contexts use to talk to C
  errors.py            ← domain exceptions
apis/tests/
  unit/C/
  contract/C/
  integration/C/
```

Features may not introduce new top-level folders. If a feature truly needs a new bounded context, that requires an ADR first.

## 5. Migrations

- Every schema change ships as an **idempotent SQL script** in `apis/migrations/` with a numbered prefix (`001_`, `002_`, …).
- Scripts must be safe to re-run: use `CREATE TABLE IF NOT EXISTS`, `ALTER TABLE … ADD COLUMN IF NOT EXISTS`, `CREATE OR REPLACE FUNCTION/TRIGGER`, `INSERT … ON CONFLICT DO NOTHING`.
- Naming: `apis/migrations/{NNN}_{context}_{slug}.sql` (e.g. `apis/migrations/004_commercial_document_lines.sql`).
- A script that adds a tenant-scoped table must include the `tenant_id UUID NOT NULL` column and the corresponding RLS policy.
- Backfills are PROHIBITED per `.cursorrules` §"Backfills". Data seeding is idempotent — use `ON CONFLICT DO NOTHING` or `WHERE NOT EXISTS`. Operator-run data scripts live under `apis/scripts/data/`.

## 6. Observability hooks

Every feature must emit at minimum:

- One metric of the form `acb_<context>_<verb>_total{tenant_id, result}` (Counter).
- One latency histogram `acb_<context>_<verb>_seconds{tenant_id}` for any operation that crosses the network or the queue.
- One trace span per use case, named `<context>.<usecase>`.
- One structured log event on each success/failure.

The SPEC §9 lists the exact names.

## 7. Definition of Done (DoD)

A feature is **Done** only when all of the following are true (per `.cursorrules` §"Completion Gate"):

1. SPEC is approved and merged.
2. All §4 code locations populated as needed.
3. All §6 observability hooks emit data.
4. Unit + contract + integration tests added and green; sandbox tests pass on manual trigger.
5. Migration applied to `local-dev`, `ci`, and `staging` databases without error.
6. Documentation (SPEC, CHANGELOG, and any user-facing docs) merged.
7. ADRs referenced are in `Decided` state; deferred items are explicitly listed in SPEC §13.
8. The PR description's "verification" block lists exactly what was run (commands and outputs), per `.cursorrules`.

## 8. Naming examples

Good:

- Feature: `fiscal-pipeline`. Slug: `fiscal-pipeline`. Spec: `.work/features/fiscal-pipeline/20260517-SPEC.md`.
- Feature: `cabys-sync`. Slug: `cabys-sync`. Spec: `.work/features/cabys-sync/20260601-SPEC.md`.

Bad:

- `.work/features/feature1/` — uninformative slug.
- `.work/features/fiscal/` — collides with the bounded-context name; use a verb or noun phrase.

## 9. Anti-patterns

- A feature that has no SPEC. Block the PR.
- A SPEC merged after the code. Block the PR.
- A SPEC that contradicts an existing ADR without superseding it. Block the PR.
- Cross-context imports outside published ports. Block the PR (CI rule, per CONVENTIONS §8).
- New top-level folders without an ADR. Block the PR.
- "Generated by" or AI-attribution markers in any artifact. Block the PR.
