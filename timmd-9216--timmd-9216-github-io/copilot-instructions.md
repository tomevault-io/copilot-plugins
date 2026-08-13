## timmd-9216-github-io

> This file orients AI coding agents to this repository so they can be immediately productive.

Purpose
-------
This file orients AI coding agents to this repository so they can be immediately productive.

**Quick context**
- **Repo type:** Minimal GitHub Pages / static site repository. At root there is `README.md` and a `.git` directory; there are no detected build files (no `package.json`, `Gemfile`, or other tool manifests).

**Primary goals for an AI agent**
- Locate and edit site content (Markdown/HTML) in the repository root.
- Avoid introducing tooling changes unless asked; keep edits minimal and self-contained.

**What I looked for (evidence)**
- Root files: `README.md` — see that file for the current human-provided description.
- No `package.json`, `Gemfile`, `Makefile`, or `docs/` directory were found.

**How to be productive (actions & examples)**
- Inspect `README.md` first to understand the site purpose and language.
- When editing content pages, prefer keeping files in the repository root unless an existing folder is discovered.
- Preview changes locally by serving the repository as a static site, for example:

```bash
# from repo root
python -m http.server 8000
# then open http://localhost:8000
```

- If adding JS/CSS or build tooling, first check for any existing manifest files. If none exist, document the change in the PR description and explain why a build system was added.

**Patterns & conventions (discoverable)**
- This repo currently contains only static content; treat commits as edits to content rather than code features.
- Prefer plain Markdown/HTML edits; avoid introducing dependencies or CI changes without explicit human approval.

**When to ask for clarification**
- If a change requires introducing a build system (npm, Jekyll, etc.), stop and request guidance from the maintainers.
- If you cannot find expected pages or assets, ask which branch or path is authoritative (some projects use `gh-pages` or `docs/`).

**Files to reference**
- `README.md` — starting point for project intent and any human notes.

If this guidance is missing anything you need (build commands, target branch, or content locations), please tell me what to look for and I will update this file.

---
> Source: [timmd-9216/timmd-9216.github.io](https://github.com/timmd-9216/timmd-9216.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
