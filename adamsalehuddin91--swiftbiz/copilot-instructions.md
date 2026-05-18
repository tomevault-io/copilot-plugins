## swiftbiz

> - Root: `CLAUDE.md`, `ARCHITECTURE.md` (Tokwi system)

# Repository Guidelines

## Project Structure
- Root: `CLAUDE.md`, `ARCHITECTURE.md` (Tokwi system)
- Core memory: `main/` (identity, session, relationship, guild card)
- Runbooks: `enhanced-features/` (8 command runbooks)
- Insights: `insights/` (auto-generated knowledge capture)
- Project specs: `ProjectBrief/` (specs, screenshots, reference data)
- Apps: `SwiftApp Dev/` (Next.js/Vite TypeScript apps, each self-contained)

## Build & Dev Commands
```bash
# HMS Salon (primary)
cd "SwiftApp Dev/hms-salon" && npm run dev
npm run build    # production build
npm run lint     # eslint check

# Stock Monitor
cd "SwiftApp Dev/stock-monitoring-cruip" && npm run dev
```

## Coding Style
- TypeScript/JS: 2-space indent, single quotes, semicolons
- React/TSX: PascalCase components, camelCase variables
- Keep modules small; colocate component + styles under feature folders

## Commit Guidelines
- Conventional Commits: `feat:`, `fix:`, `docs:`, `chore:`
- Scope with project: `feat(hms-salon): add receipt QR code`
- Ensure: build passes, lint clean before push

## Security
- Never commit `.env`, secrets, API keys
- Use `.env.local` per app, provide `.env.example`
- Supabase RLS enabled on all tables

## Agent Notes
- Follow per-app conventions
- Do not rename/move files without explicit request
- Read files before modifying - understand context first

---
> Source: [adamsalehuddin91/SwiftBiz](https://github.com/adamsalehuddin91/SwiftBiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
