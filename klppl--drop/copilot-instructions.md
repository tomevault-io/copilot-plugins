## drop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`drop` is a self-hosted file-sharing service: a single Go binary with no external dependencies, fronted by nginx in Docker. Users upload files via `curl` or browser and get back a short URL.

## Commands

```bash
# Build the binary locally
go build -o drop .

# Run locally (files stored at /data/files/, log at /data/uploads.log)
go run main.go

# Run in purge mode (age-expires files, used by Docker healthcheck)
go run main.go purge

# Docker (production)
cp .env.example .env
# Edit .env — set UPLOAD_TOKEN and ADMIN_PASSWORD
docker compose up -d --build

# Manually trigger a purge inside the running container
docker compose exec drop /app/drop purge
```

There are no tests in this repository.

## Architecture

Everything lives in a single file: `main.go`. The binary runs as an HTTP server (default port 8080) behind nginx.

**Request flow:**
- `POST /` → `handleUpload` — authenticates, validates extension + MIME, re-encodes raster images (strips EXIF), writes to `/data/files/`, logs to `/data/uploads.log`
- `GET /f/<id>.<ext>` → `handleFiles` — serves files; only `jpg/jpeg/png/gif` are served inline, all others force download
- `GET|POST /admin` → `handleAdmin` — session-based admin panel (list/delete files, bulk purge, view log)
- `GET /?sharex` / `GET /?hupl` → `serveShareX` / `serveHupl` — download pre-filled uploader config files

**Background goroutines:**
- `startCleaner` (hourly): age-based purge via `purgeDecay` + disk-pressure eviction (`evictOldest`) if disk > 90%
- Session/login-fail janitor (every 30 min)

**Storage:**
- Files: `/data/files/<randomID>.<ext>` (4–24 char alphanum IDs)
- Log: `/data/uploads.log` (TSV: timestamp, IP, size, original name, stored name)
- Sessions: in-memory `sync.Map` (not persistent across restarts)

**File retention formula:**
```
days = MIN_FILE_AGE + (MAX_FILE_AGE − MIN_FILE_AGE) × (1 − size ÷ MAX_FILESIZE)²
```

**Security properties to preserve:**
- JPEG/PNG/GIF are decoded to pixels and re-encoded to strip EXIF and active content
- Non-inline types (`pdf`, `webp`, `avif`, `heic`, `txt`, `md`, etc.) are forced to download — never rendered inline
- SVG, HTML, XML are intentionally not in the allowlist (XSS risk)
- Upload token auth checked via `Authorization: Bearer` header, `X-Upload-Token` header, or form field
- Admin password compared with `subtle.ConstantTimeCompare`; login rate-limited per IP (10 attempts/hour)
- `X-Real-IP` (set by nginx from `$remote_addr`) is trusted over `X-Forwarded-For` (client-controlled)
- Decompression-bomb guard: image dimensions checked via `image.DecodeConfig` before full decode (50 MP cap)

## Environment variables

| Variable | Default | Notes |
|---|---|---|
| `UPLOAD_TOKEN` | *(empty)* | Required for auth; empty = open uploads |
| `REQUIRE_AUTH` | `true` if token set | Override auth behaviour |
| `MAX_FILESIZE` | `256` | MiB; also update `NGINX_MAX_BODY_MIB` to `MAX_FILESIZE + 2` |
| `NGINX_MAX_BODY_MIB` | `258` | nginx `client_max_body_size`; must stay in sync |
| `MAX_FILE_AGE` | `30` | Days (smallest files) |
| `MIN_FILE_AGE` | `3` | Days (largest files) |
| `ADMIN_PASSWORD` | `changeme` | `/admin` panel password |
| `ADMIN_EMAIL` | `admin@example.com` | Shown on index page |
| `TZ` | `UTC` | Log timestamps |
| `PORT` | `8080` | Listening port |

## nginx / Docker notes

- nginx uses `envsubst` templating — the active config template is `nginx/templates/default.conf.template`; `nginx/default.conf` is a static fallback
- The `drop` container runs as non-root user `drop` (UID 65534), with `read_only: true` filesystem and a `/tmp` tmpfs (Go multipart temp files)
- The Docker healthcheck (`--interval=12h`) runs `/app/drop purge` as a safety net alongside the in-process hourly cleaner

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/klppl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
