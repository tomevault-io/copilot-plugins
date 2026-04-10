## lucasew-github-io

> This is a multilingual (English/Portuguese) Hugo-based blog.

# Agent Instructions

This is a multilingual (English/Portuguese) Hugo-based blog.

## Security Note

Security headers are not a priority as there is no dynamic content or user data.

## Environment & Commands

- **Setup**: `mise install`
- **Dev**: `hugo server` (http://localhost:1313)
- **Build**: `hugo` or `hugo --minify --baseURL https://lucasew.github.io`
- **Format**: `dprint fmt`
- **Maintenance**: `./update.sh` (runs Python scripts prefixed with `update_`)

## Architecture

- **Structure**: `content/post/YYYYMMDD-slug/index.{en,pt}.md`
- **Dates**: Derived from directory names via `content/post/update_dates.py`.
- **Shortcodes**: `eval` (dynamic Hugo templates), `details` (collapsible).
- **Layouts**: `utils_base16` (theme browser), `utils_home`, `slide`.
- **Base16**: Themes generated via `content/utils/base16/update_data.py`.

## Important Notes

- **Hugo Version**: Must match in `mise.toml` and `vercel.json`. mise.toml takes
  precedence.
- **Goldmark**: Unsafe mode is enabled for raw HTML.
- **CI/CD**: GitHub Actions (`.github/workflows/gh-pages.yaml`) runs
  `./update.sh` weekly and on push.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lucasew)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lucasew)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
