# Code Conventions — AC Billing System

**Status:** binding once code lands. Quoted in PR reviews.
**Scope:** the Python backend (`apis/`), the Next.js dashboard (`dashboard/`), the Celery workers (`workers/`), and any shared tooling.
**Out of scope:** product/UX writing and translation guidance (see locale plan).

This file pairs with:

- `.work/plans/foundation/20260517-04-foundation-architecture.md` §5 (layout) and §11 (testing).
- `DOCS_TECH_STACK.md` (pinned versions).
- ADRs under `.work/decisions/`.

---

## 1. Language and tooling baselines

| Surface | Tooling | Notes |
|---------|---------|-------|
| Python | `ruff` (lint + format), `pyright` strict, `pytest`, `hypothesis`, `pre-commit` | All four mandatory on every PR. No `# noqa` without an inline reason. |
| TypeScript | `eslint` (config strict), `prettier`, `tsc --noEmit`, Vitest/Playwright | `any` is forbidden in source; use `unknown` + narrow. |
| SQL | Migrations via idempotent numbered SQL scripts in `apis/migrations/`; no inline DDL in code paths. | DML in seeders is allowed only under `apis/scripts/seeders/`. |
| Shell | POSIX `sh` or `bash` with `set -euo pipefail`. No undefined-variable expansion. | Use `shellcheck` in CI. |
| Markdown | `mdformat-gfm` (or equivalent) to normalize tables. | Avoid trailing whitespace except deliberate line breaks. |

## 2. Type discipline

- Python: every function in `src/` carries full annotations. `pyright` runs in `strict` mode. `from __future__ import annotations` enabled project-wide.
- Pydantic `BaseModel` for any data crossing a process boundary (HTTP, queue, persistence boundary in/out, external API). Domain entities are plain dataclasses (`@dataclass(frozen=True, slots=True)` where appropriate). **Pydantic models are not domain entities.**
- TypeScript: `strict: true`, `noUncheckedIndexedAccess: true`, `exactOptionalPropertyTypes: true`. Return types are explicit on exported functions.

## 3. Naming

- Python modules and packages: `snake_case`. Classes: `PascalCase`. Constants: `SCREAMING_SNAKE_CASE`.
- **Platform layer (Python):** shared cross-cutting code lives in package **`acb_platform`** at `apis/src/acb_platform/` (import name `acb_platform`, not `platform`, to avoid shadowing the stdlib `platform` module). Architecture prose may still say *platform layer* for the logical bounded context; file paths and imports use `acb_platform`.
- TypeScript files: `kebab-case.ts` for utilities, `PascalCase.tsx` for React components. Hooks start with `use`.
- HTTP routes: `kebab-case`, plural nouns, no verbs: `/comprobantes`, `/master-data/items`. Document IDs in path segments are UUIDs except where the domain identifier is the `clave` (50-char Hacienda key).
- DB tables: `snake_case`, plural, prefixed by bounded context: `fiscal_documents`, `fiscal_events`, `commercial_documents`, `master_parties`. Indexes named `ix_{table}_{cols}`.
- Event types (in `fiscal_events`): `PascalCase` past-tense verb: `Drafted`, `Signed`, `Submitted`, `Received`, `Accepted`, `Rejected`, `Errored`, `ContingencyQueued`.
- ADR filenames: `YYYYMMDD-NNN-slug.md` (3-digit zero-padded).
- All other dated docs under `.ai/` (standards, plans, features): use `YYYYMMDD-` prefix per `.cursorrules` and `.ai/README.md`.

## 4. Money, tax, and decimals (fiscal modules)

- **Decimal-only.** `float` is banned in `apis/src/{fiscal,commercial,ar,reporting}/`. `ruff` rule (custom or `flake8-no-implicit-concat`-style) enforces.
- Internal monetary scale: 5 decimal digits during calculation, **2** for CRC presentation, **2** for USD/EUR presentation. Hacienda XSDs allow up to 5 in some quantity fields; respect the XSD per field.
- Rounding rule: `ROUND_HALF_EVEN` for final monetary persistence. Intermediate calculations preserve full precision until the last step.
- FX rates from `api.hacienda.go.cr/indicadores/tc/*` are stored with **5** decimal digits.
- Tax tariffs are stored as `Decimal('0.01')`-step fractions (i.e., `0.13` for 13%, never `13`).

## 5. Time

- All timestamps stored as `timestamptz` (UTC). The application never persists a naive datetime.
- For Hacienda's `fecha` field, format = `RFC 3339` with offset `-06:00` (Costa Rica has no DST). Wrapper: `acb_platform.time.cr_now()` (`apis/src/acb_platform/time.py`) returns a CR-localized aware datetime; `cr_iso(dt)` formats with the `-06:00` offset.
- UI presentation uses the user's locale via `Intl` / `babel.dates`; the application layer never string-formats time.

## 6. Errors

