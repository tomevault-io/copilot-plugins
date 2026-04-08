## jhblogv2

> This repository is a Hugo site using the PaperMod theme.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a Hugo site using the PaperMod theme.
- `content/`: site content, especially posts in `content/blog/` and pages like `content/about.md`.
- `static/`: static assets (images, icons) copied as-is to the built site.
- `themes/PaperMod/`: PaperMod theme as a git submodule.
- `layouts/`: local layout overrides (if needed).
- Root config: `config.yaml` (site config), `netlify.toml` (deployment build settings).

## Build, Test, and Development Commands
- `hugo server -D`: run local dev server and include draft content.
- `hugo`: build production output into `public/`.
- `HUGO_CACHEDIR=$(pwd)/.cache/hugo hugo --gc --minify`: production-like local build with cache in project directory.

Netlify is pinned via `netlify.toml` (`HUGO_VERSION = "0.146.0"`). Keep local Hugo at `>= 0.146.0` for PaperMod compatibility.

## Coding Style & Naming Conventions
- Use 4-space indentation in project-edited code and config blocks.
- Prefer lowercase, hyphen-separated slugs in file/folder names.
- For new posts, follow date-first naming in `content/blog/`, for example:
  - `content/blog/2026-02-06-my-topic/index.md`
- Keep front matter fields consistent with existing posts (`title`, `date`, `tags`, `categories` when needed).

## Testing Guidelines
There is no dedicated automated test suite in this repository.
- Run `hugo` before opening a PR; treat build warnings/errors as blockers.
- Preview changes with `hugo server -D` and verify page rendering, links, code highlighting, and images.
- For post bundles, confirm relative asset paths resolve correctly.

## Commit & Pull Request Guidelines
- Follow the existing commit style: short, imperative messages (for example, `add zotero shortcut post`, `update website`).
- Keep commits focused to one logical change.
- Primary development and deployment branch is `main`. The previous blogdown site is archived on `archive/blogdown-r` (read-only).
- PRs should include:
  - a concise summary,
  - affected paths (for example, `content/blog/...`, `layouts/...`),
  - local validation performed (`hugo` build result),
  - screenshots for visible layout/style changes.

## Security & Configuration Tips
- Do not commit generated outputs or local state: `public/`, `resources/`, `.hugo_build.lock`, `.Rhistory`, `.RData`, `.Rproj.user/`.
- Never commit credentials, API keys, or private tokens in content or config files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/timedreamer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/timedreamer)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
