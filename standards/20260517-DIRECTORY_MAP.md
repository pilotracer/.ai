# Directory Map — AC Billing System

**Status:** binding before first application code · 2026-05-17
**Source layout:** `.work/plans/foundation/20260517-04-foundation-architecture.md` §5, updated for artifacts added through 2026-05-17.

---

## Repository roots

| Path | Purpose |
|------|---------|
| `.ai/` | **Agnostic:** skills, standards, concepts, workflow guides, integration mirror, `START_HERE.md`, `PROCESS_ROUTER.md` |
| `.work/` | **Project:** plans, SPECs, ADRs, prompts, session `HANDOFF.md` — see `.work/README.md` |
| `.ai/docs/integration/` | Vendor XSD/PDF/HTML; `MANIFEST.txt` |
| `.work/plans/` | Foundation, full plan, registries, `NEXT.md`, personas, operations |
| `.work/features/<slug>/` | Feature SPECs per FEATURE_STANDARD |
| `.work/decisions/` | ADRs |
| `.work/prompts/` | Product seed, decision questionnaires |
| `.work/context/` | `HANDOFF.md` |
| `.ai/skills/` | Portable agent skills |
| `.ai/standards/` | CONVENTIONS, FEATURE_STANDARD, DIRECTORY_MAP, … |
| `apis/` | Python FastAPI backend (**planned** — not on disk yet) |
| `dashboard/` | Next.js App Router UI (**planned**) |
| `workers/` | Celery worker entrypoints (**planned**) |
| `DOCS_TECH_STACK.md` | Pinned stack versions |
| `.cursorrules` | Agent + engineering rules |
| `tmp/` | Gitignored scratch (`.gitignore`) |
| `.obfuscation/` | Sanitizer for sensitive files |
| `credentials/` | Local secrets only; **never commit** |

---

## Backend (`apis/` — planned)

```
apis/
├── pyproject.toml
├── migrations/
│   ├── 001_init.sql
│   ├── 002_identity.sql
│   ├── 003_master_data.sql
│   ├── 004_commercial.sql
│   ├── 005_fiscal.sql
│   ├── 006_triggers.sql
│   ├── 007_inserts.sql
│   └── ...                           ← numbered, idempotent, executed in order
├── src/
│   ├── main.py
│   ├── acb_platform/           ← platform layer (import acb_platform; not stdlib platform)
│   ├── identity/
│   ├── master_data/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   ├── http/
│   │   ├── events/
│   │   └── ports/
│   ├── commercial/
│   ├── fiscal/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   │   ├── hacienda/
│   │   │   ├── signing/
│   │   │   └── persistence/
│   │   └── http/
│   ├── inventory/              ← Phase A; stub optional v1
│   ├── ar/
│   └── reporting/
├── scripts/
│   ├── provision_tenant.py
│   ├── sandbox/
│   └── validate_fixtures.py
└── tests/
    ├── unit/
    ├── contract/
    ├── integration/
    ├── e2e/
    ├── lint/                     ← import boundary checks
    └── fixtures/v4_4/            ← synthetic-fixtures SPEC
```

**Dependency rule:** contexts import only `acb_platform` (platform layer) and published `ports/` / `events/` from other contexts (CONVENTIONS §8).

---

## Dashboard (`dashboard/` — planned)

```
dashboard/
├── package.json                  ← protected
├── app/                          ← App Router
├── components/
├── lib/                          ← API client, auth
├── messages/                     ← ICU: en, es, zh-Hans, ru
└── tests/
```

---

## Workers (`workers/` — planned)

```
workers/
└── fiscal_pipeline/              ← Celery app; imports apis.src.fiscal
```

---

## Documentation map (read order)

| Task | Read first |
|------|------------|
| Any code change | `.cursorrules`, `HANDOFF.md` |
| Layout | This file |
| Stack versions | `DOCS_TECH_STACK.md` |
| Fiscal work | `.work/features/fiscal-pipeline/20260517-SPEC.md` |
| Master data | `.work/features/master-data/20260517-SPEC.md` |
| Commercial / UX mode | `.work/features/commercial-documents/20260517-SPEC.md`, ADR 012 |
| Peripherals | `.work/features/peripherals/20260517-SPEC.md` |
| API design | `.ai/standards/20260517-api-style-guide.md` |
| Personas | `.work/plans/20260517-personas-v1.md` |
| Local dev (proposal) | `.work/plans/operations/20260517-docker-compose-proposal.md` |
| Hacienda integration | `.work/plans/foundation/20260517-02-*.md`, integration mirror |
| Security | `.ai/standards/20260517-threat-model.md`, `.ai/standards/20260517-data-classification.md` |
| Observability | `.ai/standards/20260517-observability-spec.md` |

---

## Gate

Doc 04 §14 item 3 (**directory map**) is satisfied by this file. Update this map when adding a new top-level directory (requires ADR per FEATURE_STANDARD §9).
