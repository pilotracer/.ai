# Changelog

All notable changes to Agent OS are documented here. Format inspired by [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html) once tagged.

## [Unreleased]

### Added
- **`probe` mode** for `@plan-foundation` and `@plan-master` — adaptive, gap-driven interrogation loop. Scores knowledge/plan coverage across fixed dimensions, asks ≤5 targeted questions per iteration, records answers into the canonical registries + doc 01 / plan body, and loops to a confidence target (default 85%). Sub-modes: `probe`, `probe - until ready`, `probe - status`.
- **`skills/probe-protocol.md`** — single-source-of-truth engine (loop, Coverage Score, ledger, ease-of-use rules) reused by both probe modes; skills supply only a coverage profile. New canonical verb `probe` registered in `SKILL_DEPENDENCIES.md`. Not a skill folder (14-skill count unchanged).
- **`plan-foundation` § Foundation coverage map** (D1–D10) → exit gate S4 plan-master-ready; **`plan-master` § Master coverage map** (M-D1…M-D7, the interactive front-end to `integrity`) → exit gate implementation-ready.
- **`templates/work/plans/foundation/PROBE_LEDGER.md.template`** — resumable, auditable probe state (`{PLANS_ROOT}/foundation/PROBE_LEDGER.md` · `{PLANS_ROOT}/full/PROBE_LEDGER.md`).
- **`scripts/readiness-verify.sh`** — machine-checkable honesty linter for probe ledgers (CI-ready). Fails when a `confirmed/high` dimension cites no evidence, when the header Coverage % disagrees with the table-computed value (tol 2 pts), or when coverage ≥ target while a gate-blocking (★) dimension is still `unknown`. Exits 0 when no ledger exists (probe is optional).
- **`@plan-verify coverage`** (alias `registry`) — read-only code-to-SPEC parity audit; surfaces vs `{FEATURE_SPEC_ROOT}` Implementation map and DIRECTORY_MAP.
- **`@plan-repair repair - from coverage`** — register unmapped surfaces via `@feature-spec create` + DIRECTORY_MAP (no parallel `feature.yml` canon).
- **FEATURE_STANDARD §14 Implementation map** (optional) — primary code paths for brownfield locate-ability.

### Changed
- **CI + `@session-control close`** now run `scripts/readiness-verify.sh` — `.github/workflows/framework-verify.yml` adds a probe-ledger honesty step (no-op without a ledger); session-control close C2 adds a probe-ledger pre-check that routes failures to `@plan-foundation probe` / `@plan-master probe`.
- **`.cursorrules` / `cursorrules.template` § Host hygiene** — container-first package installs; anti-pattern for host `npm`/`pip` on Compose projects; frontend `npm ci` in-container example.

## [0.2.0] - 2026-05-21

