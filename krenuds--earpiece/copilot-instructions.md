## earpiece

> Two Docker microservices for voice I/O — text-to-speech and speech-to-text — exposed as plain HTTP APIs. Independent resource; not tied to any specific consumer.

# earpiece

Two Docker microservices for voice I/O — text-to-speech and speech-to-text — exposed as plain HTTP APIs. Independent resource; not tied to any specific consumer.

## Services

| Service | Role | Port | Hardware | Lifecycle owner |
|---|---|---|---|---|
| `piper-service` | TTS — text → WAV / Ogg-Opus | 9001 | CPU only | `piper-service/docker-compose.yml` |
| `whisper-service` | STT — audio → text / json / srt / vtt | 9000 | NVIDIA GPU, CPU fallback | `whisper-service/docker-compose.yml` |

**Each service is launched from its own directory.** There is no root compose. Running both: `cd piper-service && docker compose up -d` and same in `whisper-service/`.

## Shared host paths

| Host path | Purpose |
|---|---|
| `./logs/` | Bind-mounted into both containers as `/app/logs`. Both services write rotated logs here (10MB × 5). |
| `~/services/data/piper/models` | Bind-mounted piper voice models (`.onnx` + `.json`). |
| `~/services/data/whisper/models` | Bind-mounted HF cache for whisper models. |

## Common operations

```bash
# Restart a service
cd piper-service && docker compose restart

# Rebuild after source changes
cd piper-service && docker compose up -d --build

# Tail logs (host-side, both services)
tail -f logs/piper-service.log logs/whisper-service.log

# Check health
curl -s localhost:9001/health   # piper
curl -s localhost:9000/health   # whisper
```

## Configuration

All config flows through each service's `.env` (template at `.env.example` at repo root). Per-service composes consume those vars — there is no separate per-service `.env`. See each service's `CLAUDE.md` for the keys each one reads.

## Repo conventions

- Per-service compose files own their containers. Don't add a root `docker-compose.yml` — the previous one was deleted as stale legacy.
- `.nelson/` is a local Nelson skill workspace (gitignored).
- Service code changes only ship to the running container after a `docker compose up -d --build` from that service's directory.

## Open work

- `ROADMAP.md` at repo root tracks deferred behavior-changing fixes (see §4 SKIP list). Not blocking; pick up when convenient.

---
> Source: [Krenuds/earpiece](https://github.com/Krenuds/earpiece) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
