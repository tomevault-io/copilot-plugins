## skillify-foundation

> This is **not an application** — it is a research-backed toolkit for engineering `SKILL.md` files (Agent Skills) for AI coding agents. There is no build, server, or runtime; the "product" is Markdown skill folders. Tooling is Python 3 standard-library only — no dependencies, no virtualenv. Run all commands from the repo root.

# AGENTS.md

This is **not an application** — it is a research-backed toolkit for engineering `SKILL.md` files (Agent Skills) for AI coding agents. There is no build, server, or runtime; the "product" is Markdown skill folders. Tooling is Python 3 standard-library only — no dependencies, no virtualenv. Run all commands from the repo root.

## Commands

- Scaffold a skill skeleton: `python3 skillify/scripts/init_skill.py <skill-name>`
- Validate frontmatter + structure (gate 1): `python3 skillify/scripts/quick_validate.py treasury/<skill-folder>`
- Check local reference/template/script links resolve: `python3 skillify/scripts/check_links.py treasury/<skill-folder>`

## Testing

Validating a skill folder *is* the test suite — there is no global test runner.

- A change "passes" when `quick_validate.py` and `check_links.py` both exit zero **and** the skill scores ≥4/5 on every rubric dimension and ≥40/50 total (`skillify/references/validation-rubric.md`).
- Some skills ship their own checks (e.g. `treasury/eliciting-banking-brief/scripts/check_frame_rule_data_drift.py`) — run those directly when touching that skill.
- If gate 1 fails, **do not write target files** — surface the error verbatim and stop.

## Project structure

Three layers, each grounded in the one before it:

- `literature/` — 68 curated source documents (~168K words, clean markdown as of the 2026-07-05 corpus refresh) across the agent ecosystem. Read-only research corpus; do not edit source files.
- `skillify/` — the skill-creator *meta-skill*: one `SKILL.md` (8 modes) plus `references/`, `templates/`, `scripts/`, `examples/`. The engine that produces and audits every other skill.
- `treasury/` — 131 production skills grouped into 9 purpose groups. `treasury/README.md` is the **authoritative catalog** (the root `README.md` count can lag — trust `treasury/README.md`).

## Code style

A skill is a folder mapped onto a 3-tier context-loading model; content lives in **exactly one** tier (never duplicate guidance — point to the reference instead):

- **Tier 1** (always loaded): `name` + `description` frontmatter, ~100 words.
- **Tier 2** (loaded on trigger): the `SKILL.md` body — imperative numbered steps with entry/exit conditions and an exact output contract; <5,000 words / ≤500 lines.
- **Tier 3** (loaded on demand): `references/` (one level deep), `templates/`, `scripts/`, `schemas/`, `examples/`, `assets/`.

`quick_validate.py` mechanically enforces: `name` is lowercase kebab-case, ≤64 chars, **equals the folder name**, and contains no `claude`/`anthropic`; `description` ≤1024 chars with trigger language ("Use when" / "Use for" / "Use after" / "Triggers on" / "Activate when"); no XML angle brackets in any frontmatter field; `SKILL.md` ≤500 lines; `references/` exactly one level deep; and no `README.md`/`CHANGELOG.md`/`INSTALLATION_GUIDE.md`/`QUICK_REFERENCE.md`/`CONTRIBUTING.md` inside a skill folder.

`skillify/SKILL.md` operates in 8 modes — Create, Refactor, Review, Audit, Compress, Split, Merge, Adapt. Review and Audit are **read-only — they never edit files**. Refactor/Compress/Split/Merge preserve the original (diff or sibling dir) unless the user authorizes in-place overwrite.

## Git workflow

- Work on feature branches off `main`, merged via PR (e.g. `Jin9/<topic>`). Git author of record is `chinnawat.w` (GitHub `Jin9`).
- Commit and push only when the user asks; branch off `main` first. Do not self-merge — request review.
- PR body: state what changed and how it was validated (which gate steps ran and their result).

## Boundaries

### Always allowed (no approval)
- Read any file; run the validation/link scripts above; scaffold a skill skeleton; open a draft PR.

### Requires human approval first
- Committing or pushing — wait for an explicit request.
- Inventing trigger phrases, target users, or output contracts the user did not provide — ask once instead.
- Editing repo-policy files (`AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`) as part of skill work — only when the user explicitly asks for platform-policy adaptation.

### Explicitly prohibited (with the allowed alternative)
- Do not edit `literature/` source documents — treat the corpus as read-only evidence.
- Do not produce the artifact a skill *would generate* — produce the **skill** instead.
- Do not write target files when gate 1 fails — surface the error verbatim and stop.
- Do not bake host-specific assumptions into a skill body — keep skills portable across hosts (Claude Code, Codex, Copilot, Gemini/Antigravity) and use Adapt mode for platform conventions.
- Do not rely on `skill-design-methodology/` (root) being present — it is gitignored and local-only.

---
> Source: [Jin9/skillify-foundation](https://github.com/Jin9/skillify-foundation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
