## design-guard

> Design Guard is a design intelligence CLI and library for AI-generated web design.

# Design Guard — Claude Code Development Guide

## Project Overview

Design Guard is a design intelligence CLI and library for AI-generated web design.
It provides business research, design system synthesis, anti-slop validation,
DESIGN.md management, multi-generator support, and integration with Claude Code via skills.

**Stack**: Node.js 20+, TypeScript, Ink (React for terminal), Vitest
**MCP**: `@_davideast/stitch-mcp` for Stitch API access
**Auth**: Stitch API Key (env var `STITCH_API_KEY`) or gcloud OAuth

## Architecture

```
design-guard/                        # Monorepo root (npm workspaces)
├── CLAUDE.md                        # You are here
├── package.json                     # Workspace root — delegates to packages
├── tsconfig.json                    # Project references (core + cli)
├── tsconfig.base.json               # Shared TypeScript compiler options
├── .claude/
│   ├── agents/                      # Claude Code custom agents
│   │   └── dg-critic.md             # Post-generation quality gate agent
│   ├── rules/                       # Anti-slop design rules
│   └── skills/                      # Claude Code skills (dg-* prefix)
├── packages/
│   ├── core/                        # @design-guard/core — design intelligence library
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── vitest.config.ts
│   │   └── src/
│   │       ├── index.ts             # Public API barrel export
│   │       ├── research/
│   │       │   ├── types.ts         # Shared types (BusinessBrief, SiteAnalysis, etc.)
│   │       │   ├��─ business-researcher.ts  # Site scraping + brand analysis
│   │       │   ├── design-synthesizer.ts   # Research -> DESIGN.md synthesis
│   │       │   └── research-cache.ts       # Research result caching
│   │       ├── validation/
│   │       │   ├── design-validator.ts     # DESIGN.md quality scoring
│   │       │   └── output-validator.ts     # HTML output anti-slop validation
│   │       ├── templates/
│   │       │   ├── design-md.ts     # DESIGN.md template generator
│   │       ���   └── prompts.ts       # Prompt templates (initial, refinement, locale)
│   │       └── utils/
│   │           ├── validators.ts    # Zod schemas + prompt validation
│   │           └── prompt-enhancer.ts # Prompt improvement + slop risk scoring
│   └── cli/                         # design-guard CLI — depends on @design-guard/core
│       ├── package.json
│       ├── tsconfig.json
│       ├── vitest.config.ts
│       └── src/
│           ├── index.ts             # CLI entry point (commander + ink)
│           ├── commands/
│           │   ├── init.ts          # dg init — setup project, auth
│           │   ├── design.ts        # dg design — generate/import DESIGN.md
│           │   ├── discover.ts      # dg discover — research + synthesize DESIGN.md
│           │   ├── generate.ts      # dg generate — send prompts to Stitch
│           │   ├── build.ts         # dg build — build Astro site from screens
│           │   ├── preview.ts       # dg preview — local preview server
│           │   ├── research.ts      # dg research — auto-update knowledge base
│           │   ├── sync.ts          # dg sync — pull project state from Stitch
│           │   └── workflow.ts      # dg workflow — multi-step workflows
│           ├── tui/
│           │   ├── App.tsx          # Main Ink app component
│           │   ├── Dashboard.tsx    # Project overview, screen list, quota usage
│           │   ├── PromptBuilder.tsx # Interactive prompt composer with guardrails
│           │   ├── DesignEditor.tsx  # DESIGN.md section editor
│           │   └── components/
│           │       ├── ScreenCard.tsx
│           │       ├── QuotaMeter.tsx
│           │       ├── StatusBar.tsx
│           │       └── Spinner.tsx
│           ├── mcp/
│           │   ├── client.ts        # MCP client wrapper for Stitch tools
│           │   ├── tools.ts         # Tool call builders
│           │   └── auth.ts          # Auth handler (API key or OAuth)
│           ├── adapters/
│           │   ├── index.ts         # Adapter registry
│           │   ├── types.ts         # Adapter types
│           │   ├── astro.ts         # Astro site builder
│           │   ├── static.ts        # Static HTML site builder
│           │   └── nextjs.ts        # Next.js site builder
│           ├── research/
│           │   ├── crawler.ts       # Fetch and parse Stitch docs/changelog
│           │   ├── differ.ts        # Compare current vs new state
│           │   ├── updater.ts       # Apply updates to knowledge base
│           │   └── known-state.json # Last known state of Stitch features
│           ├── templates/
│           │   └── workflows.ts     # Workflow sequences (redesign, new app)
│           └── utils/
│               ├── config.ts        # Project config (.guardrc.json)
│               ├── logger.ts        # Structured logging
│               ├── quota.ts         # Track generation usage against limits
│               └── preview.ts       # Local preview server
├── docs/
│   ├── design-md-guide.md
│   ├── prompting-guide.md
│   └── known-state.md
├── public/                          # Landing page assets
├── scripts/                         # Repo scripts (demo.tape)
├── README.md
├── LICENSE
└── DESIGN.md
```

