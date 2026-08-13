## contabo-storage-manager

> <!-- From: /root/contabo_storage_manager/AGENTS.md -->

<!-- From: /root/contabo_storage_manager/AGENTS.md -->
<!-- From: /root/contabo_storage_manager/AGENTS.md -->
# AGENTS.md – Contabo Storage Manager

> This file contains essential context for AI coding agents working on this project.  
> The project is a multi-service storage bridge and static file server for a Contabo VPS.

---

## Project Overview

This project provides webhook receivers, REST APIs, and static file serving for multiple frontend applications. It persists payloads and files locally under a single directory, syncs to external FTP/SFTP (including automatic FLAC mirroring), and serves content back over HTTPS via Nginx.

### Architecture

```
Internet ──→ (nginx / Caddy / direct)
               │
               ├─── :8000  Python Bridge (FastAPI) ── primary service
               │             ├── Webhooks  → /webhook/*
               │             ├── Admin     → /admin (upload dashboard)
               │             ├── Shaders   → /api/shaders*, /api/maps
               │             ├── Music     → /api/songs*, /api/music/*
               │             ├── Sequencer → /api/songs*, /api/patterns*, /api/banks*, /api/samples*, /api/items
               │             ├── Notes     → /api/notes/*
               │             ├── Pachinball → /maps*, /music*, /backbox, /zones*, /upload/*
               │             ├── Leaderboard → /api/leaderboard*
               │             ├── Adventure  → /api/adventure/*
               │             ├── VPS Browser → /api/vps/*
               │             ├── Models    → /models/* (Range/HEAD support for WebLLM)
               │             ├── Mods      → /api/mods/* (MOD music files)
               │             ├── Presets   → /api/presets/* (MilkDrop .milk files)
               │             ├── Textures  → /api/textures/* (image textures)
               │             ├── Static    → /files/{path}
               │             └── Remote    → /api/admin/run, /api/admin/logs/{task_id}
               │
               ├─── :3000  Node Bridge (Express) ── minimal webhook receiver
               │             ├── POST /webhook/generic
               │             ├── POST /webhook/shopify
               │             └── POST /webhook/github
               │
               └─── :8080  Nginx static server (nginx-files container)
                             └── GET /<any-path>  → serves FILES_DIR directly

All services write to /home/ftpbridge/files  ←── single FTP account, vsftpd served
```

### Admin Dashboard

The Python bridge serves a universal upload dashboard at `GET /admin`. It supports drag-and-drop uploads for:
- Audio (`.mp3`, `.flac`, `.wav`, `.ogg`, `.m4a`, `.aac`) → `/api/songs/upload`
- Notes (`.md`) → `/api/notes/write/{name}`
- Shaders (`.wgsl`, `.json`) → `/api/shaders`

The admin panel also supports remote SSH command execution via `POST /api/admin/run` (whitelisted commands only) and SSE log streaming via `GET /api/admin/logs/{task_id}`.

### Officially Supported Apps

The following applications have **first-class support** with dedicated endpoints and organized storage:

1. **image_video_effects** — `/webhook/image-effects` — shader effects and outputs
2. **flac_player** — `/api/songs/*`, `/api/music/*` — audio library streaming
3. **web_sequencer** — `/webhook/sequencer`, `/api/songs/*`, `/api/patterns/*`, etc. — music composition
4. **rain_edit** — `/api/notes/*` — markdown note-taking (REST API only)
5. **cloud_notes** — `/api/notes/*`, `/webhook/notes` — cloud-based note sync with webhook support

All apps share the same single FTP account and storage directory (`/home/ftpbridge/files`). The **Notes** API is shared between `rain_edit` and `cloud_notes`, allowing both apps to read/write notes interchangeably.

---

## Technology Stack

