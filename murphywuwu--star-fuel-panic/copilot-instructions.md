## star-fuel-panic

> This file provides instructions for Claude Code when working in this repository.

# CLAUDE.md

This file provides instructions for Claude Code when working in this repository.

## Project Overview

**Fuel Frog Panic** - A competitive on-chain gaming platform built on EVE Frontier (Sui blockchain).

- **Frontend**: Next.js 14 (App Router) + React 18 + Zustand + TailwindCSS
- **Backend**: Next.js API Routes + Node.js Workers + Supabase (PostgreSQL + Realtime)
- **Blockchain**: Sui Move (world::fuel contracts)
- **Deployment**: Vercel + Railway/Fly.io + Supabase Cloud

## Project Documentation

Always check these docs before implementation:

| Doc | Path | Purpose |
|-----|------|---------|
| PRD | `docs/PRD.md` | Product requirements and business logic |
| Architecture | `docs/architecture.md` | System architecture and technical design |
| SPEC | `docs/SPEC.md` | Interface contracts and implementation specs |
| TODO | `docs/TODO.md` | Active implementation task list |
| TEST-TODO | `docs/TEST-TODO.md` | Testing checklist by layer |
| Contracts | `docs/all_contracts.move.txt` | EVE Frontier contract reference |
| EVE Bootcamp | `docs/eve_bootcamp.md` | EVE Frontier integration guide and wallet setup |
| Move on Sui | `docs/move-on-sui.md` | Sui Move language reference |
| Devnet Testing | `docs/sui-devnet-testing-standard.md` | Sui devnet testing procedures |

## Agent Roles

This project uses four specialized agent roles:

### PM Agent (`/pm`)
- Product design and PRD creation/maintenance
- Use `/pm` command to enter PM mode

### Architecture Agent
- System architecture and SPEC design
- Produces `docs/architecture.md` and `docs/SPEC.md`

### Todo Agent
- Task decomposition and execution planning
- Maintains `docs/TODO.md`

### Testing Agent
- Test planning and quality validation
- Uses `docs/templates/test-plan-template.md`

## Coding Standards

### Frontend Architecture (Mandatory)
- Use Zustand for state management
- Follow directory structure in `docs/architecture.md`
- Components in `/components`, stores in `/store`, API in `/app/api`

### Before Writing Code
1. Read `docs/PRD.md` for business requirements
2. Read `docs/SPEC.md` for interface contracts
3. Read `docs/architecture.md` for technical constraints
4. Check `docs/TODO.md` for current task status
5. For blockchain/contract work, also read:
   - `docs/all_contracts.move.txt` for EVE Frontier contracts
   - `docs/eve_bootcamp.md` for EVE integration patterns
   - `docs/move-on-sui.md` for Sui Move syntax
   - `docs/sui-devnet-testing-standard.md` for testing procedures

### After Writing Code
1. Update `docs/TODO.md` to mark completed tasks
2. If interface changed, update `docs/SPEC.md`
3. If behavior changed, update `docs/PRD.md`

## Common Commands

```bash
# Development
npm run dev

# Build
npm run build

# Lint
npm run lint
```

## Quick Reference

- Supabase Realtime for WebSocket subscriptions
- Sui RPC for blockchain interactions
- Three Workers: eventListener, nodeScanner, stateMachine
- Match states: lobby -> prestart -> running -> panic -> settling -> settled

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/murphywuwu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