- Domain errors are exceptions defined per bounded context (`apis/src/{context}/errors.py`) and inherit a single base `DomainError(Exception)` with structured fields: `code`, `message`, `context`, optional `retryable: bool`.
- HTTP layer maps `DomainError` to RFC 7807 problem-detail JSON. No bare 500s for known domain conditions.
- Hacienda errors map to a single `HaciendaError(DomainError)` carrying `x_error_cause`, `validation_exception`, and `http_status`.
- Never swallow exceptions. `except Exception:` must re-raise or convert to an explicit `DomainError`.

## 7. Logging

- Structured JSON via `structlog` (Python) and `pino` (Node tooling, if used). One line per event.
- Mandatory fields on every log line: `ts` (ISO-8601 UTC), `level`, `event`, `tenant_id`, `correlation_id`. Optional: `user_id`, `clave`, `doc_id`, `duration_ms`.
- **Forbidden in any log line:** raw email, phone, tax ID, customer name, financial amount, JWT, key material, password, callback payload bodies. Per `.cursorrules` §"No PII in Logs". The repository ships a `redact_pii()` helper; CI runs a grep-based guard against high-risk substrings in code (`logger.*emisor.*nombre`, etc.).
- Error logs: include exception class and message; never the full traceback in prod unless behind a sampling gate.

## 8. Imports and dependency direction

- One-way dependency between bounded contexts (foundation plan §2). Allowed paths only:
  - `commercial → fiscal` via published event types under `apis/src/fiscal/events/`.
  - `inventory ← commercial` via published events.
  - `ar ← fiscal` via published events.
  - All contexts → `acb_platform` (platform layer; lower level).
- Code that imports across this boundary in the wrong direction fails a CI guard (`pytest`-style import-linter rule under `apis/tests/lint/`).
- No circular imports. No `__init__.py` re-exports that mask the real source.

## 9. Persistence

- Source-of-truth tables are append-only where the domain demands it (`fiscal_events`). All other tables can be mutable but carry `created_at`, `updated_at`, and `version` columns for optimistic concurrency.
- Every tenant-scoped table has `tenant_id UUID NOT NULL` (ADR 004) and a Postgres RLS policy keyed on `current_setting('app.tenant_id')::uuid`.
- Migrations are per-context (foundation plan §5). Cross-context scripts live under `apis/migrations/cross/` and require ≥2 reviewers. Scripts are idempotent — safe to re-run — see §17 for the full strategy.
- No raw SQL in application code. Use SQLAlchemy Core/ORM. Exception: read-only reporting projections may use raw SQL behind a `Read` repository, never inline in handlers.

## 10. Concurrency

- Per-`clave` serialization for fiscal state-machine writes: Postgres advisory lock on `hashtext(clave::text)` held for the duration of a single event-append + projection update. Prevents two workers from advancing the same comprobante in parallel.
- HTTP request handlers never block on cross-context I/O; long-running operations are dispatched to Celery and surfaced to the UI via a job-id + polling.
- Celery tasks are **idempotent**: every task accepts and honours an `idempotency_key`; reruns are safe.

## 11. Tests

- Layout mirrors `src/`: `apis/tests/unit/<context>/...`, `apis/tests/contract/...`, `apis/tests/integration/...`, `apis/tests/e2e/...`.
- Markers: `@pytest.mark.unit`, `@pytest.mark.contract`, `@pytest.mark.integration`, `@pytest.mark.sandbox`. CI gates on `unit` + `contract` + `integration`; `sandbox` runs on a manual trigger.
- Every PR that touches `fiscal/` or `commercial/` must add or update at least one contract test referencing the cached XSDs in `.ai/docs/integration/`.
- Fixtures are synthetic. No real cédulas, emails, names, or amounts. Helper `apis/tests/fixtures/factories.py` generates compliant synthetic data (CABYS codes drawn from a small synthetic palette, identifiers in the `00000000-` range, etc.).

## 12. Pull requests

- Title: `type: short description` per `.cursorrules` §"Commit Message Format". Body references the ADR or feature spec it implements.
- Diff size target: ≤500 lines of production code per PR (review fatigue). Larger PRs require a "why this can't be split" justification.
- Required reviewers: ≥1 for application code; ≥2 (one with the security tag) for `fiscal/`, `acb_platform` secrets/auth modules (when present), and any migration under `cross/`.
- Required CI: lint + type + unit + contract + integration + security scans + license scan.
- Required artifact in the PR description: a link to the relevant ADR(s) or feature SPEC, and a "verification" section listing what was actually run (per `.cursorrules` §"Verification & Quality Discipline").

## 13. Comments and documentation

- Per `.cursorrules`: comments explain **non-obvious intent, trade-offs, or constraints** only. No narration of what the code obviously does.
- Module docstring required on every `apis/src/` module: one sentence purpose, one paragraph context, references to the SPEC or ADR if applicable.
- Public functions/classes carry docstrings with `Args / Returns / Raises` sections.
- No AI-attribution markers, no signatures, no "generated by" lines anywhere.

