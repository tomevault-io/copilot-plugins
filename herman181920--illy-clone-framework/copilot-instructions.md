## illy-clone-framework

> A Claude Code **plugin** for universal website cloning. Guides the user through a systematic pipeline: clone → QA → fix → convert → analyze. Learns from each clone to get better.

# CLAUDE.md — illy-clone-framework

## What This Is
A Claude Code **plugin** for universal website cloning. Guides the user through a systematic pipeline: clone → QA → fix → convert → analyze. Learns from each clone to get better.

## Two Phases
1. **R&D:** Iterate on tools and skills until they consistently hit 98%+ on any site. Test on multiple site types.
2. **Production:** Skills are stable. User types `/clone https://example.com` and the pipeline runs to target parity without interactive questions on policy-level decisions (all governed by `clone-config.json`).

## Plugin Components
- **Skills:** `/clone`, `/qa` (visual pixel-comparison), `/manual-qa` (interactive behavior testing via Playwright MCP), `/import-browser-cookies` (decrypt + load Chromium cookies into Playwright), `/diff-flow-match` (replay source flow against clone, produce match-matrix with Stage 6 convergence loop), `/convert`, `/analyze`, `/improve`
- **Agents:** `cloner` (autonomous page crawling), `qa-reviewer` (screenshot comparison), `converter` (HTML→framework)
- **Hooks:** `post-clone-qa` (auto-trigger QA after cloning)
- **Scripts:** `clone_site.py` (generic CLI), `qa_compare.py`, `rebrand.py`, `capture_editor_states.py`, `import_browser_cookies.py`, `record_flow.py` (record/replay Playwright interaction flows as portable JSON specs)
- **Fixtures:** `scripts/test-fixtures/` (reproducible test media for manual-qa)
- **Docs:** `docs/clone-config.md` (policy config schema), `docs/patterns/modern-saas-editor.md` (IA + component skeleton for any timeline-centric creation SaaS), `docs/patterns/nextauth-subdomain-auth.md` (subdomain auth topology, cookie pitfalls, provider variants)

## Workflow
1. `/clone {url}` — Step 0 loads `clone-config.json` → auto-detect site (framework, pages, auth requirements) → clone → fix → QA
2. If QA < target_parity: auto-fix up to `max_fix_iterations` rounds → if still failing, get expert review → user decides
3. If QA ≥ target_parity: ask framework choice (React/Next.js/etc.) → `/convert` → `/analyze`
4. Update `docs/learnings.md`

## Key Rules
- Before starting any new clone, read `docs/patterns/modern-saas-editor.md` — the IA skeleton is already captured for timeline-centric creation SaaS. Only deviate where the target demonstrably differs.
- Always auto-detect auth pages before asking — present findings as a question with options.
- Always ask which framework before converting (Vite+React, Next.js, keep static, other).
- Use headless `playwright.sync_api` in Python for visual `/qa` (NOT Playwright MCP — causes conflicts).
- Use Playwright **MCP** (not Python) for `/manual-qa` — it stays open across steps for interactive driving.
- **Never hand testing to the user.** Drive the cloned app yourself until ≥99% of spec entries pass, then hand off. If a flow is genuinely unreachable from the environment, mark it BLOCKED with a reason — don't imply it passes.
- **Public audit is NOT enough for authenticated SPAs.** If the target has an `/app` or `/dashboard` subdomain, the marketing site is ~10% of the product. The real information architecture, feature taxonomy, and flows live behind auth. Always drive authenticated surfaces before writing the audit. If you can't authenticate, the audit is incomplete — say so explicitly.
- **Cookie-subdomain pitfall.** A session cookie for `app.target.com` lives ONLY on that exact subdomain. Cookies on `target.com` (marketing) or `auth.target.com` (SSO) do NOT authenticate the web app. Before loading cookies, inspect the Chrome DB for the exact subdomain. Use `/import-browser-cookies` skill's Step 4 troubleshooting.
- **Original visual design is the legal firewall, structural match is the value.** When cloning a commercial product, the copyrightable parts are: marketing copy, distinctive icons, illustrations, specific visual tokens (exact colors/fonts/spacing that form a distinctive brand), literal code. The NON-copyrightable parts are: information architecture, feature grouping, flow sequences, interaction patterns, standard UI layouts common to the category. A defensible competitor product matches the structural patterns 1:1 (same screens, same flows, same button positions in functional grid) while using an entirely original visual language (different font, different color palette, different icons, original copy, original illustrations). Do the structural work; keep the visual expression original.
- Agents run in **worktrees** for isolation.
- Rebuild before QA comparison (`npm run build`) — stale builds give false results.
- Document every revision in `clones/{domain}/revisions/revision-log.md`.
- Update `docs/learnings.md` with new patterns after each clone.

## Policy Config
Framework-wide behavior (auto-commit between phases, target parity, allow subagent plan mode, "no inventions" audit, max fix-loop iterations) lives in `clone-config.json`. Resolution order: `clones/{domain}/clone-config.json` → `~/.claude/clone-config.json` → `.claude-plugin/clone-config.default.json`. Full schema + per-field semantics in `docs/clone-config.md`. Every skill in the pipeline reads this file instead of asking you a policy question.

## The 95% Wall (R&D Finding)
Rebuilding components from visual reference caps at ~95%. To hit 99%+: convert original HTML to JSX while keeping original CSS verbatim. The `/convert` skill's HTML-to-JSX converter is the main R&D focus.

## Benchmarks
- Static clone target: ≥99% (visual, via `/qa`)
- Framework template target: ≥98% (visual, via `/qa`)
- Interactive SPA target: 100% PASS on all advertised behaviors (via `/manual-qa` or `/diff-flow-match`) — BLOCKED entries allowed only when documented with a concrete env limitation.

## Auth
- Use a disposable or dedicated email when creating accounts on target sites for cloning.
- Open visible browser, log in, capture cookies for reuse via `/import-browser-cookies`.

## Dependencies
- Python 3.11+: `playwright`, `beautifulsoup4`, `aiohttp`, `lxml`, `Pillow`
- Node (for templates): `vite`, `react`, `react-router-dom`, `tailwindcss`, `lucide-react`
- Run `playwright install chromium` if browsers not installed.

## Project Structure
```
.claude-plugin/plugin.json               # Plugin manifest
.claude-plugin/clone-config.default.json # Framework policy defaults
skills/{clone,qa,convert,analyze,improve,manual-qa,diff-flow-match,import-browser-cookies}/SKILL.md
agents/{cloner,qa-reviewer,converter}.md
hooks/hooks.json                         # Post-clone QA trigger
scripts/                                 # Python tools
docs/clone-config.md                     # Policy schema
docs/patterns/                           # Reusable IA skeletons
docs/learnings.md                        # Auto-updated patterns per site
```

## Your clones
Your cloned sites live outside this plugin — typically in a separate working directory. This repo ships the framework; your clones are your own project using the framework.

---
> Source: [herman181920/illy-clone-framework](https://github.com/herman181920/illy-clone-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