### Added
- **`plan-verify`** and **`plan-repair`** skills — symmetric planning verify/repair; brownfield framework alignment without formal `@plan-foundation` / `@plan-master`; open-language Request interpretation and R0-free alignment maps.
- **`code-repair`** skill — remediate verifier, migration, and SPEC findings with mandatory re-verify.
- **`code-verify`** / **`code-repair`** — open-language free requests: Request interpretation (verify), Implementation alignment map R0-free (repair); `code-verify status` mode.
- **`code-implementation continue` batch targets:** `- N` (e.g. `- 5`), `- until blocked`, `- M{N}-T{a}..T{b}`; default `continue` = 1 task. All modes stop on task gate fail or blocker; progress lines `Batch k/N: M4-T4 done`; mandatory batch summary.
- **Framework CI** - `.github/workflows/framework-verify.yml` runs `scripts/framework-verify.sh` and `scripts/smoke-consumer.sh` on push/PR.
- **`scripts/framework-verify.sh`** - self-hosted layout, consumer bootstrap smoke, stale integration path grep, template `REPLACE:` check, markdown relative link scan (expects **14** skill directories).
- **`scripts/smoke-consumer.sh`** - local consumer adoption smoke with next-step hints.
- **`docs/adoption/minimal-adoption.md`** - lite vs full adoption paths.
- **README / START_HERE / templates/README** - path convention table (nested `.ai/` vs self-hosted git root).
- `README.md` discoverability: hook rewrite, badges, comparison table, "Skills at a glance" (**14** skills).
- `LICENSE` (MIT).
- `CONTRIBUTING.md` and this `CHANGELOG.md`.
- "Template - your repo's HANDOFF.md will be filled by `@session-control`" notes on demo `.work/` artifacts so visitors don't mistake them for real project content.
- **`SKILL_DEPENDENCIES.md § Canonical command vocabulary`** - single source of truth for shared verbs (`status`, `init`, `start`, `continue`, `plan`, `complete`, `verify`, `repair`, …).
- **`SKILL_DEPENDENCIES.md § Blocked report shape`** - uniform "blocked (prerequisite)" report that every skill emits when a gate stops execution.
- **`dev-stack`** gains `status` and `init` modes plus a brownfield gate that refuses to silently overwrite an existing `bin/start.sh`.
- **G1** `code-implementation plan` (was `plan-iteration`) PI1 stops with the unified blocked-report shape on missing/Draft plan without HANDOFF waiver.
- **G2** `feature-spec create` adds CR0: hard-stops on existing slug folder; warns (and proceeds on confirm) when `plan-master-ready: no`.
- **G3** `db-migration create`/`add`/`run`/`verify` now require `db-migration init` to have run (runner + `001_init.sql` baseline).
- **G4** `code-verify milestone` M0 gate stops if the requested milestone is not defined in master plan §19 or active in `NEXT.md`.
- **B1** `project-bootstrap init` adds B0 brownfield detection (overwrite-all / overwrite-missing / keep / abort).
- **B2** `db-migration init` adds IB0 brownfield detection (keep / overwrite-runner / overwrite-all / abort).
- **`dev-stack`** `status` mode for `bin/start.sh` health-check.

### Changed
- **Invocation punctuation:** Skills, workflows, and command examples use ASCII hyphen `-` (not em dash `—`) between verb and argument (e.g. `@code-implementation plan - M1`).
- **`code-implementation`** mode renamed: **`plan-iteration` → `plan`** (legacy alias retained).
- **`plan-master`** mode renamed: **`task` → `show`** (legacy alias retained).
- **`process-router`** documents explicit **`route`** token.
- **Unified "blocked (prerequisite)" report shape** across plan-foundation, plan-master, code-implementation, code-verify, feature-spec, db-migration.
- **`db-migration`** I5 long FastAPI lifespan code blocks moved from `skill.md` to `reference.md`.
- **Canonical migration policy** in `.cursorrules` / `cursorrules.template`.

## [0.1.0] - 2026-05-19

First public-ready cut of Agent OS as a portable framework.

### Added
- **`project-bootstrap`** skill plus `templates/bootstrap.sh` + `templates/work/` skeleton (one-command adoption into any repo).
- **`skills/SKILL_DEPENDENCIES.md`** registry: explicit prerequisite gates between skills (foundation → certify → master plan → status → implementation).
- **Prerequisite-gate enforcement** in `plan-foundation`, `plan-master`, `code-implementation`, `session-control` - agents now **stop** with a redirect if an upstream step was skipped.
- **README bird's-eye flow** (`@project-bootstrap init` → … → `@code-implementation continue`) and "Skills at a glance" table covering all 11 skills.
- **Standards templates** (`20260517-CONVENTIONS`, `FEATURE_STANDARD`, `DIRECTORY_MAP`, `observability-spec`, `api-style-guide`, `threat-model`, `data-classification`) generalized with `REPLACE:` tokens.
- **Demo `.work/` skeleton** at repo root so pointer links resolve when Agent OS is the git root.

### Changed
- Skill/router invocation syntax standardized to hyphen (e.g. `@feature-spec create - <slug>`) instead of em dash.
- Workflow guides under `docs/guides/workflows/` generalized; vendor specifics removed.

### Removed
- All project-specific integration artifacts under `docs/integration/` (Costa Rica Hacienda, OIDC, XAdES bundles) - replaced with agnostic `MANIFEST.template.txt` + `README.md`.

[Unreleased]: https://github.com/PiloTracer/.ai/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/PiloTracer/.ai/releases/tag/v0.2.0
[0.1.0]: https://github.com/PiloTracer/.ai/releases/tag/v0.1.0