### Python Bridge
- **Runtime**: Python 3.12+
- **Framework**: FastAPI 0.111+
- **Server**: Uvicorn with standard workers (default 2)
- **Key Dependencies**:
  - `fastapi` / `uvicorn[standard]` — web framework and ASGI server
  - `pydantic` / `pydantic-settings` — configuration and request/response models
  - `aiofiles` — async file I/O
  - `httpx` — async HTTP client
  - `python-multipart` — multipart form parsing
  - `paramiko` — SFTP connections
  - `asyncssh` — remote SSH command execution for admin panel
  - `jinja2` — admin panel HTML templating
  - `google-cloud-storage` — GCS bucket sync for music
  - `watchdog` — filesystem watcher for auto-indexing audio files
  - `pydub` — audio metadata extraction
  - `aiocache` — caching layer
  - `gunicorn` — production WSGI/ASGI worker alternative

### Node Bridge
- **Runtime**: Node.js 18+ (Node 20 recommended)
- **Framework**: Express.js 4.19+
- **Key Dependencies**:
  - `basic-ftp` — FTP operations
  - `winston` — structured logging
  - `express-rate-limit` — rate limiting on webhook endpoints
  - `dotenv` — environment configuration

### Infrastructure
- **Containerization**: Docker + Docker Compose (profiles: `full`, `python`, `node`, `storage`)
- **Static File Server**: Nginx (port 8080)
- **Deployment Target**: Contabo Ubuntu VPS with vsftpd
- **CI/CD**: GitHub Actions workflow (`.github/workflows/deploy.yml`) deploys on every push to `main`

---

## Project Structure

```
contabo_storage_manager/
├── packages/
│   ├── python-bridge/          # FastAPI service (port 8000)
│   │   ├── app/
│   │   │   ├── __init__.py
│   │   │   ├── main.py         # FastAPI app entry + CORS + admin routes
│   │   │   ├── webhooks.py     # Webhook route handlers + static /files
│   │   │   ├── api.py          # Shaders, maps, images, flac_player song API
│   │   │   ├── api_full.py     # Extended API router variant (unused in main.py)
│   │   │   ├── api_shim.py     # Compatibility shim router (unused)
│   │   │   ├── api_simple.py   # Minimal API router variant (unused)
│   │   │   ├── audio_router.py # Pachinball music & samples API
│   │   │   ├── sequencer_router.py  # web_sequencer cloud storage API
│   │   │   ├── notes_router.py      # Plain-text markdown notes API
│   │   │   ├── pachinball_router.py # Pachinball maps/music/backbox/zones
│   │   │   ├── leaderboard_router.py # High scores
│   │   │   ├── adventure_router.py   # Adventure mode progress & levels
│   │   │   ├── vps_browser_router.py # VPS file browser (browse/upload/delete)
│   │   │   ├── models_router.py      # Model serving with Range/HEAD support
│   │   │   ├── mod_router.py         # MOD music file indexing & metadata
│   │   │   ├── presets_router.py     # MilkDrop preset upload/list/download
│   │   │   ├── textures_router.py    # Image texture upload/list/download
│   │   │   ├── presets.py            # Preset index loader (startup)
│   │   │   ├── cors.py               # CORS middleware options builder
│   │   │   ├── flac_client.py        # External FLAC Player registration client
│   │   │   ├── file_watcher.py       # Background watchdog auto-indexer
│   │   │   ├── ftp_client.py         # FTPS + SFTP upload client
│   │   │   ├── remote_sync.py        # Background remote-to-local sync loop
│   │   │   ├── sync.py               # Legacy background external API polling loop
│   │   │   ├── config.py             # pydantic-settings configuration
│   │   │   ├── models.py             # Shared Pydantic models
│   │   │   ├── logger.py             # Structured logger setup
│   │   │   └── templates/
│   │   │       └── admin.html        # Universal upload dashboard
│   │   ├── main_gcs.py         # Standalone GCS sync entry point
│   │   └── requirements.txt
│   ├── node-bridge/            # Express service (port 3000)
│   │   ├── src/
│   │   │   ├── index.js        # Express app entry point
│   │   │   ├── webhooks.js     # Webhook handlers + signature verification
│   │   │   └── logger.js       # Winston logger
│   │   ├── package.json
│   │   └── package-lock.json
│   └── shared/                 # Common utilities
│       ├── ftp/
│       │   ├── ftp_utils.py    # Python FTP helpers
│       │   └── ftpUtils.js     # Node.js FTP helpers
│       ├── logger/
│       │   └── logger.py       # Shared Python logger
│       └── config/
│           └── config.py       # Standalone script config loader
├── scripts/                    # Standalone utilities
│   ├── poll_api.py             # API polling script
│   ├── ftp_sync.py             # Sync local dir → FTP
│   ├── sync_gcs_music.py       # Sync Google Cloud Storage music → local
│   ├── sync_music_index.py     # Music index sync helper
│   ├── sync_mods.py            # MOD file sync helper
│   ├── sync_presets.py         # Preset sync helper
│   ├── import_shaders.py       # Shader import utility
│   ├── import_shaders_with_params.py
│   ├── upload_model_to_vps.py  # Model upload helper
│   ├── index_mods.py           # MOD index builder
│   ├── download_shaders.sh     # Shader download script
│   └── listFtpFiles.js         # List FTP contents
├── tests/                      # pytest test suite
│   ├── test_api_rating_filters.py
│   ├── test_cors.py
│   ├── test_flac_client.py
│   └── test_presets_router.py
├── config/
│   ├── nginx-files.conf        # Nginx static server config
│   ├── nginx.conf.example      # Example reverse proxy config
│   ├── nginx-models.conf       # Nginx config for model serving
│   ├── storage.noahcohn.com.conf # Domain-specific nginx config
│   └── vsftpd.conf.example     # Example vsftpd config
├── systemd/                    # Systemd service files
│   ├── contabo-storage-node.service
│   ├── contabo-storage-python.service
│   ├── contabo-storage-sync.service
│   ├── contabo-storage-sync.timer
│   ├── ftpbridge-node.service
│   └── ftpbridge-python.service
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions CI/CD deploy workflow
├── docker-compose.yml          # Docker Compose orchestration
├── Dockerfile.python           # Python bridge container
├── Dockerfile.node             # Node bridge container
├── pyproject.toml              # Python project metadata & dev deps
├── package.json                # Node.js workspace scripts
├── .env.example                # Environment variable template
├── setup-services.sh           # Systemd service installer
├── check-deployment.sh         # Deployment verification script
├── restart-vps.sh              # VPS restart helper
├── vps-git-fix.sh              # Git fix helper
├── README.md
├── AGENTS.md
└── SEQUENCER_MIGRATION.md
```

