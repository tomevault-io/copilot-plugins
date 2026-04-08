## sow-importer

> Web utility to import Statement of Work (SOW) documents into ScopeStack. Users upload Word/PDF files, AI extracts services/subservices, users review/edit mappings, then export JSON for ScopeStack API.

# SOW Importer - Agent Guidelines

## Project Overview
Web utility to import Statement of Work (SOW) documents into ScopeStack. Users upload Word/PDF files, AI extracts services/subservices, users review/edit mappings, then export JSON for ScopeStack API.

## Tech Stack
- **Framework:** Next.js 14+ with App Router
- **Language:** TypeScript (strict mode)
- **Styling:** Tailwind CSS with CSS custom properties
- **Deployment:** Vercel
- **LLM:** OpenRouter API

## Commands
```bash
npm run dev          # Start development server
npm run build        # Production build
npm run typecheck    # TypeScript type checking
npm run lint         # ESLint
npm test             # Run tests
```

## Project Structure
```
app/                 # Next.js App Router pages
  api/               # API routes (upload, analyze)
  review/            # Review/edit page
  export/            # Export page
components/          # React components
lib/                 # Utilities, parsers, API clients
  parsers/           # Document parsers (docx, pdf)
  prompts/           # LLM prompts
types/               # TypeScript type definitions
scripts/ralph/       # Ralph autonomous agent scripts
tasks/               # PRD and documentation
```

## Code Conventions
- Use functional components with TypeScript
- Prefer named exports
- Use CSS custom properties for theming (defined in globals.css)
- All API routes use Route Handlers (app/api/*/route.ts)
- Zod for runtime validation
- Preserve Markdown formatting in service descriptions

## Design Guidelines
- Avoid generic fonts (Inter, Roboto, Arial)
- Use distinctive typography that elevates the experience
- Color-code entity types: Phases, Services, Subservices
- Highlight ambiguous items with warning colors
- Add polish: animations, transitions, hover states

## Environment Variables
```
OPENROUTER_API_KEY=  # OpenRouter API key for LLM calls
```

## Key Files
- `/tasks/prd-sow-importer.md` - Full PRD with requirements
- `/scripts/ralph/progress.txt` - Ralph's learning log

## ScopeStack API Format
All exports use JSON:API format. See PRD for full schema:
- `project-services`: POST /v1/project-services
- `project-subservices`: POST /v1/project-subservices

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scopestack-jon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/scopestack-jon)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
