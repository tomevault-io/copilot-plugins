## demo-video-mcp

> Core Python code lives in `src/demo_video_mcp/`. Keep product-independent

# Repository Guidelines

## Project Structure & Module Organization

Core Python code lives in `src/demo_video_mcp/`. Keep product-independent
planning, validation, recording, storage, and MCP protocol logic there.
JSON schemas are under `src/demo_video_mcp/schemas/`.

Site-specific behavior belongs in `plugins/<plugin-id>/`, with configuration in
`plugin.json`, reusable guidance in `guides/`, and scenario examples in
`scenarios/`. `plugins/generic-web/` is the baseline implementation;
`plugins/protectgo/` is an optional product adapter. Tests live in `tests/`,
with browser fixtures in `tests/fixtures/`. Operational documentation is in
`docs/`, while `bin/demo-video-mcp` is the local launcher.

Do not commit runtime artifacts from `.demo-video-data/`, `.auth/`, `videos/`,
or `.venv/`.

## Build, Test, and Development Commands

```bash
python3 -m venv .venv
.venv/bin/pip install -e .
.venv/bin/playwright install chromium
```

These commands create the development environment, install the package in
editable mode, and install the supported browser runtime.

```bash
PYTHONPATH=src .venv/bin/python -m unittest discover -s tests -p 'test_*.py' -v
./bin/demo-video-mcp
```

The first command runs the complete test suite. The second starts the `stdio`
MCP server for Codex or Claude Code. Android recording additionally requires
`adb`, an emulator, FFmpeg, Appium, and the UiAutomator2 driver.

## Coding Style & Naming Conventions

Use four-space indentation and standard Python conventions: `snake_case` for
functions and variables, `PascalCase` for classes, and `UPPER_SNAKE_CASE` for
constants. Add type hints to public interfaces and keep functions focused.
Prefer existing validation, storage, and error types over new abstractions.
Core modules must not contain product URLs, selectors, or scenario content;
place those in plugins.

## Testing Guidelines

Tests use `unittest` and follow the `tests/test_*.py` naming pattern. Add
targeted regression tests for schema validation, approval boundaries, runtime
detection, and artifact handling. Keep unit tests deterministic. Tests that
require Chromium, Appium, an emulator, or an APK should be clearly isolated as
integration tests.

## Commit & Pull Request Guidelines

History uses short imperative subjects such as `Add brief-driven demo video MCP
server`. Keep commits scoped and avoid generated recordings. Pull requests
should explain behavior changes, list verification commands, link relevant
issues, and include screenshots or sample manifests when capture output
changes. Call out changes to schemas, approval semantics, or stored artifact
formats explicitly.

## Security & Configuration

Never commit credentials, browser profiles, APKs, or authenticated screenshots.
Use absolute paths and environment variables such as
`DEMO_VIDEO_ALLOWED_ROOTS`; do not embed secrets in MCP arguments or logs.

---
> Source: [rignore/demo-video-mcp](https://github.com/rignore/demo-video-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
