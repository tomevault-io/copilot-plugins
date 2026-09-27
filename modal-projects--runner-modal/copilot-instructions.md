## runner-modal

> Concise always-on rules for this repo. Prefer Modal SDK shapes, idiomatic Python, and library-native APIs. Do not treat this as a product README. Code is liability: keep it small; complexity only when warranted.

# Agent rules — runner-modal

Concise always-on rules for this repo. Prefer Modal SDK shapes, idiomatic Python, and library-native APIs. Do not treat this as a product README. Code is liability: keep it small; complexity only when warranted.

## Naming

- Prefer short, concrete nouns and verbs. Say what the thing **is** or **does** in one pass.
- **No underscore-prefix for “privacy.”** Types, module constants, methods, and test helpers are normal names; omit from `__all__` if internal. Do not invent `_Foo`, `_helper`, `_post`.
- **Do not shadow Modal / stdlib names.** Never call our FastAPI control plane `App` (that is `modal.App`). Prefer `WebhookApp`, `DeliveryStore`, etc.
- **Avoid redundant / encoding noise** in names: no `…Helper`, `…Manager`, `…Utils`, `…Sync` (ambiguous), `…Data`, `FooBarBazResponse` when `FooResponse` / `JitResponse` is enough. Spell units in constants (`DELIVERY_TTL_SECONDS`, not `DELIVERY_TTL_S`).
- **Tag / env keys:** name the *key* `…_TAG` (e.g. `KIND_TAG`, `POOL_TAG`) and the *value* plainly (`JOB_KIND`). Do not use `TAG_KIND` + `TAG_KIND_VALUE`.
- **Async boundary:** `async` route reads I/O; sync worker has a clear verb (`process_webhook`), not `…_sync`.
- Temporary diagnosis / repro hooks stay out of product PRs; use a throwaway local branch or one-off workflow for testing, not names that look permanent.
- Match Modal twin vocabulary for public entities (`create` / `from_name` / `objects` / `ephemeral` / `hydrate`); do not invent parallel jargon for the same idea.

## Public DX

- Public surface is entity-based: `Runner` + nested `Runner.Job`. Prefer Modal SDK vocabulary (`create` / `from_name` / `objects` / `ephemeral`; Job ≈ Sandbox).
- Export only via `__all__`. Do not underscore-prefix types for “privacy”; omit them from `__all__` instead.
- No free public helpers (`spawn`, `parse_labels`, pools, ASGI attach helpers). If a concern is a lead responsibility, put it on an entity (or a method on the owning entity).
- One `Runner.create` per App. Control plane is a Runner-registered Function `@modal.asgi_app` named `webhook` with `@modal.concurrent`. Runner identity is `RUNNER_MODAL_NAME`. Inputs queue on cold start — do not use Modal Server for GitHub webhooks (503 on zero→one).
- Soft capacity (`has_capacity` / `max_concurrent`) is **soft** (list-then-create TOCTOU). Document it as soft; never present it as a linearizable lock.
- **Job resources:** Modal-twin flat kwargs — `cpu`, `memory`, `gpu`, `experimental_options` (e.g. `{"vm_runtime": True}` for Docker/VM). Do **not** invent `ResourceSpec | DockerResources`, xor validators, or `isinstance` resource dispatch. Prefer `docker_image()` when `vm_runtime` is set; let Modal enforce GPU vs VM limits.
- **Admission:** `repositories` is required (non-empty). `admit_reason(labels, repository)` fail-closed. No admit-all mode. `Job.create` enforces the same rules.

## Explicit config (Modal twin)

- Public primitives take kwargs / `modal.Secret` handles. No ambient `os.environ.get` for credentials or product config on `Runner` / `Job` / `WebhookApp`.
- Clients and scripts may use env / Modal CLI tokens. Optional `client=` like Modal entities.
- In-container entrypoints (`webhook()` ASGI entry, `python -m runner_modal.entrypoint`) may read only values **mounted** via `secrets=` / `env=` at create time; KeyError if missing.
- Split Secrets: `github_secret` (`GITHUB_TOKEN`, Jobs only) and `webhook_secret` (`WEBHOOK_SECRET`, webhook Function only). Never co-mount webhook into Jobs.

## Python construction & style

- Normal `__init__` + classmethod factories. No blocked constructors, no `object.__new__`, no `getattr` / `hasattr` / `setattr`, no dynamic class creation / `__name__` mutation.
- No process-local registries or fake idempotency caches — use Modal Dict (and claim keys properly).
- Call Modal / FastAPI / httpx with real kwargs. No build-dict / strip-Nones / `**kwargs` bags.
- Prefer library-native APIs (Modal, FastAPI, Pydantic, httpx, tenacity, uv Image methods).
- Composition over inheritance: wrap `Sandbox` / `Dict` / `Volume`; do not subclass Modal types for product API.
- Frozen Pydantic models for boundary DTOs / snapshots (`ConfigDict(frozen=True)` preferred).
- EAFP at mutation boundaries (`Job.create` raises). Optional LBYL helpers must be labeled soft.
- Break import cycles with lazy imports at Modal lifecycle boundaries (`@modal.enter`), not circular top-level imports.

