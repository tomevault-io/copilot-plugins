## abs-mcp

> Build a high-quality Model Context Protocol server for Audiobookshelf. The server should expose safe, typed MCP tools and resources for inspecting and managing an Audiobookshelf instance, with repeatable tests against a local Docker Audiobookshelf fixture.

# Agent Instructions

## Project Goal

Build a high-quality Model Context Protocol server for Audiobookshelf. The server should expose safe, typed MCP tools and resources for inspecting and managing an Audiobookshelf instance, with repeatable tests against a local Docker Audiobookshelf fixture.

## Working Assumptions

- Prefer Go for the first implementation unless a later decision record changes that. This matches the nearby Audiobook Organizer ABS client and the official Go MCP SDK.
- Treat the public Audiobookshelf API docs as useful but not authoritative. They currently state that they are out of date and no longer maintained.
- Treat Audiobookshelf source code and verified Docker round-trip tests as the source of truth for behavior.
- The user cannot configure this corporate Codex instance with MCP servers, so local verification must not depend on registering this server in Codex.

## Tooling Preferences

- Use `rg` for content searches.
- Use `fd` or `find` for file discovery.
- Use `rtk` as the command prefix for shell commands in this workspace.
- Use `uv` for Python helper scripts and virtual environments if Python is added.
- Use `go test ./...` for Go verification once Go code exists.
- Use `docker compose` only through repo scripts or documented make targets so the ABS fixture remains resettable.

## Coding Discipline

- For non-trivial changes, state the working assumption and the smallest verifiable goal before editing.
- Keep edits surgical and traceable to the current task.
- Prefer simple, typed request/response structures over generic maps at MCP boundaries.
- Do not add speculative MCP tools. Add tools only when backed by an ABS endpoint, a fixture scenario, or a documented user workflow.
- Do not log bearer tokens, API keys, cookies, or raw Authorization headers.
- Do not persist user credentials in committed files.
- Do not mutate files in `/Users/Shared/Docker/audiobook-organizer` unless explicitly asked. That fixture can be read and invoked as an external dependency while planning.

## MCP Design Rules

- Tools perform actions or bounded queries.
- Resources expose readable state snapshots such as server info, libraries, item summaries, generated API inventory, and fixture status.
- Prompts should be limited to repeatable operator workflows such as library audit, scan troubleshooting, or API update review.
- Tool names should be stable, explicit, and namespaced with `abs_`.
- Each tool schema must define required inputs, optional inputs, output shape, error cases, and whether it can mutate ABS state.
- Always target token-use efficiency when it does not compromise server usability. Prefer bounded result sets, compact summaries, truncation flags, and opt-in raw/detail tools over returning large raw ABS payloads by default.
- Mutating tools must be opt-in and should require IDs rather than fuzzy names where possible.
- Destructive tools, including delete, remove, purge, overwrite, replace, and broad batch operations, must require an explicit confirmation input. Prefer an exact phrase that includes the relevant ABS ID, plus expected-count checks when affected records can be previewed.
- Long operations such as scans should return job/status information and provide a separate status/read tool instead of blocking indefinitely.

## Release and Registry Rules

- Keep `internal/version.Version`, `server.json` top-level `version`, and release tags aligned.
- For MCP Registry OCI packages, put the package version only in the image tag inside `packages[].identifier`.
- Do not add `packages[].version` when `packages[].registryType` is `oci`; the MCP Registry rejects that shape even if local JSON checks pass.
- Run `python3 scripts/validate_server_json.py` or `prek run validate-server-json --all-files` before release metadata changes.
- Tag releases as `vX.Y.Z`; tag pushes trigger the release workflow, GHCR image publish, MCP Registry publish, and GitHub release creation.
- When publishing or editing GitHub release notes, include a container image section that links to the versioned GHCR package tag and shows the `docker pull ghcr.io/jeeftor/abs-mcp:<version>` command.

## Audiobookshelf API Update Workflow

- Maintain an API inventory generated from Audiobookshelf source, especially `server/routers/ApiRouter.js` and controller methods under `server/controllers/`.
- Compare generated inventory changes before updating MCP tools.
- For changed endpoints, update typed client methods, MCP tool schemas, docs, and round-trip tests together.
- Validate against the Docker ABS fixture before marking an API update complete.

## Test Fixture

The repo-local fixture at `test/abs` provides:

- `docker-compose.yml` with plain and metadata-enabled Audiobookshelf instances.
- Reset and scan scripts.
- Compose project name `abs-mcp`.
- Unique container names `abs-mcp-abs-plain` and `abs-mcp-abs-metadata`.
- Known ports: `13388` for plain and `13389` for metadata-enabled.
- Test credentials and tokens through its `.env.testing`/baseline setup.

## Verification

- Prefer repo-native verification first.
- Every new MCP feature must be backed by matrix coverage. Add or update the relevant unit, protocol, and Docker ABS fixture tests so CI proves the feature through the same matrix used for releases.
- For docs-only changes, run a quick file review and `git status`.
- For API client changes, add or update unit tests with `httptest`.
- For MCP protocol behavior, add tests that spawn the MCP server over stdio and issue JSON-RPC/MCP requests directly.
- For ABS behavior, run the resettable Docker fixture with `make abs-ci-smoke` or `make abs-dev-reset-scan`.

---
> Source: [jeeftor/abs-mcp](https://github.com/jeeftor/abs-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
