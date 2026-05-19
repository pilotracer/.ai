# Templates (`.ai/templates/`)

Portable starter files for new repositories using the **Agent OS** process layer.

## `.cursorrules` for your repo

Cursor (and compatible agents) read **`.cursorrules` at the repository root** — not inside `.ai/`.

| Step | Action |
|------|--------|
| 1 | Copy the template: `cp .ai/templates/cursorrules.template .cursorrules` |
| 2 | Search for `REPLACE:` tokens and fill in project name, paths, Docker services, protected files |
| 3 | Commit `.cursorrules` at the repo root next to `.ai/` and `.work/` |
| 4 | Keep **one** agent-rules file — do not add `AGENTS.md` unless your team explicitly standardizes on it |

The live project may already have a tailored `.cursorrules` at the root (e.g. AC Billing). Use this template only when **bootstrapping** a new repo or realigning governance.

**Reference:** [`.ai/README.md`](../README.md) · [path bootstrap tutorial](../docs/guides/workflows/20260518-tutorial-path-bootstrap.md)
