## agentspool

> **ALWAYS read .md files in project root at the start of any task!**

# Cursor AI Rules for AgentsPool Project

## CRITICAL: Read Documentation First
**ALWAYS read .md files in project root at the start of any task!**

### Required Reading:
1. **README.md** - Project overview, architecture, how everything works
2. **DEVELOPMENT_RULES.md** - Local dev setup, DB connection, testing
3. **ADMIN_SECURITY.md** - Admin panel access and security
4. **DOMAIN_SETUP.md** - Domain configuration for deployment

Location: Root directory `/`

Note: If DEVELOPMENT_RULES.md doesn't exist, copy from DEVELOPMENT_RULES.example.md and fill in credentials.

## Project-Specific Rules
- Strictly do only what is described in the prompt. No freedom
- We don't need sh files
- All comments in English
- Always write short, convenient code. We making MVP, not prod.
- Backend uses FastAPI + PostgreSQL
- Frontend uses Next.js + TypeScript + Tailwind CSS
- All local development connects to PRODUCTION database on Railway

## Testing & Commits
**CRITICAL: Always run local tests BEFORE committing!**

```bash
cd backend
python run_tests.py local
# Wait for all tests to pass ✅
# Only then commit your changes
```

Never commit without testing - all tests must pass!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DMomot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
