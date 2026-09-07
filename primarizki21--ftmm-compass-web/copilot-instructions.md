## ftmm-compass-web

> Academic-advisor **mockup** for FTMM (Fakultas Teknologi Maju dan Multidisiplin), Universitas Airlangga. React 19 + Vite 8 + Tailwind CSS v4 SPA, frontend-only: **no backend, no persistence, all data mocked**. Born in Figma Make; standalone-cloneable.

# FTMM Compass — Agent Guide

Academic-advisor **mockup** for FTMM (Fakultas Teknologi Maju dan Multidisiplin), Universitas Airlangga. React 19 + Vite 8 + Tailwind CSS v4 SPA, frontend-only: **no backend, no persistence, all data mocked**. Born in Figma Make; standalone-cloneable.

## Commands

```bash
pnpm install        # deps (pnpm-lock.yaml is source of truth; .mise.toml pins node 22 + pnpm)
pnpm dev            # Vite on 0.0.0.0:$PORT (default 8443, strictPort)
pnpm build          # production build to dist/
pnpm preview        # serve dist/
pnpm format         # oxfmt
```

Inside Figma Make a dev server is already running on `$PORT`; in a normal clone you start it yourself. `vite.config.ts` requires `.figma/make/site.json` at import time — it is committed; do not delete it until the planned Figma-Make decoupling lands.

## Architecture

No router. `src/App.tsx` owns all top state and swaps pages via `useState<Page>`:

```
main.tsx → App.tsx ─┬─ isLoggedIn=false → pages/Login.tsx (fake auth: any input, 1s delay)
                    └─ sidebar nav (useState) ─┬─ Dashboard       (stats + timetable widget)
                                               ├─ CourseFinder    (search/filter/modal/CourseDetailView/PrerequisiteDiagram)
                                               ├─ DegreePlanner   (drag-drop roadmap + timetable tab)
                                               └─ Chatbot         (keyword-matched canned replies)
```

State contract (App.tsx):
- `pendingCourses: Course[]` — courses added in CourseFinder, consumed by DegreePlanner.
- DegreePlanner auto-files `Wajib` (compulsory) courses into their fixed semester via `useEffect`, keeps `Pilihan` (elective) in a drag bank; drops are parity-constrained (`odd`→semesters 1/3/5/7, `even`→2/4/6/8).
- Nothing survives reload.

Data lives in two places (known fragmentation): catalog + schedule in `src/data.ts` (`MOCK_COURSES`: 6 courses, `SCHEDULE`: 4 items with one **intentional** conflict II4042×II4045 Selasa 10:00 that drives conflict UI); semester-plan seed inline in `DegreePlanner.tsx` `INITIAL_PLAN` (courses MA1101/FI1101/MA1201/IF1210 exist only there).

## Design system

Tailwind v4 `@theme` in `src/index.css` — no tailwind.config. Palette: navy `#0f3e32`, gold `#d7b03d`, teal `#93f08e`, orange/danger `#ad5712`, warm off-white bg `#faf9f7`. Fonts: headings Poppins (`--font-serif`), body Inter, mono JetBrains Mono (Google Fonts @import). Use tokens as utilities (`bg-navy`, `text-gold`, …) — never hardcode hex in JSX. Class merging helper: `cn()` in `src/utils.ts`.

## Code conventions

- Default exports for components.
- Double-quote strings containing apostrophes (or escape them) — unescaped `'` in single quotes breaks the build.
- Closed tags, balanced braces; UI text is Indonesian, code identifiers English.
- Format with `pnpm format` before handing off; no linter/test suite exists.

## Known mock boundaries (intentional — do not "fix" silently)

| Symptom | Reality |
|---|---|
| Login accepts anything | Auth is simulated (`Login.tsx`) |
| Chatbot answers 2 topics only | Keyword rules in `Chatbot.tsx` (`data science|rekomendasi`, `krs|planning`) |
| Dashboard numbers never move | Hardcoded (84/144 SKS, IPK 3.75, sem 5) |
| `TimetableBuilder.tsx` unreachable | Orphaned by design decision 2026-08-26; its grid is duplicated in Dashboard widget + DegreePlanner tab |
| Schedule shows a clash | Seeded deliberately to demo conflict styling |

Planned future work (user-stated): remove the Figma Make integration (`.figma/`, plugins in `vite.config.ts`) entirely.

## Tools for agents
- **codegraph** (indexed; `.codegraph/` gitignored): query architecture/symbols/call-paths first via the `codegraph_explore` MCP tool with `projectPath` = this repo root. Refresh after substantial changes: `~/.codegraph/versions/v1.5.0/bin/codegraph init .`
- **LSP**: TypeScript server available — use for definitions/references/rename instead of text search.
- **Browser**: verify UI changes against the real surface. Flow: open `http://localhost:8443` → submit login form (any values) → nav buttons are `nav button:nth-of-type(1..4)` in order Dashboard/Course Finder/Degree Planner/Compass AI.
- **grep/read**: fallback for everything else; `src/` is small (~1.9k lines across 11 TS/TSX files).

---
> Source: [Primarizki21/ftmm-compass-web](https://github.com/Primarizki21/ftmm-compass-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
