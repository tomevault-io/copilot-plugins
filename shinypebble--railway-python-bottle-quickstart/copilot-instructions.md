## railway-python-bottle-quickstart

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is a Railway template repository for a minimal Python Bottle web application. It's designed to be forked by Railway users for quick deployment of Python web services.

## Common Commands

### Local Development
```bash
# Setup virtual environment with uv (recommended)
uv venv --python 3.13
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
uv sync

# Run the application locally with development server
python main.py

# Run with Gunicorn (production-like, IPv4 only)
gunicorn main:app --bind "0.0.0.0:8080"

# Run with Gunicorn (dual-stack IPv4/IPv6, same as Railway)
gunicorn main:app --bind 0.0.0.0:${PORT:-8080} --bind "[::]:${PORT:-8080}"

# Run with custom port
PORT=3000 python main.py
```

### Deployment
```bash
# Deploy to Railway
railway up

# Railway automatically provides a PORT variable - no need to set it manually!
# Set log level if needed (defaults to 'info' for production)
railway variables set LOG_LEVEL=debug
```

## Architecture

### Core Components
- **main.py**: Single-file Bottle application with two routes:
  - `/` - Main index route returning HTML
  - `/health` - JSON health check endpoint for Railway monitoring
  
### Configuration
- **pyproject.toml**: Modern Python packaging using setuptools, requires Python 3.13+
- **railway.json**: Railway deployment configuration using Railpack builder with Gunicorn as WSGI server
- Environment variables handled via `os.getenv()` with sensible defaults

### Key Design Decisions
1. Uses Railpack (not Nixpacks) for faster builds with uv package manager
2. Uses Railway's auto-provided PORT variable (best practice) with fallback to 8080 for local development
3. Gunicorn binds to `[::]` for dual-stack IPv4/IPv6 support (required for Railway's public and private networking)
4. Health check endpoint returns JSON for Railway's health monitoring
5. Gunicorn log level controlled via LOG_LEVEL environment variable (debug, info, warning, error, critical)
6. Production uses Gunicorn directly via start command, local development uses Bottle's built-in server

## Testing Changes

Since this is a template repository, ensure any modifications maintain:
1. Zero-configuration deployment capability on Railway
2. Proper handling of Railway's auto-provided PORT variable (do not hardcode ports)
3. The health check endpoint at `/health` returning 200 status
4. Local development fallback when PORT is not provided

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shinypebble) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