## Conventions

- All source in TypeScript with strict mode
- Use `import` not `require`
- Ink components use functional components with hooks
- All MCP calls go through `packages/cli/src/mcp/client.ts`, never direct fetch
- Config stored in `.guardrc.json` at project root
- Logs to stderr, output to stdout
- All user-facing strings in English (i18n not needed yet)

## Slash Commands for Claude Code

These are in `.claude/skills/` and are the primary interface when using
Design Guard inside Claude Code. All skills are prefixed with `dg-`.

### /dg-design
Input: Brand brief (name, industry, target audience, aesthetic adjectives)
Action: Generate a complete DESIGN.md following the 8-section spec
Output: Write `DESIGN.md` to project root

### /dg-generate
Input: Screen description (page type, sections, target user)
Action: Build prompt using zoom-out-zoom-in framework → send to Stitch MCP → receive HTML
Output: Save HTML to `screens/[screen-name].html`, show screenshot

### /dg-build
Input: Route mapping (which screens map to which URLs)
Action: Call `build_site` MCP tool → generate Astro project
Output: Astro site in `dist/`

### /dg-research
Input: None (or optional topic)
Action: Scrape Stitch docs, blog, forum → diff against known-state.json → update if changed
Output: Report of changes, updated knowledge base files

### /dg-sync
Input: Stitch project ID
Action: Pull all screens, design system, project metadata from Stitch
Output: Update local `screens/` and `.guardrc.json`

## MCP Integration

Design Guard uses `@_davideast/stitch-mcp` as the MCP server for Google Stitch integration.
When running inside Claude Code, the MCP server should be configured in
`~/.claude.json` or project-level `.mcp.json`:

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["@_davideast/stitch-mcp", "proxy"]
    }
  }
}
```

For API key auth (simpler):
```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["@_davideast/stitch-mcp", "proxy"],
      "env": {
        "STITCH_API_KEY": "${STITCH_API_KEY}"
      }
    }
  }
}
```

Alternatively, for direct HTTP MCP (no wrapper):
```json
{
  "mcpServers": {
    "stitch": {
      "type": "http",
      "url": "https://stitch.googleapis.com/mcp",
      "headers": {
        "X-Goog-Api-Key": "${STITCH_API_KEY}"
      }
    }
  }
}
```

### Available MCP Tools (from Stitch)

Core tools exposed by the Stitch MCP server:
- `list_projects` — List all Stitch projects
- `get_project` — Get project details
- `generate_screen_from_text` — Generate a screen from a text prompt
- `list_screens` — List screens in a project
- `get_screen` — Get screen details
- `get_screen_code` — Get HTML/CSS of a screen (proxy tool)
- `get_screen_image` — Get screenshot as base64 (proxy tool)
- `build_site` — Map screens to routes, generate Astro site (proxy tool)

## DESIGN.md Specification

Every DESIGN.md must contain these 8 sections:

1. **Visual Theme & Atmosphere** — Overall aesthetic direction (2-3 sentences)
2. **Color Palette & Roles** — Hex values with semantic names (Primary, Secondary, Surface, Error, etc.) in a markdown table
3. **Typography** — Font families, sizes, weights, line heights. Specific values, not vague descriptions
4. **Spacing & Layout** — Base unit, scale, max-width, grid system, breakpoints
5. **Component Patterns** — Buttons, cards, inputs, navbars — structure and style rules
6. **Iconography** — Icon style (outline/solid/duotone), size, source library
7. **Imagery Guidelines** — Photo style, illustration style, aspect ratios, treatment
8. **Do's and Don'ts** — Explicit rules to prevent AI generation mistakes

### Validation Rules
- All colors must be hex values (no "trustworthy blue")
- Font sizes in rem or px
- Spacing in px or rem
- At least 5 color roles defined
- At least 3 component patterns
- At least 3 Do's and 3 Don'ts

## Prompt Guardrails

Design Guard enforces these rules on all prompts sent to generators:

1. **Max length**: 5000 characters. Reject longer prompts with suggestion to split
2. **One screen per prompt**: Detect multi-screen intent and warn
3. **One change per refinement**: Detect compound changes and suggest splitting
4. **No vague refinements**: Reject "make it better", "improve it", require specifics
5. **Specificity check**: Flag generic terms, suggest UI/UX vocabulary replacements

## Auto-Research Protocol

The research module checks for Stitch updates by:

1. Fetching `https://stitch.withgoogle.com/docs/` (main docs)
2. Fetching `https://blog.google/technology/ai/stitch/` (announcements)
3. Searching `site:discuss.ai.google.dev stitch` (forum)
4. Comparing extracted info against `packages/cli/src/research/known-state.json`
5. If differences found:
   - Update `known-state.json` with new data and timestamp
   - Update `docs/known-state.md` human-readable version
   - Log changes to console
   - If breaking change detected, warn loudly