---

## Supported Endpoints

### Python Bridge (port 8000)

#### Root & Health
| Method | Path | Description |
|--------|------|-------------|
| `GET`  | `/` | Media gallery placeholder page |
| `GET`  | `/health` | Health check (includes storage writable test) |

#### Webhooks
| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/webhook/generic` | Generic JSON webhooks |
| `POST` | `/webhook/github` | GitHub webhook events |
| `POST` | `/webhook/image-effects` | image_video_effects app |
| `POST` | `/webhook/flac` | flac_player multipart upload |
| `POST` | `/webhook/sequencer` | web_sequencer multipart upload |
| `POST` | `/webhook/notes` | cloud_notes structured note data |

#### Admin
| Method | Path | Description |
|--------|------|-------------|
| `GET`  | `/admin` | Universal upload dashboard |
| `POST` | `/api/admin/sync-music` | Trigger GCS music sync |
| `POST` | `/api/admin/run` | Run remote SSH command (whitelisted) |
| `GET`  | `/api/admin/logs/{task_id}` | Stream remote command logs (SSE) |

#### Shaders & Maps
| `GET`  | `/api/shaders` | List shaders with pagination/filtering |
| `POST` | `/api/shaders` | Create shader |
| `GET`  | `/api/shaders/{id}` | Get shader metadata |
| `PUT`  | `/api/shaders/{id}` | Update shader metadata |
| `POST` | `/api/shaders/{id}/rate` | Rate a shader |
| `GET`  | `/api/shaders/{id}/code` | Get WGSL source |
| `GET`  | `/api/maps` | List LCD table maps |

#### Music Library (flac_player)
| `GET`  | `/api/songs` | List songs |
| `GET`  | `/api/songs/stats` | Library statistics |
| `GET`  | `/api/songs/{id}` | Get song metadata |
| `POST` | `/api/songs/{id}/play` | Record play event |
| `PATCH`| `/api/songs/{id}` | Update song metadata |
| `POST` | `/api/songs/{id}/trash` | Mark as trashed |
| `GET`  | `/api/music/{song_id}` | Stream audio file |
| `POST` | `/api/songs/upload` | Upload audio file |

#### Sequencer (web_sequencer)
| `GET`  | `/api/songs` | List sequencer songs |
| `POST` | `/api/songs` | Upload song JSON |
| `GET`  | `/api/songs/{id}` | Get song data |
| `PATCH`| `/api/songs/{id}` | Update song |
| `DELETE`| `/api/songs/{id}` | Delete song |
| `GET`  | `/api/patterns` | List patterns |
| `POST` | `/api/patterns` | Upload pattern |
| `GET`  | `/api/banks` | List banks |
| `GET`  | `/api/samples` | List samples |
| `POST` | `/api/samples` | Upload audio sample |
| `GET`  | `/api/items` | List all items (HuggingFace compat) |

#### Notes (rain_edit / cloud_notes)
| `GET`  | `/api/notes/list` | List all notes |
| `GET`  | `/api/notes/read/{name}` | Read a note |
| `POST` | `/api/notes/write/{name}` | Write a note |
| `POST` | `/api/notes/save` | Save note with title |
| `POST` | `/api/notes/sync` | Sync from cloud_notes payload (direct browser sync) |
| `POST` | `/api/notes/sync/batch` | Batch sync multiple notes |
| `DELETE`| `/api/notes/delete/{name}` | Delete a note |

**Webhook for cloud_notes:**
| `POST` | `/webhook/notes` | cloud_notes webhook-based sync (no HMAC required) |

#### Pachinball
| `GET`  | `/maps` | List maps |
| `GET`  | `/maps/{id}` | Get map config |
| `POST` | `/maps` | Create map |
| `PUT`  | `/maps/{id}` | Update map |
| `DELETE`| `/maps/{id}` | Delete map |
| `GET`  | `/music` | List music tracks |
| `GET`  | `/music/{id}` | Get track |
| `POST` | `/music` | Create track entry |
| `POST` | `/upload/music` | Upload music file |
| `POST` | `/upload/backbox` | Upload backbox media |
| `POST` | `/upload/zone` | Upload zone video |
| `GET`  | `/backbox` | Backbox manifest |
| `GET`  | `/zones` | Zone manifest |
| `GET`  | `/files/{path}` | Serve pachinball static files |

#### Leaderboard & Adventure
| `GET`  | `/api/leaderboard` | Get high scores |
| `POST` | `/api/leaderboard` | Submit score |
| `GET`  | `/api/adventure/progress` | Get progress |
| `POST` | `/api/adventure/progress` | Save progress |
| `GET`  | `/api/adventure/levels` | List levels |
| `POST` | `/api/adventure/complete-level/{id}` | Complete level |

#### VPS Browser
| `GET`  | `/api/vps/browse?path=` | List directory |
| `GET`  | `/api/vps/file?path=` | Download file |
| `POST` | `/api/vps/upload` | Upload file |
| `PUT`  | `/api/vps/file` | Overwrite file |
| `DELETE`| `/api/vps/file?path=` | Delete file |

#### Models (WebLLM / TTS)
| `GET`  | `/models/health` | Model serving health |
| `GET`  | `/models/list` | List available models |
| `GET`  | `/models/{model_id}/{file_path}` | Serve model file with Range support |
| `HEAD` | `/models/{model_id}/{file_path}` | Model file headers |
| `GET`  | `/models/tts/list` | List TTS models |
| `GET`  | `/models/tts/health` | TTS model health |

#### MOD Music
| `GET`  | `/api/mods` | List MOD files with metadata (search/tag filter) |
| `GET`  | `/api/mods/scan` | Sync from remote FTP + scan directory + rebuild index |
| `POST` | `/api/mods/reindex` | Re-extract metadata for all indexed mods |
| `GET`  | `/api/mods/{mod_id}` | Get MOD metadata |
| `PATCH`| `/api/mods/{mod_id}` | Update MOD metadata |
| `GET`  | `/api/mods/{mod_id}/download` | Download MOD file (CORS-safe proxy) |

#### MilkDrop Presets
| `GET`  | `/api/presets/` | List preset directories with counts |
| `GET`  | `/api/presets/{dir_name}` | List `.milk` files in a directory |
| `GET`  | `/api/presets/{dir_name}/{filename}` | Get raw preset content |
| `POST` | `/api/presets/{dir_name}` | Upload/overwrite a `.milk` file |
| `DELETE`| `/api/presets/{dir_name}/{filename}` | Delete a `.milk` file |
| `GET`  | `/api/presets/random` | Return a random preset (local URL if synced, else glsl fallback) |
| `GET`  | `/api/presets/stats` | Preset index statistics |
| `POST` | `/api/presets/rescan` | Rebuild cached preset indexes from glsl.1ink.us |

#### Textures
| `GET`  | `/api/textures/` | List texture directories with counts |
| `GET`  | `/api/textures/{dir_name}` | List image files in a directory |
| `GET`  | `/api/textures/{dir_name}/{filename}` | Serve texture image file |
| `POST` | `/api/textures/{dir_name}` | Upload a texture file |
| `DELETE`| `/api/textures/{dir_name}/{filename}` | Delete a texture file |

#### Redirects (Backwards Compatibility)
| `GET` | `/music` | 307 redirect to `/api/music` |
| `GET` | `/leaderboard` | 307 redirect to `/api/leaderboard` |

#### Static Files
| `GET` | `/files/{path:path}` | Serve stored files with correct MIME types |

### Node Bridge (port 3000)
| Method | Path | Description |
|--------|------|-------------|
| `GET`  | `/health` | Health check |
| `POST` | `/webhook/generic` | Generic JSON webhooks |
| `POST` | `/webhook/shopify` | Shopify webhook events |
| `POST` | `/webhook/github` | GitHub webhook events |

---

## Build and Run Commands

### Docker (Recommended)

```bash
# Start both bridges + nginx
docker compose --profile full up -d

