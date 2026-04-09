## inspector-gadget

> Next.js 16 TypeScript React application for analyzing construction documents and contractor bids. Integrates with OpenAI API for document intelligence.

# Inspector Gadget

Next.js 16 TypeScript React application for analyzing construction documents and contractor bids. Integrates with OpenAI API for document intelligence.

## Quick Start

```bash
npm install
npm run dev  # Dev server on http://localhost:3000
npm run build && npm run start  # Production build
npm run lint  # ESLint check
```

## Project Structure

```
app/
  - (tools)/          # Tool pages (bid-checker, etc)
  - api/              # API routes for document analysis
    - parse-pdf/      # PDF parsing endpoint
    - analyze/        # OpenAI-powered document analysis
    - bid-checker/    # Contractor bid checking
components/           # Shadcn/UI + custom React components
lib/                  # Utilities and helpers
docs/                 # Reference docs (agents, setup, OpenAI)
```

## Key Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | Start development server (hot reload) |
| `npm run build` | Production build |
| `npm run lint` | Run ESLint |

## Environment Setup

Create `.env.local` with:
```
OPENAI_API_KEY=sk_...  # Required for document analysis features
```

See `.env.example` for template.

## Tech Stack

- **Framework:** Next.js 16 (App Router)
- **Language:** TypeScript 5
- **UI:** React 19 + Shadcn/UI + TailwindCSS 4
- **PDF Handling:** pdf-parse, jspdf, html2canvas
- **AI/ML:** OpenAI API integration
- **Styling:** Class-variance-authority + Tailwind Merge
- **Deployment:** Vercel (vercel.json configured)

## Architecture Patterns

### PDF Processing Flow
- Client uploads PDF → `api/parse-pdf` → returns text + metadata
- Text sent to OpenAI → `api/analyze` → structured analysis results
- Results displayed in tool-specific UI

### Tool Structure
- Each tool lives in `app/(tools)/[tool-name]/page.tsx`
- Tools:
  - **bid-checker** - Analyzes contractor bids (Linear ticket: bid-checker)
  - **inspection-report-analyser** - Analyzes inspection reports (Linear ticket: inspection-report-analyser)
- Shared components in `components/`

## Testing & Quality

- **Linting:** ESLint (using Next.js config)
- **Build:** Next.js compilation catches TypeScript errors
- **No unit tests configured** - add if needed

## Deployment

- **Target:** Vercel (configured in vercel.json)
- **Build:** `npm run build`
- **Environment:** OPENAI_API_KEY required in Vercel env vars

## Known Patterns & Gotchas

- **PDF Parse Library:** Using v1 (not v2) due to DOMMatrix errors in serverless environment
- **Iframe Support:** App allows sfwconstruction.com to iframe it (CORS configured)
- **State Labels:** Phone numbers use state-specific formatting
- **Component Library:** Using Shadcn CLI for component generation

## Workflow & Ticketing

**Linear Project:** Inspector Gadget

**Active Tickets (Tools):**
- **bid-checker** - Contractor bid analysis tool
- **inspection-report-analyser** - Construction inspection report analysis tool

**Documentation Workflow:**
- Write project documentation in the `docs/` directory (organized by tool/feature)
- Update the corresponding Linear ticket with:
  - Commit reference (git commit hash)
  - Path to documentation file
- Documentation is the source of truth for feature context

## Docs Reference

- `docs/agents/setup.md` - Initial setup instructions
- `docs/agents/openai.md` - OpenAI integration details
- `docs/agents/components.md` - Component patterns
- See git commit messages for feature details (e.g., "feat: add contractor bid checker")

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tbhogstrom)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tbhogstrom)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