### known-state.json Schema
```json
{
  "lastUpdated": "2026-04-09T00:00:00Z",
  "models": [
    { "id": "GEMINI_3_PRO", "quotaMonthly": 50, "notes": "High quality" },
    { "id": "GEMINI_2_5_FLASH", "quotaMonthly": 350, "notes": "Fast" }
  ],
  "exportOptions": ["figma", "html_css", "ai_studio", "antigravity"],
  "mcpTools": ["list_projects", "get_project", "generate_screen_from_text", "list_screens", "get_screen"],
  "proxyTools": ["get_screen_code", "get_screen_image", "build_site"],
  "designMdSections": 8,
  "knownLimitations": [
    "No JS-heavy interactive pages",
    "Login-walled URLs fail extraction",
    "Long prompts cause component omissions"
  ]
}
```

## Testing Strategy

- **Unit tests**: Validators, template generators, prompt builders, diff logic
- **Integration tests**: MCP client with mocked server responses
- **No E2E tests against live Stitch** — use fixtures for reproducibility
- Mock MCP responses in `packages/cli/tests/fixtures/stitch-response.json`
- Test quota tracking with simulated generation sequences
- Test research differ with before/after state snapshots

Run all: `npm test` (delegates to both workspaces)
Run core only: `npm run test:core`
Run CLI only: `npm run test:cli`
Run single: `npm run test:run --workspace=packages/core -- tests/unit/prompts.test.ts`

## Development Workflow

1. `npm install`
2. Copy `.env.example` to `.env`, add `STITCH_API_KEY`
3. `npm run dev` — run TUI in dev mode
4. `npm test` — run all tests
5. `npm run build` — compile TypeScript
6. `npm link` — make `forge` command available globally

## Implementation Order

Build in this sequence:

1. **Phase 1 — Foundation**
   - package.json, tsconfig, vitest config
   - src/utils/ (config, logger, validators)
   - src/templates/ (design-md, prompts, workflows)
   - Unit tests for validators and templates

2. **Phase 2 — MCP Integration**
   - src/mcp/ (client, tools, auth)
   - Integration tests with mocked responses
   - src/commands/init.ts (setup auth and config)

3. **Phase 3 — Core Commands**
   - src/commands/design.ts
   - src/commands/generate.ts
   - src/commands/build.ts
   - src/commands/sync.ts
   - .claude/commands/ (slash command definitions)

4. **Phase 4 — TUI**
   - src/tui/ components (Ink)
   - Dashboard, PromptBuilder, DesignEditor
   - Wire to commands

5. **Phase 5 — Auto-Research**
   - src/research/ (crawler, differ, updater)
   - known-state.json baseline
   - docs/ knowledge base files
   - Research tests

6. **Phase 6 — Polish**
   - Error handling and recovery
   - Quota enforcement
   - Help text and onboarding UX

---
> Source: [FReptar0/design-guard](https://github.com/FReptar0/design-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