# Start Python bridge only
docker compose --profile python up -d

# Start Node bridge only
docker compose --profile node up -d

# Start nginx static server only
docker compose --profile storage up -d

# View logs
docker compose logs -f python-bridge
docker compose logs -f node-bridge

# Rebuild after code changes
docker compose --profile full up -d --build

# Stop everything
docker compose --profile full down
```

### Without Docker (systemd)

**Python bridge:**
```bash
# Setup virtual environment
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r packages/python-bridge/requirements.txt

# Install and start service
sudo cp systemd/contabo-storage-python.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now contabo-storage-python
```

**Node bridge:**
```bash
# Install dependencies
cd packages/node-bridge && npm ci --omit=dev

# Install and start service
sudo cp systemd/contabo-storage-node.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now contabo-storage-node
```

### Development

```bash
# Node.js development with auto-reload
npm run dev:node

# Python development
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

---

## Environment Configuration

Copy `.env.example` to `.env` and configure:

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_ENV` | `production` | `development` or `production` |
| `FTP_HOST` | `127.0.0.1` | Local vsftpd host |
| `FTP_PORT` | `21` | FTP port (21 for FTPS, 22 for SFTP) |
| `FTP_USER` | `ftpbridge` | FTP username |
| `FTP_PASS` | *(empty)* | FTP password |
| `FTP_UPLOAD_DIR` | `/home/ftpbridge/files` | Remote upload directory |
| `FTP_PASSIVE_MIN` | `40000` | Passive mode port range lower bound |
| `FTP_PASSIVE_MAX` | `40100` | Passive mode port range upper bound |
| `FTP_TLS` | `false` | Enable FTPS |
| `EXTERNAL_FTP_HOST` | *(empty)* | External SFTP host |
| `EXTERNAL_FTP_USER` | *(empty)* | External SFTP user |
| `EXTERNAL_FTP_PASS` | *(empty)* | External SFTP password |
| `EXTERNAL_FTP_PORT` | `22` | External SFTP port |
| `EXTERNAL_FTP_DIR` | `/` | External SFTP base directory |
| `WEBHOOK_SECRET` | *(empty)* | HMAC secret for signature verification |
| `WEBHOOK_HMAC_ALGO` | `sha256` | HMAC algorithm (`sha256` or `sha1`) |
| `PYTHON_HOST` | `0.0.0.0` | Python bridge bind host |
| `PYTHON_PORT` | `8000` | Python bridge port |
| `PYTHON_WORKERS` | `2` | Uvicorn worker count |
| `NODE_HOST` | `0.0.0.0` | Node bridge bind host |
| `NODE_PORT` | `3000` | Node bridge port |
| `CORS_ORIGINS` | `*` | Comma-separated CORS origins |
| `CORS_ORIGIN_REGEX` | *(built-in default)* | Regex fallback for trusted browser origins |
| `FILES_DIR` | `/home/ftpbridge/files` | Local storage directory |
| `STATIC_BASE_URL` | `https://storage.1ink.us` | Public HTTPS URL for file links |
| `MAX_UPLOAD_MB` | `8192` | Maximum upload size in MB |
| `NGINX_PORT` | `8080` | Nginx static server host port |
| `LOG_LEVEL` | `info` | Logging level |
| `LOG_FILE` | `/var/log/ftpbridge/app.log` | Log file path |
| `EXTERNAL_API_URL` | *(empty)* | URL for polling sync |
| `EXTERNAL_API_KEY` | *(empty)* | Bearer token for external API |
| `POLL_INTERVAL_SECONDS` | `60` | Polling interval |
| `FLAC_PLAYER_API_URL` | *(empty)* | External FLAC Player backend URL for auto-registration |
| `EXTERNAL_FLAC_DIR` | `flac_songs` | Remote folder for FLAC song mirroring |

