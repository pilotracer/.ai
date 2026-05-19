---
name: dev-stack
description: >
  Generate or update a `start.sh` (or `bin/start.sh`) dev-stack management script for
  Docker Compose projects. Use when the user invokes @dev-stack, says dev-stack, or asks
  to create, scaffold, rewrite, or extend a start/stop/manage shell script for a Docker-based
  dev environment, mentions terms like "start script", "dev menu", "docker compose wrapper",
  "stack manager", "nuke volumes", "drop tables", or asks how to safely manage a docker compose
  stack without affecting other projects on the host. Also trigger when the user shares an
  existing start.sh and asks to add commands, adapt it to a new stack, or harden its
  isolation guarantees.
---

# dev-stack

Generate or update a **single-file** POSIX shell script (`bin/start.sh` preferred) that gives developers a **safe, isolated** control plane for **one** Docker Compose project — without affecting other stacks on the host. **Project-agnostic:** adapt compose paths, service names, profiles, and DB tooling to the target repo.

**Tool-agnostic** (Cursor, Claude Code, opencode, Codex). **Pairs with:** project `docker-compose*.yml`, `.env.example`, local `.env` (never committed).

**Canonical path:** `.ai/skills/dev-stack/skill.md` · **Invocation patterns, tables, CI:** `reference.md`

**Hard rules:**

- **Never `source .env`.** Parse key=value safely (see `reference.md` §Safe env parsing).
- **Stack isolation is mandatory.** Every `docker compose` call uses the full scoping pattern in `reference.md` §Compose invocation — no bare `docker stop` / `docker rm` / wildcard volume nukes for "cleanup".
- **Dangerous operations** (`down -v`, drop schema, flush caches) require **double confirmation** with project/database identity — pattern in `reference.md` §Dangerous commands.
- **Dual mode:** interactive **full menu** when invoked with no args or `dev`; headless subcommands (`start`, `stop`, …) for CI; `MENU_QUIET=1` for non-interactive, capture-on-failure logging — see `reference.md` §CI.
- **Menu UX:** categorized sections, live stack summary (running/total, health probes), optional terminal colors, `r` refresh, `?` help — see `reference.md` §Interactive menu.
- **`set -euo pipefail`** at top; in interactive menu loops, scope `set +e` / `set -e` around compose calls so failures return to the menu instead of exiting the shell.
- **Respect protected files** where the repo marks them (e.g. this project's `.cursorrules` §Protected Files): do not change `docker-compose*.yml`, `Dockerfile.*`, or `.env*` **unless the user explicitly permits** those edits in the same message. The skill may still **generate** `bin/start.sh` without touching compose.

---

## Default output

| Artifact | Rule |
|----------|------|
| **Path** | Prefer **`bin/start.sh`** at the repository root (create `bin/` if missing). Use another path only if the user names it. |
| **Executable** | Instruct `chmod +x bin/start.sh` (or apply in the same change if the user asked to create the file). |
| **Self-contained** | One file; no required sibling `lib/` scripts unless the user explicitly wants a split layout. |

---

## When the user has not asked for a file yet

If they only want the **skill** or "how to": output the **script body** in a fenced `sh` block and state that they should save it as `bin/start.sh` and chmod. If they asked to **create** the file, write `bin/start.sh` and verify syntax (`sh -n`) when possible.

---

## Generation protocol

### S1 — Discover the stack (ask or read)

Minimum inputs (record gaps as **Unverified** until filled):

1. Compose file path(s) — default `docker-compose.yml` at repo root.
2. **`COMPOSE_PROJECT_NAME`** (or equivalent) — must be stable and unique per clone; recommend reading from `.env` with a documented default.
3. **Compose profile** — e.g. `dev`, `local` (omit `--profile` only if the project truly uses no profiles).
4. Service names for DB/Redis waits and one-off `exec` targets.
5. DB engine (Postgres, MySQL, none) and how schema is applied (migrations CLI, raw SQL, none).
6. URL/port hints — which env vars expose host ports (project-specific).

### S2 — Scaffold from the contract

Implement the **anatomy** and **command set** in `reference.md`. Adapt:

- `_compose_invoke` / `dc` / `quiet_dc` to the repo's compose files and profile.
- `urls_hint` to actual env vars.
- DB commands: include, omit, or rename per stack.

### S3 — Isolation audit (mandatory before delivery)

Confirm in the generated script:

- [ ] No bare `docker` mutations outside `docker compose` with project name + project directory + compose file(s).
- [ ] No `source .env` or `eval` on env content.
- [ ] Destructive paths use double confirmation + `print_banner` (or equivalent) showing **which** project.

### S4 — Completion checklist

| # | Check | Result |
|---|--------|--------|
| 1 | `bin/start.sh` path (or user path) agreed | pass |
| 2 | Compose scoping pattern present | pass |
| 3 | Safe `.env` reader present | pass |
| 4 | `--help` lists subcommands | pass |
| 5 | `MENU_QUIET=1` + CI path documented | pass |
| 6 | `bash -n bin/start.sh` (or `sh -n` for POSIX scripts) | pass / skip |

---

## Anti-patterns

- `docker compose down` without `-p` / wrong project directory (destroys wrong stack).
- Sourcing `.env` for convenience.
- Single `y/n` for volume wipe.
- Hard-coded container names from other projects.
- Embedding secrets in the script; read from env only.

---

## Source note

Plain-text authoring for this skill lives in **`skill.md` + `reference.md`**. If a collaborator provides a `*.skill` binary blob, do not treat it as canonical — extract requirements in chat or replace with this markdown skill.
