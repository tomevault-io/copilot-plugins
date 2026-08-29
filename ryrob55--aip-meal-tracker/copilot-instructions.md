## aip-meal-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Start development server (port 3002)
cd web && npm run dev

# Build for production
cd web && npm run build

# Lint
cd web && npm run lint

# Database commands (run from web/)
npm run db:push        # Apply schema changes (creates/updates tables)
npm run db:generate    # Regenerate Prisma client
npm run db:studio      # Open Prisma Studio GUI
npm run db:seed-aip    # Seed the AIP food database
```

### Docker Deployment

```bash
# Build and start all services (postgres + app)
docker compose up -d --build

# Rebuild just the app after code changes
docker compose up -d --build app

# View logs
docker compose logs -f app
```

The app runs on port 3002 (mapped from container 3000).

## Architecture

**Stack**: Next.js 14 (App Router) + Prisma + PostgreSQL

### Key Directories

- `web/src/app/` - Next.js pages and API routes
- `web/src/app/api/` - Backend API endpoints
- `web/src/components/` - React components
- `web/src/lib/` - Shared utilities (db.ts, llm.ts, types.ts, macro-calculator.ts, aip-foods.ts)
- `web/prisma/schema.prisma` - Database schema

### Core Data Models

- **Recipe** - Stores ingredients (JSON), instructions, tags, nutritional data, AIP compliance flag
- **MealPlan** -> **MealPlanItem** -> Recipe - Weekly dinner planning
- **GroceryList** -> **GroceryItem** - Auto-generated shopping lists with AIP quality preferences
- **AIPQuestionnaire** -> **AIPRestriction** - User's macro targets, fasting window, restrictions
- **AIPFood** - Food database with AIP phase, tyramine/histamine levels
- **AIPDailyLog** -> **AIPMealEntry** - Daily meal tracking with eaten/skipped status

### LLM Integration (Optional)

AI features are optional and gracefully degrade when no API key is set. Configure via:
- `LLM_BASE_URL` - Any OpenAI-compatible endpoint
- `LLM_API_KEY` - API key
- `LLM_MODEL` - Model name

AI powers: `suggestMeals()`, `generateRecipeDetails()`, `categorizeIngredient()`

### Main UI Pages

- `/` - Dashboard with links to all features
- `/aip-diet` - Daily meal tracking with week/day views
- `/aip-diet/onboarding` - Questionnaire for macro targets and restrictions
- `/aip-diet/foods` - AIP food database browser
- `/aip-diet/monthly` - Monthly calendar view of meals
- `/aip-diet/report` - Progress reports with macro trends
- `/recipes` - Recipe database with search and AI generation
- `/meals` - Weekly dinner planning with grocery lists

## Recipe Formatting Rules

When creating or modifying recipes:
- **No microwave**: NEVER include microwave instructions — always stovetop, oven, or other methods
- **No "AIP" in recipe names**: Use "AIP" only as a tag, never in the recipe title
- **Cauliflower pizza**: Uses pre-bought riced cauliflower (bags from Costco/Whole Foods), not whole heads
- **Instructions format**: Clean numbered format (1. Step. 2. Step.) — no section headers or sub-bullets

## Database Notes

- Use `npx prisma db push` (not `prisma migrate`) to apply schema changes
- Docker postgres runs on port 5433 (mapped from container 5432)
- Default DB: `postgresql://postgres:aip_password@localhost:5433/aip`

---
> Source: [ryrob55/aip-meal-tracker](https://github.com/ryrob55/aip-meal-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
