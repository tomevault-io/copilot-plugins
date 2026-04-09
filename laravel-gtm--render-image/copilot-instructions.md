## render-image

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
uv sync                              # Install dependencies
uv run playwright install chromium   # Install browser (required once)
uv run pytest                        # Run all tests
uv run pytest tests/test_foo.py      # Run a single test file
uv run pytest -k "test_name"         # Run a single test by name
uv run ruff check .                  # Lint
uv run ruff format .                 # Format
uv run ty check                      # Type check
```

CLI usage: `echo '{"html":"<h1>Hello</h1>"}' | uv run python main.py`

## Architecture

Framework-agnostic PDF renderer: HTML in, PDF bytes out via Playwright/Chromium. Designed to be called from Laravel (Spatie PDF driver) across multiple runtimes.

**Core layer** (`render_image/`):
- `contracts.py` — `RenderRequest` (input) and `RenderResult` (output) dataclasses with `from_payload()` factory for JSON deserialization and validation
- `service.py` — `render_pdf()` (sync) and `render_pdf_async()` (async) entrypoints. Uses `PlaywrightRenderer` by default; accepts any `Renderer` protocol implementation for testing
- `errors.py` — `RenderError` (runtime) and `ValidationError` (input) exceptions
- `__init__.py` — Public API re-exports: `RenderRequest`, `RenderResult`, `render_pdf`, `render_pdf_async`

**Adapter layer** (`render_image/adapters/`):
- `aws_lambda.py` — API Gateway proxy event handler, returns base64-encoded PDF response
- `container.py` — Reads JSON from stdin/file, writes JSON to stdout. Also exposes `handle_payload()` used by other adapters
- `cloudflare.py` — Thin wrapper over container adapter; unwraps optional `render_request` key from Worker payloads

All adapters funnel through the same `render_pdf()` call. The Cloudflare adapter delegates to container's `handle_payload()`.

**Infrastructure** (`infrastructure/`): AWS CDK stack deploying the Lambda container behind HTTP API Gateway. Has its own `CLAUDE.md` with CDK-specific commands and context.

**CI**: A single `release.yml` workflow runs the full release pipeline on tag push (`v*.*.*`) — creates a GitHub Release with auto-categorized notes (configured in `.github/release.yml`), builds and pushes the Docker image to GHCR (with Lambda image tarball attached to the release), then copies the image to ECR and runs `cdk deploy`. A separate `deploy-aws.yml` provides a manual `workflow_dispatch` trigger for re-deploying a previously published image. All deployment config (region, image name, stack name, Lambda memory/timeout) is driven by GitHub Actions variables and CDK context.

## Key Constraints

- Python >=3.14, managed with `uv`
- `format` and `paper_size` are mutually exclusive on `RenderRequest`
- `render_pdf()` detects active event loops and raises if called inside one — use `render_pdf_async()` instead
- HTML payload max size: 2MB; timeout range: 1–120000ms
- Supported `format` values: a0–a6, legal, letter, tabloid, ledger (case-insensitive)
- Lambda mode auto-detected via `AWS_LAMBDA_FUNCTION_NAME` env var — launches Chromium with restricted flags (no-sandbox, single-process, etc.)
- The `Renderer` protocol in `service.py` is the test seam — pass a custom implementation to `render_pdf()`/`render_pdf_async()` to avoid launching a real browser

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laravel-gtm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/laravel-gtm)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
