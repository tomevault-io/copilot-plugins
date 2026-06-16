## gitlab-issues-to-github

> This project pulls information about Gitlab issues from a repository and mirrors the issue to Github.

# Gitlab Issues to Github Development Skill

## Project Context

This project pulls information about Gitlab issues from a repository and mirrors the issue to Github.

## Key Commands

- `uv sync` - Install dependencies
- `uv run pytest` - Run tests
- `uv run pre-commit run --all-files` - Run linters
- `uv add <package>` - Add dependency
- `uv add --dev <package>` - Add dev dependency

## Project Structure

```bash
main.py  # Main code
```

## Development Workflow

1. Make changes
2. Run pre-commit hooks automatically on commit
3. Tests run via uv run python script
4. Modify github actions
5. Update README.md to remove non-relevance.

## Important Notes

- Always support a CLI interface
- Use uv best practices

## Configuration Files

Refer to pyproject.toml and .pre-commit-config.yaml in this directory for exact setup.

---
> Source: [fartbagxp/gitlab-issues-to-github](https://github.com/fartbagxp/gitlab-issues-to-github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
