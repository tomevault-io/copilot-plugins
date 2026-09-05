## cbn-navalcarnero-sol

> This repository is the WordPress redesign for Club Baloncesto Navalcarnero.

# Agent instructions

## Project

This repository is the WordPress redesign for Club Baloncesto Navalcarnero.

Use Manchester Basketball Club only as conceptual inspiration for sports energy, rhythm, navigation and content hierarchy. Do not copy its code, exact layout, text, images, assets, colors or identity.

## Visual rules

- Color proportion across the site: 60% white, 30% red, 10% black.
- White must be the dominant surface color.
- Red is for brand energy, CTAs, active states, tickers and key bands.
- Black is for text, structural contrast, header/footer and limited emphasis.
- Yellow/gold is allowed only when it comes from official crest/assets, not as a main UI token.

## Stack

- WordPress.
- Custom theme: `wp-content/themes/cbn-theme`.
- MU plugin for structured content: `wp-content/mu-plugins/cbn-core`.
- Vite for frontend assets.
- GSAP + ScrollTrigger for selective animation.
- Lenis for progressive smooth scroll.
- Swiper for real carousel needs.
- WooCommerce + Redsys (bank virtual TPV, client decision 2026-07-02) is the payment direction, with Stripe as documented fallback only; payment implementation requires explicit scope and test mode. See `PAYMENTS.md` section 2.

## Commands

- `npm run build` compiles frontend assets.
- `npm run format` formats CSS, JS, JSON, Markdown and YAML.
- `npm run verify` runs formatting check and build.

## Rules

- Check `git status --short --branch` before edits.
- Prefer small, reviewable changes.
- Do not install dependencies without approval.
- Do not commit or push unless explicitly asked.
- Do not introduce secrets, tokens or credentials.
- Do not edit generated `assets/dist` manually.
- Use `apply_patch` for manual code and documentation edits.
- Respect existing PHP/theme structure unless a documented reason requires changing it.
- Keep frontend enhancements progressive and accessible.
- Respect `prefers-reduced-motion`.
- Verify visual work in a browser when possible.
- Before committing, pushing, creating a PR, or merging visual changes, show desktop and mobile screenshots to the user and wait for validation.
- Before committing, pushing, creating a PR, or merging non-visual changes, show a concise diff/summary to the user and wait for validation.
- Domain and federation credentials must never be committed, pasted into docs, or logged.

## Documentation to read first

- `README.md`
- `docs/redesign/navalcarnero-redesign-plan.md`
- `docs/redesign/navalcarnero-requirements-update-2026-06-27.md`
- `docs/redesign/homepage-mvp-2026-06-27.md`
- `docs/redesign/home-content-foundation-2026-06-27.md`
- `docs/redesign/federation-domain-architecture-2026-06-27.md`

# GitHub Multiagent Workflow

This repository contains a portable multiagent workflow for GitHub work, with surfaces for Codex and Claude Code.

When the user asks for branch, commit, pull request, merge, review, alignment, or changelog work, use the `github-multiagent-workflow` skill if it is available (Codex: `.agents/skills/`; Claude Code: `.claude/skills/`, with subagents in `.claude/agents/`). If skills are not available in the current runtime, read `docs/agent-system/README.md` and use the prompts under `docs/agent-prompts/` as role instructions.

## Operating Rules

- Treat the original user request as the controlling source of scope.
- Consult `AGENT_CHANGELOG.md` before actions that depend on prior decisions.
- Record relevant actions, decisions, blocked actions, assumptions, and validation results in `AGENT_CHANGELOG.md`.
- Ask for explicit human confirmation before destructive actions, history rewrites, deleting files, overwriting important work, force pushes, or merges with unresolved risk.
- Do not merge when tests fail, conflicts exist, alignment is unclear, PR context is incomplete, or the changelog is stale.
- Prefer reversible Git operations and clear branch names.
- Distinguish what Codex can execute directly from what it can only recommend because of permissions, missing tools, or required human approval.

## Model Routing Policy

- Use the smallest capable model for simple, low-risk tasks when the current Codex account/runtime supports model selection.
- Simple tasks include changelog entries, PR body drafting, status summaries, file inventory, formatting-only documentation edits, and other low-risk operations with clear inputs.
- Use a stronger reasoning model for architecture decisions, alignment checks, code review, security-sensitive work, merge readiness, payment/federation decisions, large refactors, unclear requirements, or any task with irreversible risk.
- If this Codex version does not expose per-agent or per-task model selection, keep this policy as routing guidance and state that the active Codex model is being used.
- Do not add unsupported `model` fields to `.codex/agents/*.toml` unless the target Codex version documents that syntax.

## Agents

Use these logical agents:

- `coordinator`: decomposes the task, assigns work, reads history, prevents duplicate or contradictory actions.
- `github_operator`: prepares or executes branch, commit, PR, and merge actions within permission limits.
- `code_reviewer`: reviews code changes for correctness, regressions, scope control, and missing tests.
- `pull_request_agent`: prepares PR title, body, checklist, risks, and test evidence.
- `alignment_guard`: checks every relevant action against the original user request.
- `changelog_keeper`: maintains `AGENT_CHANGELOG.md`.
- `final_validator`: checks merge readiness and blocks unsafe merges.

Current Codex versions can spawn custom agents from `.codex/agents/*.toml`. If that surface is unavailable, apply the same roles sequentially in the main thread using the Markdown prompts in `docs/agent-prompts/`.

---
> Source: [PabloColomo/cbn-navalcarnero-SOL](https://github.com/PabloColomo/cbn-navalcarnero-SOL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
