## math-trade-buddy

> Firefox extension (MV3) for BoardGameGeek Math Trades.

# Math Trade Buddy

Firefox extension (MV3) for BoardGameGeek Math Trades.

## Build Commands

```bash
npm install                      # Install dependencies
npm run dev:firefox              # Dev build with watch
npm run build:firefox            # Production build
npx wxt build --browser firefox  # Explicit build
```

Output goes to `.output/firefox-mv3/`. Load via `about:debugging#/runtime/this-firefox` → Load Temporary Add-on → select `.output/firefox-mv3/manifest.json`.

## Architecture

```
entrypoints/
├── popup/           # Extension popup (launcher + OLWLG fill button + style toggle)
├── manage/          # Full-page tab: settings, descriptions, push to BGG, popularity
├── content.ts       # OLWLG content script (fill textareas, extract popularity, CSS toggle, sidebar nav)
├── content/style.css # OLWLG restyling (scoped to .mtb-styled class, includes sidebar + table styles)
├── background.ts    # Routes messages: BGG push via bgg-bridge, popularity via hidden tab
└── bgg-bridge.content.ts  # Runs on boardgamegeek.com, POSTs with user's session cookies
utils/
├── bgg.ts                # BGG API: fetch collection, game details, ranks, format descriptions, push (optional)
├── manage-state.ts       # Pure state logic for manage page buttons/sync (tested)
├── olwlg.ts              # OLWLG page detection
├── description-tags.ts   # Pure parser: description text → condition + language tag (tested)
└── parser.ts             # math_trade_comments.txt parser (legacy, may remove)
```

## Key Patterns

- **BGG writes bypass Cloudflare** by using `bgg-bridge.content.ts` on a BGG tab — it POSTs from BGG's origin with session cookies
- **OLWLG data extraction** happens via background script opening a hidden tab, waiting for content script, sending a message, then closing the tab
- **Descriptions stored in `browser.storage.local`** under `descriptions` key (Record<number, string>). BGG push is an optional developer mode behind an Advanced toggle.
- **BGG push** (when enabled) writes to `fieldname: 'conditiontext'` (Trade Condition) via bgg-bridge. The push field value is `value`, not `conditiontext` — BGG silently ignores wrong field names.
- **OLWLG sidebar navigation** replaces the top navbar with a vertical sidebar when styling is enabled. Built dynamically from the original navbar links in `content.ts`.
- **Edit Wants tags + column resize** are added when styling is on. Tags are parsed from the page's inline `desc['ITEM_ID'] = '...'` script globals (not from stored descriptions) so they work for every trader's items. The Game column width is stored in `mtb_game_col_width` in `browser.storage.local`.
- CSS restyling is scoped to `body.mtb-styled` so it can be toggled without reload
- **Manage page state logic** is extracted into pure functions in `utils/manage-state.ts` with 36 unit tests covering approve/sync button states
- Popularity sort tie-breakers are always fixed direction (BGG rank ASC, wanted DESC) regardless of primary sort direction

## Conventions

- TypeScript, no framework (vanilla DOM manipulation)
- WXT handles manifest generation from file structure
- Unit tests with Vitest (`npm test`); state logic in `utils/manage-state.ts` is tested
- Test scripts in `scripts/` use Playwright with saved OLWLG session (`scripts/olwlg-session.json`, gitignored)
- `scripts/save-session.ts` — run once to capture OLWLG login for headless testing

---
> Source: [hancohogenbirk/math-trade-buddy](https://github.com/hancohogenbirk/math-trade-buddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