---

## Code Style Guidelines

### Python
- **Formatter**: Ruff (configured in `pyproject.toml`)
- **Line length**: 120 characters
- **Target version**: Python 3.12
- **Lint rules**: E, F, I, UP (ignore E501)

```bash
# Run linter
ruff check packages/python-bridge

# Run formatter
ruff format packages/python-bridge
```

### JavaScript/Node.js
- Use `"use strict";` at the top of files
- Prefer `const` and `let` over `var`
- Use async/await for asynchronous operations
- JSDoc comments for function documentation

---

## Testing Instructions

### Python Tests

The project uses `pytest` with `asyncio_mode = "auto"`. Tests live in the `tests/` directory at the project root.

**Existing test files:**

| File | What it tests |
|------|---------------|
| `tests/test_api_rating_filters.py` | Query-parameter filtering on `/api/songs` (`rating_gte`, `rating_lt`) |
| `tests/test_cors.py` | CORS preflight behavior: allowed origins pass, unknown origins are rejected |
| `tests/test_flac_client.py` | `flac_client.register_song_with_flac_player()` — forwards extended metadata when `FLAC_PLAYER_API_URL` is set, returns `None` when unset |
| `tests/test_presets_router.py` | Full CRUD on `/api/presets` — list dirs, list files, upload `.milk`, download, delete, path-traversal rejection |
| `tests/test_presets_random.py` | `/api/presets/random` — returns local URL when file exists, falls back to glsl URL when missing, 503 when empty |

