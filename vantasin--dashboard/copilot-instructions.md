## dashboard

> - Root contains `docker-compose.yml`, `.env` (local only), and helper scripts like `git_push.py` and `summarize_codebase.sh`.

# Repository Guidelines

## Project Structure & Module Organization
- Root contains `docker-compose.yml`, `.env` (local only), and helper scripts like `git_push.py` and `summarize_codebase.sh`.
- All Homepage configuration lives under `config/` (e.g., `bookmarks.*`, `services.*`, `settings.*`, `widgets.*`, `docker.*`).
- Example files (`*.example`) are the templates you should edit and then copy to `*.yaml` for local use; YAML files and `.env` are intentionally git-ignored.

## Build, Test, and Development Commands
- `docker compose up -d` — start or update the Dashboard stack in detached mode.
- `docker compose down` — stop and remove the stack (containers only).
- `docker compose logs homepage` — inspect the Homepage container logs for troubleshooting.
- `docker compose config` — validate the compose configuration before pushing.
- `./summarize_codebase.sh .` — optional helper to generate a local `codebase_summary.txt` (not committed).

## Coding Style & Configuration Conventions
- Prefer small, focused changes in `config/*.example` and keep `*.yaml` environment-specific.
- YAML: two-space indentation, lowercase keys with hyphen-separated words where possible.
- Python helpers use 4-space indentation and `snake_case` for functions; shell scripts follow existing style and use descriptive variable names.

## Testing Guidelines
- There is no automated test suite; rely on Docker commands for validation.
- Before opening a PR, run `docker compose config` and bring the stack up locally with `docker compose up -d` to ensure it boots cleanly.
- Manually verify the Dashboard UI loads and configured services/widgets appear as expected.

## Commit & Pull Request Guidelines
- Use short, descriptive, sentence-style commit messages (e.g., `Update services example for NPM`, `Add new bank icon`).
- Group related configuration and asset changes in a single commit; avoid mixing unrelated updates.
- PRs should include: a concise summary, a brief description of config or icon changes, any relevant screenshots of the Dashboard, and notes on how you validated the stack.

## Security & Secrets
- Never commit real secrets or host-specific details; `.env` and `config/*.yaml` are local-only and must not be added to Git.
- When sharing examples, sanitize domains, IPs, and tokens, and prefer updates to `*.example` files instead of real YAMLs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vantasin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Vantasin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
