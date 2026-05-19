## llama-cpp-server

> OpenAI-compatible LLM inference server using the native llama-server binary from llama.cpp, with a thin CORS proxy and config-driven model presets.

# llama-cpp-server

OpenAI-compatible LLM inference server using the native llama-server binary from llama.cpp, with a thin CORS proxy and config-driven model presets.

## Tech Stack

- Native llama-server binary (built from llama.cpp source)
- Python 3 launcher (stdlib only, zero dependencies) — config reader + CORS proxy
- systemd user service for production

## Commands

```bash
# Dev
python3 launcher.py

# Production
systemctl --user start llama-cpp-server
systemctl --user restart llama-cpp-server
journalctl --user -u llama-cpp-server -f

# Install (clones llama.cpp, builds binary, prompts for CPU/CUDA/Metal)
./install.sh

# Update llama.cpp (rebuild with latest)
./install.sh

# Test API
curl http://localhost:8080/health
curl http://localhost:8080/v1/models
curl -X POST http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"MODEL_NAME","messages":[{"role":"user","content":"Test"}]}'
```

## Directory Structure

- `launcher.py` — Reads config.json, generates models.preset, starts llama-server + CORS proxy
- `config.json` — All configuration (server, model defaults, per-model overrides)
- `models.preset` — Generated at launch from config.json (gitignored)
- `bin/llama-server` — Compiled binary (gitignored, built by install.sh)
- `vendor/llama.cpp/` — llama.cpp source (gitignored, cloned by install.sh)
- `systemd/llama-cpp-server.service` — systemd user service unit
- `install.sh` / `uninstall.sh` — Setup and teardown scripts

## Architecture

- **Native binary**: llama-server handles all inference, model management, and OpenAI API endpoints
- **CORS proxy**: Thin Python `ThreadingHTTPServer` on port 8080 forwards to llama-server on port 8081, adding CORS headers
- **Model discovery**: llama-server scans `models_directory` for `.gguf` files via `--models-dir`
- **Model presets**: `launcher.py` converts `config.json` to a `models.preset` INI file consumed by `--models-preset`
- **Model aliases**: `model_settings` entries with a `"file"` field become preset sections with `model = /path/to/file.gguf`
- **Idle management**: `--sleep-idle-seconds` unloads models after idle timeout
- **Lazy loading**: `--models-autoload` loads models on first request (default)
- **Concurrent requests**: llama-server supports parallel slots and continuous batching

## API Endpoints

All endpoints are provided natively by llama-server:

- `GET /health` — Health check
- `GET /v1/models` — List available models
- `POST /v1/chat/completions` — Chat completions (OpenAI-compatible, streaming supported)
- `POST /v1/completions` — Text completions
- `POST /v1/embeddings` — Embeddings
- `POST /tokenize` — Tokenization
- `POST /detokenize` — Detokenization
- `POST /models/load` — Load a model
- `POST /models/unload` — Unload a model
- `GET /slots` — View active slots
- `GET /metrics` — Prometheus metrics

## Configuration

All via `config.json`. Launcher generates `models.preset` at startup.

- `server` section: host, port, cors_origins
- `model_manager` section: defaults for all models (n_ctx, n_gpu_layers, n_threads, etc.)
- `model_settings` section: per-model overrides keyed by model name (optional)
- `override_tensor`: MoE expert offload pattern (e.g., `.ffn_.*_exps.=CPU`)
- `offload_kqv`: KV cache placement (true=GPU VRAM, false=CPU RAM)
- Models directory: `/mnt/DataShare/Models/LLM`

### Config-to-preset mapping

| config.json field | INI key |
|---|---|
| `n_ctx` | `c` |
| `n_gpu_layers` | `n-gpu-layers` |
| `n_threads` | `t` |
| `override_tensor` | `override-tensor` |
| `offload_kqv: false` | `no-kv-offload = true` |
| `n_cpu_moe` | `n-cpu-moe` (MoE: keep first N layers' experts on CPU) |
| `cache_type_k` | `cache-type-k` (e.g. `q8_0` halves K cache vs fp16) |
| `cache_type_v` | `cache-type-v` (e.g. `q8_0` halves V cache vs fp16) |
| `parallel` | `np` (concurrent slots; lower = less recurrent memory) |
| `fit` | `fit` (`true`/`false`; auto-offload to keep model in VRAM) |

## Git Conventions

- Conventional commits: `feat:`, `fix:`, `refactor:` prefix
- Sentence case after colon, concise descriptions

## Gotchas

- Binary must be built first via `install.sh` — launcher checks for `bin/llama-server`
- CORS proxy on port 8080, llama-server on port 8081 (internal)
- `models.preset` is regenerated on every launch — edit `config.json`, not the preset
- Flash attention enabled by default
- `--models-max 1` limits to one loaded model (single GPU)
- No test suite exists

---
> Source: [Mahrkeenerh/llama-cpp-server](https://github.com/Mahrkeenerh/llama-cpp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
