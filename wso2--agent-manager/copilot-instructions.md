## agent-manager

> A **Kubernetes init container** (Docker image only — no Python package) that injects Python auto-instrumentation into application pods. It copies the instrumentation packages into a shared volume that the app pod mounts onto its `PYTHONPATH`, so the app is instrumented with zero code change.

# python-instrumentation-provider — agent guide

A **Kubernetes init container** (Docker image only — no Python package) that injects Python auto-instrumentation into application pods. It copies the instrumentation packages into a shared volume that the app pod mounts onto its `PYTHONPATH`, so the app is instrumented with zero code change.

## Files

| File | Role |
|---|---|
| `Dockerfile` | builds the init-container image; installs the matching `locks/` file, runs `setup-instrumentation.py` |
| `setup-instrumentation.py` | copies packages from `/instrumentations/{INSTRUMENTATION_PROVIDER}` → shared volume `/otel-tracing-sdk`, incl. `sitecustomize.py` |
| `sitecustomize.py` | minimal bootstrap auto-imported by the app's Python: sets Traceloop env vars and calls `Traceloop.init()` |
| `Makefile` | build / load image; `make lock` (regenerate locks) / `make check-locks` (hermetic verify) |
| `requirements.in` | loose dependency inputs (extras beyond `traceloop-sdk`); compiled into `locks/` |
| `locks/traceloop-<T>-python<X.Y>.txt` | fully-pinned dependency tree per `(traceloop × python)`; the image installs these |
| `scripts/gen-locks.sh` | regenerates `locks/` (re-resolves live PyPI in `python:<X.Y>-slim`) |
| `scripts/check-locks.sh` | hermetic CI check that committed locks cover the release-config matrix |
| `RELEASING.md` | release runbook |

## How injection works

1. Init container runs `setup-instrumentation.py`, which copies the provider's packages + `sitecustomize.py` into an `emptyDir` volume at `/otel-tracing-sdk/`.
2. The app pod mounts the same volume and puts it on `PYTHONPATH`.
3. Python auto-imports `sitecustomize.py` at startup → `Traceloop.init(telemetry_enabled=False, api_endpoint=AMP_OTEL_ENDPOINT, headers={...})`.

`sitecustomize.py` reads `AMP_OTEL_ENDPOINT` + `AMP_AGENT_API_KEY`; it also sets `TRACELOOP_TRACE_CONTENT` (default `true`), `TRACELOOP_METRICS_ENABLED=false`, and `OTEL_EXPORTER_OTLP_INSECURE=true`.

## Commands

```bash
make build           # docker build (PYTHON_VERSION, TRACELOOP_VERSION args) → ghcr.io/wso2/amp-python-instrumentation-provider:TAG
make docker-load-k3d # build + load into k3d (K3D_CLUSTER_NAME=openchoreo-local-setup)
make help
```

Build args: `PYTHON_VERSION` (default 3.11), `TRACELOOP_VERSION` (default 0.60.0). Default `TAG=dev-python3.11`.

## Gotchas

- `sitecustomize.py` must land at the **root** of `/otel-tracing-sdk/` for Python to auto-import it.
- `OTEL_EXPORTER_OTLP_INSECURE=true` is hard-coded (no TLS validation) — expected for in-cluster export.
- **Fail-open by design (diverges from the library contract).** This `sitecustomize.py` wraps its entire body in `try/except Exception` and only logs (`logger.exception`) — it never re-raises. So if `AMP_OTEL_ENDPOINT`/`AMP_AGENT_API_KEY` are missing (it raises a `ValueError`), or Traceloop import/init fails, instrumentation is silently skipped and **the host app keeps running**. This is intentional: an init container must not crash the app it instruments. Note this differs from `../libs/amp-instrumentation/AGENTS.md`, where `initialize_instrumentation()` raises `ConfigurationError`/`ImportError` on missing required vars. If you change this file, preserve the swallow-and-log behavior unless you deliberately want startup to hard-fail.
- No tests — it's a build/copy script.

Ships the package documented in `../libs/amp-instrumentation/AGENTS.md`.

---
> Source: [wso2/agent-manager](https://github.com/wso2/agent-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
