# Templates (`.ai/templates/`)

Portable starter files for new repositories using **Agent OS**.

## Quick start

From the **repository root** (directory containing `.ai/`):

```bash
bash .ai/templates/bootstrap.sh
```

Or: `@project-bootstrap init`

---

## What gets created

| Output | Source template |
|--------|-----------------|
| `.cursorrules` | `cursorrules.template` |
| `DOCS_TECH_STACK.md` | `DOCS_TECH_STACK.md.template` |
| `.work/README.md` | `work/README.md.template` |
| `.work/context/HANDOFF.md` | `work/context/HANDOFF.md.template` |
| `.work/plans/NEXT.md` | `work/plans/NEXT.md.template` |
| `.work/plans/ASSUMPTIONS.md` | `work/plans/ASSUMPTIONS.md.template` |
| `.work/plans/RISK_REGISTRY.md` | `work/plans/RISK_REGISTRY.md.template` |
| `.work/plans/UNKNOWNS.md` | `work/plans/UNKNOWNS.md.template` |
| `.work/decisions/README.md` | `work/decisions/README.md.template` |
| `.work/features/README.md` | `work/features/README.md.template` |
| `.work/prompts/README.md` | `work/prompts/README.md.template` |
| Empty dirs | `plans/foundation/`, `full/`, `operations/`, `proposals/`, `archives/` |

**Not copied by bootstrap** (author via skills or copy manually):

| Artifact | Template path | Created by |
|----------|---------------|------------|
| Foundation 01–04 | `work/plans/foundation/YYYYMMDD-*.template` | `@plan-foundation greenfield` |
| Master plan | `work/plans/full/YYYYMMDD-full-plan.md.template` | `@plan-master greenfield` |
| Feature SPEC | `work/features/example-slug/…` | `@feature-spec create` |
| ADR | `work/decisions/YYYYMMDD-NNN-slug.md.template` | plan-foundation / manual |
| Ops proposal | `work/plans/operations/…` | plan-foundation P5 |

---

## `REPLACE:` checklist (`.cursorrules`)

| Token | Purpose |
|-------|---------|
| `REPLACE:PROJECT_NAME` | Product / repo name |
| `REPLACE:TECH_STACK_DOC` | Stack doc path |
| `REPLACE:CONVENTIONS_FILE` | CONVENTIONS filename under `.ai/standards/` |
| `REPLACE:FEATURE_STANDARD_FILE` | FEATURE_STANDARD filename |
| `REPLACE:DIRECTORY_MAP_FILE` | DIRECTORY_MAP filename |
| `REPLACE:OBSERVABILITY_SPEC_PATH` | Observability standard path |
| `REPLACE:BOUNDARY_MAP_PATH` | Boundary map path |
| `REPLACE:APP_ROOT` / `REPLACE:APP_WORKDIR` | Application paths |
| `REPLACE:MIGRATIONS_DIR` / `REPLACE:MIGRATION_*` | SQL migrations |
| `REPLACE:PLATFORM_PACKAGE` | Shared package import name |
| `REPLACE:SERVICE_*` / `REPLACE:STACK_SUFFIX_VAR` | Docker Compose |
| `REPLACE:TEST_COMMAND` / `LINT` / `TYPECHECK` | Verification gates |
| `REPLACE:DEV_STACK_SCRIPT` / `REPLACE:SCRIPTS_DIR` | Dev tooling |
| `REPLACE:FRONTEND_CONFIG_PATHS` | Protected frontend configs |

Full list also appears in `cursorrules.template` § Placeholder quick map.

Also customize **standards** under `.ai/standards/20260517-*.md` and set:

- `REPLACE:CONVENTIONS_FILE`
- `REPLACE:FEATURE_STANDARD_FILE`
- `REPLACE:DIRECTORY_MAP_FILE`
- `REPLACE:OBSERVABILITY_SPEC_PATH`

---

## `work/` template tree

```text
templates/work/
├── README.md.template
├── context/HANDOFF.md.template
├── plans/
│   ├── NEXT.md.template
│   ├── ASSUMPTIONS.md.template
│   ├── RISK_REGISTRY.md.template
│   ├── UNKNOWNS.md.template
│   ├── foundation/          ← 4 foundation doc templates
│   ├── full/                ← master plan outline
│   └── operations/          ← docker compose proposal
├── features/example-slug/   ← SPEC template
├── decisions/               ← README + ADR template
└── prompts/README.md.template
```

---

## Framework repository note

When the git root **is** this Agent OS tree, `.work/` at the same root holds an **empty demo skeleton** so pointer links from `.ai/plans/README.md` resolve. Application projects keep `.ai/` + `.work/` as siblings under the app repo root.

**Reference implementation** (filled examples): use a separate application repository; do not copy project-specific content into these templates.

---

## Related

- [`.ai/README.md`](../README.md)
- [`project-bootstrap` skill](../skills/project-bootstrap/skill.md)
- [Path bootstrap tutorial](../docs/guides/workflows/20260518-tutorial-path-bootstrap.md)