## 14. Secrets, config, environments

- Never hard-code a secret. Use AWS Secrets Manager via `acb_platform` secrets helpers (when implemented under `apis/src/acb_platform/`).
- Configuration loaded by Pydantic `Settings` from environment variables; `.env.example` is the documentation, never `.env` in repo.
- Per-environment differences (`local-dev`, `ci`, `staging`, `prod`) are expressed by env-var values, not by branching code.

## 15. Performance defaults (sanity, not targets)

- Hot HTTP path: budget = 200 ms p95 server side, excluding outbound Hacienda calls.
- Catalog lookups (CABYS, exoneración): always served from the local projection, never from `api.hacienda.go.cr` in a request thread.
- Signing operation per comprobante: budget = 500 ms p95 inside the worker.
- Submission to Hacienda: budget = 2 s p95 (governed by network and Hacienda).
- Numbers tightened later from real measurements; if a number is hit, file a ticket and follow §17.

## 16. Forbidden patterns (PR auto-reject)

- `print()` in `src/`. Use `structlog`.
- `time.sleep()` in `src/` outside explicit back-off helpers. Async code uses `asyncio.sleep`.
- `requests` / `urllib3` for outbound HTTP. Use `httpx`.
- `random` for anything not cryptographic; for tokens use `secrets`.
- `pickle` for any persisted or transmitted payload.
- Floats in fiscal modules (§4).
- Cross-context imports outside the published ports (§8).

## 17. Migration strategy (idempotent SQL scripts)

**Rationale:** Alembic version chains cause conflicts when migrations diverge across environments. This project uses an **idempotent-script** pattern instead: no version table, no migration chain, scripts are safe to re-run. See `.ai/skills/db-migration/skill.md` for the canonical workflow.

**Directory:** `apis/migrations/`

**Execution order:** Scripts are prefixed with a 3-digit number (`001_`, `002_`, …) and executed in ascending order on every application startup by `apis/src/acb_platform/migration_runner.py`.

**Idempotency rules:**

| Operation | Idempotent form |
|-----------|----------------|
| Create table | `CREATE TABLE IF NOT EXISTS {table} (…)` |
| Add column | `ALTER TABLE {table} ADD COLUMN IF NOT EXISTS {col} {type}` |
| Add constraint | `ALTER TABLE {table} ADD CONSTRAINT IF NOT EXISTS` (PG 9.6+) or wrap in `DO $$ … IF NOT EXISTS … END $$` |
| Create index | `CREATE INDEX IF NOT EXISTS {name} ON {table} (…)` |
| Create trigger/function | `CREATE OR REPLACE FUNCTION {name}() …` / `CREATE OR REPLACE TRIGGER {name} …` |
| Insert data | `INSERT INTO {table} (…) VALUES (…) ON CONFLICT DO NOTHING` or `WHERE NOT EXISTS (SELECT 1 FROM {table} WHERE …)` |
| Update data | `UPDATE {table} SET … WHERE … AND …` (targeted; re-running produces the same result) |
| Create schema | `CREATE SCHEMA IF NOT EXISTS {name}` |
| Grant permission | `GRANT … TO …` with check for existing grant |

**Script naming convention:**

```
migrations/
├── 001_init.sql              ← schemas, extensions
├── 002_platform_audit_log.sql
├── 003_identity_*.sql
├── 004_master_data_*.sql
├── 005_commercial_*.sql
├── 006_fiscal_*.sql
├── 007_triggers.sql          ← all CREATE OR REPLACE TRIGGER/FUNCTION
├── 008_inserts.sql           ← idempotent reference data (CABYS codes, tax tariffs, defaults)
├── 009_constraints.sql       ← ADD CONSTRAINT IF NOT EXISTS
└── ...                       ← additional scripts appended in order
```

**Runner behaviour:**

1. On startup, `migration_runner.py` scans `apis/migrations/` for `*.sql` files.
2. Executes each in alphanumeric order within a single transaction (or per-script transactions for large DDL).
3. Logs each script execution: `{"event": "migration.run", "script": "001_init.sql", "duration_ms": 12}`.
4. If a script fails, the runner stops and the container exits with error. Fix the script, restart.
5. No version table. No "already applied" check — idempotency is enforced in the SQL.

**New scripts:** To add a migration, create a new numbered `.sql` file. Append to the ledger; never renumber or delete existing scripts. The runner picks it up on next restart.

**Tenanted migrations:** For schema-per-tenant (ADR 004), a wrapper iterates registered tenants and executes each script within the tenant's schema context (`SET search_path TO tenant_{slug}`).

**Deployment:** Production deployments run migrations as an init container before the application starts, using the same runner.

## 18. Drift and supersession

When a convention conflicts with reality, propose an ADR or a CONVENTIONS amendment in the same PR that justifies the change. The old rule stays in effect until the new ADR is merged.
