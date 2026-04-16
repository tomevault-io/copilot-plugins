## clawpilot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ClawPilot is a local-first web dashboard (control plane) for supervising OpenClaw agents. It provides a drag-and-drop task board, contextual chat, realtime progress feed, and human-in-the-loop handoffs. Single-user, LAN-only for MVP.

## Tech Stack

- **Framework**: Next.js (App Router) — monolith (frontend + API route handlers)
- **UI**: React, dnd-kit (drag/drop)
- **Database**: SQLite via Prisma ORM
- **Realtime**: Server-Sent Events (SSE)
- **Platform**: macOS-first

## Architecture

Single Next.js app serving both UI and API. No separate backend for MVP.

**Core data flow**: User moves task to "Cliff" → Backend creates Job → Runner executes and emits Events → UI receives via SSE → Timeline renders

**Four core entities** (see `docs/data-model.md`):
- **Task**: unit of work (statuses: draft → ready → cliff → waiting → done)
- **Job**: execution instance of a task (statuses: queued → running → blocked → succeeded/failed/cancelled)
- **Event**: immutable records from jobs (types: state_change, log, tool, artifact, question)
- **Artifact**: output references (kinds: url, file, diff, note)

## Development Commands

Project is in scaffolding phase. Once implementation begins:

```bash
# Expected setup (not yet created)
pnpm install               # install deps
pnpm exec prisma generate  # generate Prisma client
pnpm exec prisma db push   # sync schema to SQLite
pnpm dev                   # start Next.js dev server
```

## MVP Milestones

- **M1**: App skeleton — Next.js + board UI + SQLite/Prisma schema
- **M2**: Realtime event pipeline — SSE endpoint + timeline UI
- **M3**: Job runner integration — start/pause/cancel + logging
- **M4**: Chat — global + per-task threads + "needs human input" state

## Key Design Decisions

- Monolithic Next.js app (split backend later only if needed)
- SSE over WebSockets for simplicity (upgrade path exists)
- SQLite for zero-config local persistence
- Events are immutable and durable — refresh never loses state
- OpenClaw integration via Gateway API (spawn sessions, send messages, read events)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/toml0006) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
