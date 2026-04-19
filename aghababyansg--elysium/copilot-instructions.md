## elysium

> - Treat sections marked **MUST** as non-negotiable for this repository unless the user explicitly overrides them in chat.

# Cursor rules — Elysium (VLA image editing)

## How to use this file

- Treat sections marked **MUST** as non-negotiable for this repository unless the user explicitly overrides them in chat.
- Prefer fixing root causes over workarounds.
- Scope: first-party code under `src/elysium/`, `scripts/`, and `tools/`. Do not edit generated or third-party trees (e.g. `unsloth_compiled_cache/`, vendored copies).

---

## MUST — From project decisions (always apply here)

### 1. Errors: fail fast (Python)

- Do not use `try`/`except` to swallow failures, return defaults, or substitute control flow. Invalid state should raise or fail an `assert`.
- Prefer letting exceptions propagate. Use `assert` for invariants and preconditions where appropriate.
- Do not add broad `except Exception:` (or similar) in project code to “keep going.”
- First-party Python: avoid `except` blocks unless the user explicitly asks for a narrow, justified handler; prefer no `except` at all.
- TypeScript/JavaScript rules below still allow `try`/`catch` where required for async/UI (different stack).

### 2. Logging: Loguru only (Python)

- Use **`from elysium.log import logger`** (Loguru). Do not use stdlib **`logging`** in new or refactored project code.
- Use the right method for severity: `logger.debug`, `logger.info`, `logger.warning`, `logger.error`, etc.
- Sinks are configured in **`src/elysium/log.py`** via **`configure_logging()`** (called from `elysium` package init). **`colorize=True`** is required so levels are visually distinct (e.g. errors show in red).
- Use Loguru-style messages with **`{}`** placeholders, e.g. `logger.info("Loading {}", path)`, not `%` formatting.

### 3. Log line context (caller)

- Caller context (**repo-relative file path** and **function name**) is injected automatically by the patcher in **`src/elysium/log.py`**. Do not manually prefix every message with file/function; keep messages short and factual.

---

## Project context

Python-first app: vision–language–action model for image manipulation (chunks of edit actions, canvas execution, training/inference). Code should stay clear, typed, and suitable for ML workflows.

---

## Repository layout (actual)

```
src/elysium/
  engine/     # Canvas / execution
  model/      # train, predict, RL, reward
  schemas/    # Actions / pydantic-style models
  log.py      # Loguru setup (MUST use from app code)
scripts/      # CLI entrypoints (train, infer, prepare_data, …)
tools/        # Auxiliary tools
configs/      # YAML configs
```

If `elysium.data` or other packages appear, keep the same logging and error-handling MUST rules.

---

## Python — general

- Python 3.10+: use modern syntax (`match`, `X | Y` unions).
- Type-hint public functions; avoid bare `Any` unless necessary.
- Use **`pathlib.Path`** for filesystem paths (not `os.path` for new code).
- Prefer dataclasses or Pydantic for structured data over untyped dicts.
- Export public names with **`__all__`** where it helps clarity.

---

## Python — ML / models

- Separate load vs run (e.g. load at startup, not per request in a server).
- Inference: **`torch.inference_mode()`** or **`torch.no_grad()`** as appropriate.
- Respect device and dtype (CUDA vs CPU; optional FP16 via config when relevant).
- Validate tensor/array shape, dtype, and value range before model entry.
- Do not reload large models on every call; cache where applicable.

---

## Python — images (NumPy / OpenCV / PIL)

- Internal processing: float32 where possible; uint8 for I/O or storage as needed.
- Document expected shape `(H, W, C)`, dtype, and value range in image helpers.
- No in-place mutation of caller-owned image arrays unless the API is explicitly documented that way; prefer returning new arrays.
- Prefer vectorized ops; avoid tight Python loops over pixels.

---

## Error handling (reconciled with MUST)

- **Project Python code:** follow MUST “fail fast” — no silent recovery via `except`.
- **Wrapping third-party errors:** only when the user asks for a specific boundary (e.g. API layer) and the handler re-raises or maps to a typed project exception without hiding the cause (`raise … from e`).
- Custom exception types can live under something like `src/elysium/exceptions.py` when you introduce API or domain boundaries.

---

## Logging (detail)

- **`LOGURU_LEVEL`** env var overrides default level (see `configure_logging`).
- Scripts should rely on `import elysium` (or any `from elysium…` import) so logging is configured once.
- For HTTP services later: you may add a second sink (e.g. JSON) in `log.py` without removing colored stderr for local dev.

---

## TypeScript / JavaScript (if used)

- Strict TypeScript; named exports unless framework requires default.
- Async: handle errors with `try/catch` or `.catch()` — this does not override Python MUST rules.

---

## API, security, performance, tests (guidelines)

- REST-style routes, validated uploads (size, magic bytes), no secrets in code.
- Rate-limit and cap image dimensions for untrusted input.
- Cache models; batch when useful; profile before micro-optimizing.
- Test image utilities across shapes/dtypes and edge cases; control seeds for generative checks.

---

## Style

- ~100 columns; format/lint with **ruff** / **black** for Python.
- Commits: `type(scope): description` (e.g. `feat(model): add RL checkpoint save`).

---

## Anti-patterns (project)

- Catching exceptions in Python to log and continue without re-raise (violates MUST).
- `print()` for diagnostic output in committed code — use **`logger`** from **`elysium.log`**.
- Stdlib **`logging.getLogger`** in `src/elysium`, `scripts/`, or `tools/`.
- Editing **`unsloth_compiled_cache/`** or other generated/vendor code to “fix” style — change first-party code or config instead.
- Loading huge models inside hot request paths.
- Pixel-level Python loops on full images.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AghababyanSG) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
