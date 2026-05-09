## machine-payments

> This repository contains sample implementations for machine-payments flows, including:

# AGENTS.md

## Repository purpose
This repository contains sample implementations for machine-payments flows, including:
- `mpp` examples
- `x402` examples

Current examples include Node/TypeScript and Python server samples.

## General guidelines
- Prefer minimal, sample-quality code over heavy abstractions.
- Keep implementations across languages structurally aligned when practical.
- Favor consistency in endpoint naming and behavior across comparable samples.
- Preserve existing sample conventions unless there is a clear reason to change them.

## Endpoint conventions
- For payment-protected sample endpoints, prefer `GET /paid` unless a sample explicitly demonstrates a different route for protocol-specific reasons.
- Keep README test commands aligned with the implemented endpoint.

## Python samples
- Prefer `FastAPI` + `uvicorn` for Python HTTP server samples.
- Prefer `uv` with a local `pyproject.toml` for dependency management.
- Use `python-dotenv` when environment-based setup is needed.
- Keep code straightforward and easy to compare with the TypeScript equivalent.

## TypeScript samples
- Prefer small, directly runnable examples.
- Keep `tsconfig.json` minimal unless stricter settings are required.
- Ensure sample code passes the local typecheck/build script.
- When SDK preview fields have incomplete typings, use narrow, localized type assertions rather than broad `any`.

## Stripe and payment setup
- Reuse the Stripe app info used elsewhere in the repo when adding new samples.
- Validate required environment variables at startup with clear error messages.
- When a server creates dynamic deposit addresses, cache and validate them before accepting them back from client-submitted credentials.

## Documentation expectations
- Each sample directory should include a concise `README.md` with:
  1. what the sample demonstrates,
  2. requirements,
  3. setup steps,
  4. run command,
  5. a simple test command.
- Keep setup instructions consistent with the language ecosystem used by that sample.

## When making changes
- Update related README commands/examples if routes or filenames change.
- Prefer small, targeted edits over repo-wide refactors.
- If changing one sample for consistency, check the sibling sample in the other language too.

---
> Source: [stripe-samples/machine-payments](https://github.com/stripe-samples/machine-payments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
