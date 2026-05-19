# ADRs (pointer)

**Architectural decision records live in [`.work/decisions/`](../../.work/decisions/)**, not under `.ai/`.

Binding **process** for ADRs during foundation planning is in [`plan-foundation` skill](../skills/plan-foundation/skill.md) P2. **Artifacts** are project working documents.

Portable skills reference `{DECISIONS_ROOT}` — see `.cursorrules` placeholder map.

## Index (2026-05-17)

See [`.work/decisions/`](../../.work/decisions/) for full files. Status in each ADR header wins over this table.

| ADR | Topic | Status |
|-----|-------|--------|
| 001 | Backend stack (Python/FastAPI) | Decided |
| 002 | Signing path (direct vs PAC) | Deferred |
| 003 | Hosting (AWS) | Decided |
| 004 | Tenancy (schema-per-tenant) | Decided |
| 005 | KMS (AWS) | Decided |
| 006 | First vertical (generic) | Decided |
| 007 | PDF renderer (WeasyPrint default) | Open |
| 008 | Locales (ES+EN GA) | Decided |
| 009 | CPA engagement | Accepted (risk accepted) |
| 010 | Contingency wording | Deferred |
| 011 | Document scope v1 | Decided |
| 012 | v1 interaction mode | **Decided** — evidence `.work/prompts/decision_012_v1_interaction_mode.md` |
| 013 | Inbound document ingestion | **Decided** |

Foundation register: `.work/plans/foundation/20260517-04-foundation-architecture.md` §13 must agree with ADRs; **ADRs win** on conflict.