## Errors, HTTP, secrets

- Soft absence → `None` or HTTP 204 (e.g. undeployed `url`, ignored webhook). Failures raise a small set: `ValueError`, `LookupError`, `AuthError`, `ConcurrencyLimitError`. Base `RunnerError` is rare.
- Do not wrap Modal / httpx failures in `RunnerError`. Propagate; map only product/auth cases.
- FastAPI: HTTP status conveys success. Response models without `ok: bool`. Use `HTTPException` for 401 / 400 / 503.
- Sync I/O (Modal SDK, httpx): use sync `def` endpoints or `asyncio.to_thread`. Never block `async def` handlers with sync Modal/httpx calls.
- Credentials via required named Secrets on `Runner.create(github_secret=…, webhook_secret=…)` and `Job.create(github_secret=…)`. Never put `GITHUB_TOKEN`, `WEBHOOK_SECRET`, or JIT strings in Sandbox `env=` dicts.
- JIT mint runs only inside the Job (`python -m runner_modal.entrypoint`) from `GITHUB_TOKEN` injected by `github_secret=`. `WebhookApp` takes an explicit `webhook_secret=`; ASGI entry `webhook()` requires mounted `WEBHOOK_SECRET` / `RUNNER_MODAL_NAME` (KeyError if missing).
- Verify GitHub webhooks with `hmac.compare_digest` on the raw body before parse. Require `X-GitHub-Delivery` (no job-id fallback).

## Concurrency & shared state

- Webhook idempotency: **claim** the delivery ID before side effects (`put` / `skip_if_exists`). Bind `object_id` after successful create before relying on TTL reclaim. Never release after successful create unless terminate is confirmed.
- Soft list-based capacity is enough — never fake linearizable concurrency locks.
- Shared Modal Dict writes: prefer conditional writes for init and idempotency keys. Redeploy overwrites Runner meta (last deploy wins).
- Do not full-scan / trim entire Dict stores on every request without an explicit GC strategy (separate hot path from opportunistic cleanup).

## Layout, images, tests

- Modules by responsibility: `runner` / `meta` / `entrypoint` / `webhook` / `exceptions`. Keep nested `Job` if it matches the Modal twin.
- Images: install runtime deps with ``uv_pip_install(*CONTROL_PLANE_DEPS|JOB_DEPS)`` and bake ``add_local_python_source("runner_modal", copy=True)`` until a PyPI release. ``PACKAGE_SPEC`` documents the operator ``uv add`` pin. Named Job Image ``"{name}-job"`` is published at ``Runner.create``; webhook ``Job.create`` uses ``Image.from_name``. No Path / ``add_local_dir``.
- Default `cache=False`. Shared `/cache` Volume is same-pool only — not Actions cache.
- One unit test file per impl file (`test_runner.py`, `test_entrypoint.py`, `test_webhook.py`, `test_exceptions.py`, `test_meta.py`).
- Test HMAC failure, delivery claim, admission (repo + labels), and capacity semantics — not only exports / signatures.
- Retries (tenacity): transient network / timeouts / 5xx only. Never retry 401 / 403 / validation errors.

## Never do

- Process-local `_REGISTRY` / create caches / fake idempotency
- Underscore-prefixing types or helpers for privacy (`_Foo`, `_helper`) — omit from `__all__` instead
- Naming our types `App` (conflicts with `modal.App`) or other Modal entity names
- `getattr` / `hasattr` / `object.__new__` / blocked `__init__` / mutating `__name__` for entity identity
- None-filtered `**kwargs` bags into Modal APIs
- Tagged resource unions / xor validators / `sandbox_kwargs` helpers instead of Modal flat kwargs
- `ok: bool` on HTTP response models
- Blanket `except Exception: raise RunnerError(...)`
- Exception taxonomy for every Modal failure mode
- Tokens or JIT in `env=`; `os.environ.get` credential fallbacks; minting JIT in the parent process; co-mounting `WEBHOOK_SECRET` into Jobs
- Blocking the asyncio event loop with Modal / httpx in `async def`
- Claim-after-create webhook deliveries; reclaiming pending claims that already have `object_id`
- Treating soft capacity as a hard reservation
- Inheriting Modal SDK types for the product API
- Free public helper functions as the primary DX
- Path / `add_local_dir` instead of uv-native Image install
- Substituting export/signature tests for security and race/idempotency tests
- Admit-all repositories / empty `repositories` on `Runner.create` or `objects.create`

## Workspace

- This repo’s GitHub Actions CI and Release workflows run on ``ubuntu-latest``.
- Maintainer CI App is `scripts/ci_app.py` (App `runner-modal-ci-app`) for dogfooding Jobs, not for repo CI.

---
> Source: [modal-projects/runner-modal](https://github.com/modal-projects/runner-modal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