```bash
# Install dev dependencies
pip install -e ".[dev]"

# Run all tests
pytest

# Run with coverage
pytest --cov=packages.python-bridge
```

### Manual Testing

```bash
# Health check
curl http://localhost:8000/health
curl http://localhost:3000/health

# Generic webhook
curl -X POST http://localhost:8000/webhook/generic \
  -H "Content-Type: application/json" \
  -H "X-Hub-Signature-256: sha256=<hmac>" \
  -d '{"source":"test","event":"ping","data":{}}'

# File upload (Python bridge)
curl -X POST http://localhost:8000/webhook/flac \
  -H "X-Hub-Signature-256: sha256=<hmac>" \
  -F "action=upload_audio" \
  -F "file=@audio.flac"
```

---

## Deployment

### GitHub Actions CI/CD

The repository includes `.github/workflows/deploy.yml`. On every push to `main`:

1. The workflow SSHs into the VPS using secrets (`VPS_HOST`, `VPS_USER`, `VPS_PASSWORD`).
2. Runs `git pull origin main` in `/root/contabo_storage_manager`.
3. Prefers Docker if available:
   ```bash
   docker compose --profile full up -d --build
   ```
4. Falls back to systemd if Docker is unavailable:
   ```bash
   bash setup-services.sh
   systemctl restart contabo-storage-python
   systemctl restart contabo-storage-node
   systemctl enable contabo-storage-sync.timer
   systemctl start contabo-storage-sync.timer
   ```
