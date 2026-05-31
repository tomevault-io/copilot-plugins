## agentnetworkprotocol

> - Root contains protocol specifications and white papers (e.g., `01-*.md`, `06-*.md`, `07-*.md`, `08-*.md`).

# Repository Guidelines

## Project Structure & Module Organization

- Root contains protocol specifications and white papers (e.g., `01-*.md`, `06-*.md`, `07-*.md`, `08-*.md`).
- `docs/` and `docs/chinese/` hold guides, links, and community operations.
- `chinese/` mirrors core documents in Chinese plus research notes and process docs.
- `blogs/` and `blogs/cn/` store long-form articles; `blogs/images/` holds blog assets.
- `images/` and `standard/` provide shared figures and standards references.
- `examples/` contains sample ADP assets and API interface YAMLs.
- `scripts/` includes small Python utilities for repo maintenance.

## Build, Test, and Development Commands

This repository is documentation-first and does not have a build system or automated test suite.

- Run maintenance scripts with UV:
  - `uv run python scripts/add_copyright.py`
  - `uv run python scripts/rename_images.py`
  - `uv run python scripts/replace_spaces_with_hyphens.py`

## Coding Style & Naming Conventions

- Documentation is Markdown; keep headings concise and sectioned with `##`/`###`.
- Use descriptive, kebab-case filenames for new documents and assets when possible.
- For Python scripts, follow Google Python Style; keep comments and logs in English.
- Prefer clear, action-oriented titles like `add`, `update`, `docs:` in commit messages.

## Testing Guidelines

- No automated tests are defined. For changes, manually verify:
  - Links resolve inside `README.md` and `docs/`.
  - Images referenced in Markdown exist under `images/` or `blogs/images/`.

## Commit & Pull Request Guidelines

- Commit messages are short and imperative, commonly `add ...`, `update ...`, or `docs: ...`.
- PRs should include a clear description of the change and link related issues if any.
- For content updates, mention affected document paths and provide before/after context.

## Security & Configuration Tips

- Do not add secrets or credentials to Markdown, images, or examples.
- If publishing new specs, ensure external references are stable and properly attributed.

## Agent-Specific Instructions

- Respond in Chinese in CLI interactions.
- Use UV for Python dependency management and run scripts via `uv run`.
- Keep code comments and logging in English.

---
> Source: [agent-network-protocol/AgentNetworkProtocol](https://github.com/agent-network-protocol/AgentNetworkProtocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
