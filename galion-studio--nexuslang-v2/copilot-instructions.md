## nexuslang-v2

> - **Name**: NexusLang v2 + Galion Ecosystem

# Cursor AI - Project Context for NexusLang v2

## Project Identity
- **Name**: NexusLang v2 + Galion Ecosystem
- **Type**: Revolutionary AI-native programming language + Content creation platform
- **Owner**: Maciej Grajczyk (maci.grajczyk@gmail.com)
- **Status**: Production deployment in progress
- **Repository**: galion-studio/nexuslang-v2 (private)

## Current State (November 12, 2025)
- **Deployment**: RunPod IP 213.173.105.83
- **Services**: Building via docker-compose
- **Location**: /workspace/project-nexus
- **Progress**: 95% complete, awaiting DNS configuration

## Key Files to Always Check First
1. PROJECT_STATE_COMPLETE.md - Complete project state
2. v2/.env (on RunPod) - Has all secrets
3. docker-compose.both-platforms.yml - Deployment config
4. PLATFORM_ARCHITECTURE_COMPLETE.md - System design

## Critical Credentials (NEVER EXPOSE)
- OpenRouter: sk-or-v1-ec952b... (in v2/.env on RunPod)
- OpenAI: sk-proj-qxuO6... (in v2/.env on RunPod)
- Database/Redis passwords: In v2/.env

## User Preferences
- **Speed**: Ship fast, iterate quickly
- **Quality**: Production-ready only
- **Principles**: Elon Musk's first principles
- **Communication**: Direct, action-oriented, no fluff
- **Documentation**: Comprehensive but concise

## Architecture
- **Backend**: v2/backend (FastAPI, 54 endpoints)
- **Frontend 1**: v2/frontend (developer.galion.app)
- **Frontend 2**: galion-studio (galion.studio)
- **Database**: PostgreSQL with pgvector
- **Cache**: Redis
- **AI**: OpenRouter (30+ models)

## Active Deployment
- **RunPod**: Services building via d.yml
- **Ports**: Backend 8000, Developer 3000, Studio 3001
- **DNS Needed**: 3 Cloudflare records
- **Status**: Check with `docker ps` on RunPod

## Next Actions
1. Verify services built successfully
2. Configure Cloudflare DNS
3. Test live URLs
4. Launch on ProductHunt
5. Monitor user feedback

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/galion-studio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