5. Waits for the health endpoint (`/health` on port 8000) to return OK.
6. Triggers a background music sync via `POST /api/admin/sync-music`.

### Manual VPS Deployment

```bash
# Docker path
cd /root/contabo_storage_manager
git pull origin main
docker compose --profile full up -d --build

# Systemd path (fallback)
cd /root/contabo_storage_manager
git pull origin main
bash setup-services.sh
systemctl restart contabo-storage-python
systemctl restart contabo-storage-node
```

---

## Security Considerations

### Webhook Signature Verification
- All JSON webhook endpoints support HMAC signature verification
- Signatures expected in `X-Hub-Signature-256` header (or `X-Shopify-Hmac-Sha256` for Shopify)
- Format: `sha256=<hex_digest>`
- Verification is **disabled** if `WEBHOOK_SECRET` is not set
- Uses `hmac.compare_digest()` / `crypto.timingSafeEqual()` to prevent timing attacks
- Multipart endpoints (`/webhook/flac`, `/webhook/sequencer`) check signature header presence when a secret is configured
- `/webhook/notes` is intentionally open for direct browser-to-server sync

### File Upload Security
- Filename sanitization: only alphanumeric, `._-` allowed
- Path traversal prevention in static file serving (`resolve()` + prefix check)
- Max upload size configurable via `MAX_UPLOAD_MB` (default 8192 MB)
- Notes API validates names with regex `^[a-zA-Z0-9_\-\.]+$` and rejects traversal sequences
- Preset API rejects filenames with `..`, `/`, `\`, or non-`.milk` extensions
- Texture API rejects filenames with `..`, `/`, `\`, or unsupported extensions

### Network Security
- FTP/SFTP connections use TLS when `FTP_TLS=true`
- Port 22 automatically uses SFTP (paramiko) instead of FTPS
- Rate limiting on Node bridge webhook endpoints (100 req/min per IP)
- Enhanced CORS middleware plus explicit global CORS header fallback middleware
- CORS `allow_credentials` is disabled when `*` is in `CORS_ORIGINS`

### Secrets Management
- Never commit `.env` file (listed in `.gitignore`)
- Use `.env.example` as template without real credentials
- SSH keys for remote admin commands mounted from host (`/root/.ssh`)

---

## Storage Layout

Files are organized under `FILES_DIR` (default `/home/ftpbridge/files`):

```
/home/ftpbridge/files/
├── webhooks/                   # Generic webhook payloads
│   ├── generic/
│   ├── github/
│   └── shopify/
├── image-effects/
│   ├── shaders/               # Shader JSON configs
│   ├── metadata/              # Effect metadata
│   └── outputs/               # Generated outputs (dated)
├── audio/
│   ├── music/                 # Canonical music library (all formats)
│   ├── flac/                  # Legacy FLAC audio files
│   ├── wav/                   # WAV/AIFF files
│   ├── covers/                # Cover art
│   ├── playlists/             # Playlist JSON
│   ├── samples/               # Sound samples
│   └── metadata/              # Track metadata
├── sequencer/
│   ├── songs/                 # Song JSON files
│   ├── patterns/              # Pattern JSON files
│   ├── banks/                 # Bank JSON files
│   └── samples/               # Audio samples
├── shaders/                   # Shader directories (meta.json + .wgsl)
├── notes/                     # Plain-text markdown notes
│   ├── webhook/               # Archived note JSON payloads
│   └── markdown/              # Markdown exports
├── pachinball/                # Pachinball game content
│   ├── maps/
│   │   └── maps.json
│   ├── music/
│   │   └── tracks.json
│   ├── backbox/
│   ├── zones/
│   └── adventure/
├── leaderboard/
│   └── index.json
├── mods/                      # MOD music files + index.json
├── models/                    # ML models for WebLLM / TTS
│   └── tts/
├── textures/                  # Image textures (PNG, JPG, WEBP, etc.)
├── weeks_textures/            # Weekly texture pool
├── custom_textures/           # User-uploaded custom textures
├── milk/                      # MilkDrop presets (default pool)
├── milkSML/                   # MilkDrop presets (small pool)
├── milkMED/                   # MilkDrop presets (medium pool)
├── milkLRG/                   # MilkDrop presets (large pool)
├── custom_milk/               # User-uploaded custom presets
├── images.json                # Recorded images index
└── songs.json                 # flac_player music library index
```

---

## Key Implementation Details

### Python Bridge
- Uses `pydantic-settings` for environment-based configuration
- FTP client (`ftp_client.py`) supports both FTPS (port 21) and SFTP (port 22) via paramiko. The `StorageFTPClient` class auto-selects the protocol based on port number.
- Static files served with correct MIME types for audio/video/model files
- Background file watcher (`watchdog`) auto-indexes new audio files into `songs.json`
- Background remote sync (`remote_sync.py`) polls external SFTP directories every 5 minutes and downloads new/changed files for audio, textures, and presets
- Admin panel supports remote SSH command execution via `asyncssh` with a strict whitelist (`git-pull`, `npm-install`, `npm-build`, `restart-service`, `sync-indexes`, `sync-music`)
- Model router implements full HTTP Range request support for WebLLM chunked downloads
- `flac_client.py` optionally registers uploaded audio with an external FLAC Player backend (both async and sync variants)
- Uploaded FLAC files are mirrored to the external SFTP host under `EXTERNAL_FLAC_DIR` (default `flac_songs`) as a best-effort operation
- `mod_router.py` uses `openmpt123 --info` to extract title, tracker (author), and duration from MOD files
- `presets_router.py` whitelists 5 preset directories and enforces `.milk` extension; paths are resolved via a static mapping dict to prevent traversal
- Presets must be synced locally (via `scripts/sync_presets.py`) before the VPS can serve them. `/api/presets/random` returns a local URL when the file exists on disk, falling back to the external `glsl.1ink.us` URL only when the local copy is missing
- `textures_router.py` whitelists 3 texture directories and enforces image extensions (`.png`, `.jpg`, `.jpeg`, `.webp`, `.bmp`, `.tga`)
- `notes_router.py` provides shared `/api/notes/*` REST API for both rain_edit and cloud_notes with slugified filenames, path traversal protection, and support for syncing via payloads
- `webhooks.py` includes `POST /webhook/notes` endpoint for cloud_notes webhook-based sync; does NOT require HMAC signatures (safe for browser-to-server), archives payloads in `notes/webhook/`, and supports encrypted content markers (`ENC:v1:`)
- `cors.py` builds `CORSMiddleware` options dynamically: credentials are disabled when `*` is present in origins
- `main.py` registers 13 routers in a specific order and installs both `CORSMiddleware` and a fallback `add_cors_headers` middleware

### Node Bridge
- Raw body capture middleware for HMAC verification
- Winston logger with both console and file outputs
- basic-ftp library for FTP operations

### Both Bridges
- Files saved with timestamp prefix: `YYYYMMDDTHHMMSSffffff_<name>`
- External FTP upload is optional; files always saved locally first
- Graceful handling of missing FTP credentials

---
> Source: [ford442/contabo_storage_manager](https://github.com/ford442/contabo_storage_manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
