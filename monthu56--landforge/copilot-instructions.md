## landforge

> Guidance for Claude Code working **on this system** (the landing-generation toolkit itself).

# CLAUDE.md

Guidance for Claude Code working **on this system** (the landing-generation toolkit itself).
Human overview: [`README.md`](README.md). Decisions: [`docs/adr/`](docs/adr/).

## What this is

A **content-management / generation system for static landing pages**, delivered as Claude Code
skills + deployment infra. **Not** a build-tooling project (no bundler, test runner, lint).

> **Hard invariant:** a landing is a **static HTML file with content in the markup and zero JS on
> the critical path** (CSS/JS inlined). Crawlers must see real content, not an empty root. Every
> generator and skill must preserve this.

## Skills (`.claude/skills/`)

- **`new-landing`** — landing from a brief (contract + skeleton + `check_landing.py`).
- **`landing-experiment`** — A/B on one URL (inline split, no cloaking), variant tracking, card.
- **`landing-journal`** — audit journal: `experiments/journal.jsonl` + `journal.md` + git. **Log
  every meaningful action** via `scripts/log_event.py`.
- **`landing-ads`** — UTM, ad copy, keywords, editor exports → `ads/<campaign>/` (artifacts only).
- **`init-landing-system`** — deploy this whole system into another project (snapshot + sync +
  user-level install).

## MCP server (`mcp/`)

The toolbox is also exposed as an **MCP server** (ADR-0002) — any MCP host can call it, not only
Claude Code. Thin wrapper: each tool runs the same `.claude/skills/*/scripts/*.py` via subprocess
(one source of logic). Validators / journal / UTM as **tools**; contract, brief schema, skeleton
and journal as **resources**; brief interviewer / generation / A/B checklist as **prompts**.
Transport stdio; optional dependency (Python ≥ 3.10, `mcp`); the
static-HTML invariant is untouched. New skill script → add a matching tool (keep CLI↔MCP parity).
See [`mcp/README.md`](mcp/README.md).

## Infra & deployment

`.github/workflows/deploy.yml` routes by branch: `main` → prod (indexable); `dev` → staging
(noindex); any other → preview (noindex). Server runs Traefik + two `nginx` containers; only prod
is indexed (`X-Robots-Tag: noindex` on staging via nginx). Infra values are `{{PLACEHOLDER}}`s
(domains/containers/dirs) — substituted by `init-landing-system` or by hand
(`.claude/skills/init-landing-system/reference/placeholders.md`). Deploy secrets: `DEPLOY_HOST`,
`DEPLOY_USER`, `DEPLOY_SSH_KEY`, `DEPLOY_KNOWN_HOSTS`.

When the system itself changes (skills/CI/infra), refresh the init snapshot:
`bash .claude/skills/init-landing-system/scripts/sync_templates.sh`.

## Conventions

- In-code comments in Russian; commit messages in English.
- Any landing must pass `.claude/skills/new-landing/scripts/check_landing.py` (one H1, canonical,
  OG, JSON-LD, `build:` marker, **no external CSS/JS on the critical path**).
- Changing infra/deploy/system structure → update `README.md` **and** `CLAUDE.md`, add an ADR.
- Placeholders: infra `{{PROD_DOMAIN}}`/`{{…}}` are deployment config; content placeholders in the
  skeleton (`{{TITLE}}`, `{{METRIKA_ID}}` …) are filled when generating a landing — keep them.

---
> Source: [monthu56/landforge](https://github.com/monthu56/landforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
