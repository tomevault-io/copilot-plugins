## championslab

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

<!-- BEGIN:workflow-rules -->
# End-of-Task Verification

At the end of every task, fix, or code change — before marking the task complete — make sure the local development server (`localhost`) is running and the app loads without errors. If it is not running, start it (`npm run dev` or equivalent) and do a quick smoke-test in the browser.

# Changelog Updates

After every task, fix, or code change that is user-facing (UI changes, bug fixes, new features, data updates, engine improvements, i18n additions), update the shared changelog in `src/components/last-updated.tsx`. Add a new dated entry at the top of `SHARED_ENTRIES` with concise, emoji-prefixed bullet points describing what changed. The changelog modal appears on every page, so all changes should be summarized there for users.
<!-- END:workflow-rules -->

<!-- BEGIN:documentation-rules -->
# Task-Specific Documentation

Before starting any task, identify which domain it falls under and **read the corresponding documentation file(s) first**. This ensures you follow project conventions and avoid breaking existing patterns.

| If your task involves... | Read this file first |
|:---|:---|
| **Deploying** the app, VPS operations, production builds, or service restarts | `DEPLOYMENT.md` |
| **API routes**, database schema, Supabase, Row Level Security, or backend integrations | `API_AND_BACKEND.md` |
| **UI components**, shadcn/ui primitives, page components, styling, Tailwind, or Framer Motion | `COMPONENTS.md` |
| **Translations**, adding a new language, i18n context, or locale files | `I18N.md` |
| **Pokémon data**, roster updates, running scripts, sprite management, or data pipelines | `DATA_AND_SCRIPTS.md` |
| **Tournament teams**, syncing Limitless data, or updating `CHAMPIONS_TOURNAMENT_TEAMS` | `TOURNAMENT_SYNC.md` |
| **Sprites**, uploading new sprites to object storage, or sprite URL management | `SPRITES.md` |
| **Architecture decisions**, directory structure, routing, data flow, state management, or security | `ARCHITECTURE.md` |
| **Contributing** code, commit message conventions, or community processes | `CONTRIBUTING.md` |
| **General project context**, features, tech stack, or quick start | `PROJECT.md` or `README.md` |

**Rule of thumb:** If a dedicated guide exists for the area you're touching, read it before writing code. If the task spans multiple areas (e.g., adding a new API + UI), read all relevant guides.
<!-- END:documentation-rules -->

---
> Source: [Andrew21P/ChampionsLab](https://github.com/Andrew21P/ChampionsLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
