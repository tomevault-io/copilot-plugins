## claude-coach-kit

> This project is AI-agent friendly. Here's how to work with it:

# AGENTS.md — Agent Development Guide

## For AI Coding Agents (Claude Code, Codex, etc.)

This project is AI-agent friendly. Here's how to work with it:

### Project Structure
```
src/pages/       → Dashboard pages
src/components/  → Shared components
src/lib/         → Supabase client, helpers
supabase/        → Edge functions + migrations
scripts/         → Automation cron scripts
```

### Common Tasks
1. **Add a new dashboard page**: Create in `src/pages/`, add route in `App.tsx`
2. **Add an edge function**: Create in `supabase/functions/`, deploy with `npx supabase functions deploy`
3. **Modify email templates**: Edit in `src/pages/Templates.tsx`
4. **Add a new automation**: Create script in `scripts/`, set up cron

### Testing
```bash
npm run dev      # Local development
npm run build    # Verify production build
```

### Environment
All secrets in `.env` — never hardcode. See `.env.example` for required vars.

---
> Source: [krishna-build/claude-coach-kit](https://github.com/krishna-build/claude-coach-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
