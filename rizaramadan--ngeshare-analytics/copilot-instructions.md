## ngeshare-analytics

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ngeShare Analytics is a local analytics engine for tracking "Growth" and "Curriculum Progression" on the ngeShare Da'wah platform. It syncs data from a production PostgreSQL database to a local PostgreSQL instance for analysis.

## Commands

```bash
# Start the database
docker compose up -d

# Apply schema migrations (idempotent)
./migrate.sh

# Verify schema integrity
./verify-schema.sh

# Install dependencies (first time)
npm install

# Run data sync from production to local
npm run sync

# Run sync with debug logging
npm run sync:debug

# Connect to database manually
docker compose exec postgres psql -U ngeshare -d ngeshare_analytics

# Reset database completely
docker compose down -v && docker compose up -d && ./migrate.sh
```

## Architecture

### Technology Stack
- **Runtime**: Node.js 24+ (ADR 001)
- **Database**: PostgreSQL 15+ via Docker
- **Web Framework**: NestJS (planned)
- **Frontend**: HTML + Vanilla JS + CSS (planned)
- **Database Driver**: `pg` with `node-pg-migrate` for migrations

### Data Flow
Production PostgreSQL (read-only) → Sync Scripts → Local PostgreSQL (read-write) → Analytics Views → Web Dashboard

### Sync Strategy
- **Incremental sync**: Tables with `updatedAt` columns use timestamp-based incremental sync
- **Window sync**: `UserHangoutGroupAttendance` table uses 45-day rolling window (no `updatedAt`)
- **Sync order (FK dependencies)**: User → Hangout → HangoutEpisode → HangoutGroup → UserHangoutGroup → Attendance

### Database Schema
Core tables: `User`, `Hangout`, `HangoutEpisode`, `HangoutGroup`, `UserHangoutGroup`, `UserHangoutGroupAttendance`
Supporting tables: `CircleProfile`, `Image`, `HangoutProgram`, `Order`, `sync_log`

## Business Domain Rules

### Curriculum Sequence (strictly enforced)
1. Ngeshare Sesi Aqidah (9 episodes)
2. Ngeshare Sesi Hijrah (9 episodes)
3. Ngeshare Sesi Sejarah (11 episodes)
4. Ngeshare Sesi Dakwah (7 episodes)

### Group Status Logic
- **ACTIVE**: Attendance in last 28 days
- **GRADUATED**: Reached max episode for course
- **STALLED**: No attendance >28 days AND not at max episode

### Pipeline Continuity ("Same Group" Linking)
Groups across courses are linked if:
1. Same Facilitator
2. ≥2 overlapping Members
3. Time gap ≤90 days between courses

### Alumni Rule
A Facilitator with `is_alumni = TRUE` has a historical `UserHangoutGroup` record as `MEMBER` with `joinedAt` earlier than their `FACILITATOR` record.

## Database Configuration

Default local connection (configurable via environment variables):
- Host: `localhost` / Port: `5432`
- Database: `ngeshare_analytics`
- User: `ngeshare` / Password: `ngeshare_local_password`

Environment variables: `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`

## Implementation Status

- Phase 1.1: Local DB Setup - Complete
- Phase 1.2: Base Sync Script - Complete
- Phase 1.3: Attendance Sync - Next

See `new-plan.md` for the full 5-phase implementation plan. Architecture decisions are documented in `ADR/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rizaramadan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rizaramadan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
