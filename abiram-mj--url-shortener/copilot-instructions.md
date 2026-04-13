## url-shortener

> Portfolio + warmup project before main capstone.

# URL Shortener — Project Context

## Project Goal
Portfolio + warmup project before main capstone.
Demonstrates: Spring Boot, Redis caching, PostgreSQL, Docker, REST APIs.

## Architecture Decisions
- Short code generation: Base62 encoding of auto-increment ID
- Cache: Redis for hot URL lookups (TTL 24 hours)
- DB: PostgreSQL for URL storage + analytics
- No auth for now — public API

## API Endpoints
- POST /api/shorten → create short URL
- GET /{shortCode} → redirect to original URL
- GET /api/stats/{shortCode} → get click analytics

## Data Model
URLs table:
- id (auto increment)
- short_code (varchar, unique)
- original_url (text)
- created_at (timestamp)
- expires_at (timestamp, nullable)
- click_count (integer)

## Running locally
- PostgreSQL: localhost:5432
- Redis: localhost:6379
- App: localhost:8080
- Start infra: docker-compose up -d

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Abiram-MJ)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Abiram-MJ)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
