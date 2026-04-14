## palette-mcp-server

> This is a repository that contains the code for the Palette MCP server. The Palette MCP server is a tool for interacting with Palette through the Model Context Protocol (MCP). The MCP server is designed to be deployed and ran as a container. UV is used to manage the Python dependencies and start the server.

# Overview

This is a repository that contains the code for the Palette MCP server. The Palette MCP server is a tool for interacting with Palette through the Model Context Protocol (MCP). The MCP server is designed to be deployed and ran as a container. UV is used to manage the Python dependencies and start the server.

## Repository Structure

- `src/`: Contains the Python code for the Palette MCP server.
- `tests/`: Contains unit tests and end-to-end tests for the Palette MCP server.
- `scripts/`: Contains internal scripts for development and debugging.
- `public/`: Contains public assets for the Palette MCP server.
- `tmp/`: Contains temporary files that developers use and can store without worrying about committing to the repository.

## Development

- Use Taskfile for quick and pre-established tasks.

## E2E Tests

The end-to-end test suite runs 13 isolated LLM-agent steps against a live Palette environment and a KinD cluster provisioned by Terraform.

**Run locally:**

```shell
task e2e
```

**Clean up if something goes wrong:**

```shell
task e2e-cleanup
```

**Required environment variables** (set in `.env` or the shell):

| Variable | Description |
|---|---|
| `SPECTROCLOUD_APIKEY` | Palette API key |
| `SPECTROCLOUD_HOST` | Palette API host (defaults to `api.spectrocloud.com`) |
| `SPECTROCLOUD_PROJECT_UID` | Palette project UID |
| `SPECTROCLOUD_DEFAULT_PROJECT_ID` | Palette default project ID |
| `OPENAI_API_KEY` | OpenAI API key (used by the tester agent and judge) |

Test coverage is tracked in `tests/e2e_coverage.md`.
Infrastructure helpers (Terraform, kubectl, cluster polling, smolagents utilities) live in `tests/e2e_helpers.py`.
Test definitions, the LLM judge, and the main entrypoint live in `tests/e2e.py`.

## General Guidelines

- Comments should be complete sentences and end with a period.
- Review the `pyproject.toml` file to understand what dependencies are installed and what versions are used.
- If you need to update a dependency, update the `pyproject.toml` file and run the `uv sync` command to update the `uv.lock` file. Also use the bulwark_scan_project tool to scan the project for compromised packages.
- DO NOT UPDATE THE `uv.lock` file manually. It is automatically updated by the `uv sync` command.
- DO NOT UPDATE DEPENDENCIES UNLESS EXPLICITLY REQUESTED.
- DO NOT CREATE MANUAL GIT TAGS. They are automatically created by CI/CD release workflow, thanks to Goreleaser.

## Commit Messages and Pull Requests

- Follow the [Chris Beams](https://chris.beams.io/posts/git-commit/) style for commit messages.
- Use the angular commit message format. Such as: `fix: <description>`, `feat: <description>`, `refactor: <description>`, `test: <description>`, `docs: <description>`, `chore: <description>`.

- Every pull request should answer:
  - What changed?
  - Why?
  - Breaking changes?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spectrocloud) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
