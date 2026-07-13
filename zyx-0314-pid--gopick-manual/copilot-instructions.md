## gopick-manual

> Workspace instructions for GoPick documentation and frontend edits. Use when changing HTML, CSS, JS, pages, or docs; contains run commands and PR checklist.


# GoPick — Workspace Instructions

Purpose: Provide lightweight, always-available guidance for making content and frontend changes across this repository.

When to load: Use when editing files under `pages/`, `assets/`, `index.html`, or `docs/`.

Local dev:
- Start the site with a static file server pointed at the repo root (example):
  - `npx http-server -p 82` or use your preferred static server.
  - The admin docs mention `http://localhost:82/app/administrator/` as a local entrypoint.

Conventions:
- HTML/CSS/JS: primary site files live at the repository root and under `pages/`.
- Duplicate page copies for section-specific assets are under `pages/*/assets/` — update both copies when changing shared behavior.
- Docs are in `docs/` and use Markdown for non-code content.

PR Checklist:
- **Build:** Verify pages render locally using a static server.
- **Paths:** Update both the root and page-level asset copies if applicable.
- **Docs:** If changing behavior, update `docs/*.md` to reflect the change.
- **Accessibility:** Ensure images have alt text and forms are labelled.
- **Style:** Run a quick lint/format pass on changed CSS/JS.

Anti-patterns / Do not:
- Use `applyTo: "**"` in project instructions; prefer targeted globs.
- Put long-running scripts or secrets in these instructions.

Contacts:
- Repo owner / maintainer: check project README or internal team documentation.

If you'd like, I can extend this with example prompts, or add file-scoped instructions for `pages/`.

**Coding Principles**

- **Human Priority:** Human intent and decisions take precedence over AI suggestions. Follow the process: Plan > Execute > Feedback > Revise.
- **Spacing:** Use clear spacing between functions, classes, and logical sections.
- **Naming:** Use explicit, multi-word names that clearly convey purpose; avoid generic terms.
- **Single Responsibility:** Each function should follow SOC/SRP (single responsibility / separation of concerns).
- **File Separation:** Keep JS, CSS, and HTML separated — no inline JS/CSS. Use multiple JS/CSS files for focused tasks and one `script.js`/`style.css` as the root aggregator.
- **Shared Assets:** Create shared `assets/js/` and `assets/css/` in the repository root for reusable code/styles.
- **Abstraction Level:** Keep JS at one level of abstraction per function/module.
- **CSS Structure:** Prefer nested CSS (using a preprocessor or clear section grouping) and keep styles modular.
- **JS Accessors:** Use explicit getters and setters where appropriate.
- **Comments:** Use `TODO` comments only; separate HTML sections/components with clear markers.

**Don't Do Again (AI must always follow)**

- Do not modify code in a way that overrides explicit human intent or instructions.
- Do not add inline `<script>` or `<style>` tags to HTML files.
- Do not use single-letter or overly generic names; prefer explicit multi-word identifiers.
- Do not combine multiple responsibilities in a single function or mix levels of abstraction.
- Do not add comments other than `TODO` and explicit HTML section markers.
- Do not commit workspace-level changes affecting architecture or naming without explicit user approval.
- Do not introduce implicit global state or hidden side-effects in JS.
- Do not remove or skip verification steps without discussion and approval.

---
> Source: [zyx-0314-pid/gopick_manual](https://github.com/zyx-0314-pid/gopick_manual) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
