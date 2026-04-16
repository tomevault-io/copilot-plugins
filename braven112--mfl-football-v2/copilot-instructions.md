## mfl-football-v2

> ALL features, utilities, and data structures should be designed with the **Auction Price Predictor** in mind. Every function must be **reusable** and **composable**.

# MFL Football v2

## Development Principle

ALL features, utilities, and data structures should be designed with the **Auction Price Predictor** in mind. Every function must be **reusable** and **composable**.

---

## Year Rollover System

Two critical dates drive year transitions:

| Date | Event | What Changes |
|------|-------|--------------|
| **Feb 14th @ 8:45 PT** | New MFL league created | `getCurrentLeagueYear()` updates |
| **Labor Day** | NFL season starts | `getCurrentSeasonYear()` updates |

### Decision Framework

**Use `getCurrentLeagueYear()`** for:
- Rosters, contracts, salary cap, auctions, trade analysis
- Key question: *"Does this page help manage my roster?"*

**Use `getCurrentSeasonYear()`** for:
- Standings, playoffs, MVP tracking, draft order
- Key question: *"Does this page show results from games played?"*

```typescript
import { getCurrentLeagueYear, getCurrentSeasonYear, getNextDraftYear, getNextAuctionYear } from '../utils/league-year';
```

Test date-dependent features with `?testDate=YYYY-MM-DD` URL parameter.

---

## Team Name Display

**CRITICAL:** Always use `chooseTeamName()` to prevent UI overflow:

```typescript
import { chooseTeamName } from '../utils/team-names';

const displayName = chooseTeamName({
  fullName: team.name,
  nameMedium: assets?.nameMedium,  // ≤15 chars (default)
  nameShort: assets?.nameShort,    // ≤10 chars
  abbrev: assets?.abbrev,          // 2-6 chars
}, 'default'); // Context: 'default' | 'short' | 'abbrev'
```

Config locations:
- TheLeague: `src/data/theleague.config.json`
- AFL Fantasy: `data/afl-fantasy/afl.config.json`

---

## Editorial Design Standard

**CRITICAL:** All new pages and components must follow the **editorial design language** derived from the PlayerDetailsModal system. This is the visual standard for the entire site.

**Canonical reference:** `src/components/theleague/PlayerDetailsModal.astro`
**Full pattern catalog:** [design-system.md](docs/claude/insights/domains/design-system.md) (search "Editorial Design Standard")
**Component guide:** [components.md](docs/claude/components.md) (Editorial Design Standard section)

### Signature Patterns

**Section titles** — Uppercase, left-border accent:
```css
font-size: 0.75rem;
font-weight: 700;
text-transform: uppercase;
letter-spacing: 0.06em;
padding-left: 0.625rem;
border-left: 2px solid var(--color-primary, #1c497c);
```

**Section titles with subtitles** — When a section title needs a description, wrap both in `.section-header` so the left-border spans both lines:
```html
<div class="section-header">
  <h3 class="section-header__title">NFL Analysis</h3>
  <p class="section-header__sub">Players on the same NFL team</p>
</div>
```
The subtitle is `0.8125rem`, `gray-400`, with `0.25rem` gap. Use standalone section title (above) when there's no subtitle.

**Detail rows** — Flex rows with fixed-width uppercase labels (gray-400) + flexible values, separated by gray-50 borders

**Key metrics** — 3-column grid, gray-50 bg cards, large tabular-nums values + micro uppercase labels

**Tables** — Sticky uppercase headers (0.625rem, gray-400), hover rows, tabular-nums for numbers

**Numbers** — Always `font-variant-numeric: tabular-nums`

**Defensive CSS** — Always include token fallbacks: `var(--color-gray-700, #374151)`

**Modal/overlay backdrop** — **CRITICAL:** Every modal and overlay MUST use the frosted-glass backdrop blur. Never use a plain dark overlay without blur:
```css
background: rgba(15, 23, 42, 0.45);
backdrop-filter: blur(2px);
```

### Typography Scale

| Role | Size | Weight | Notes |
|------|------|--------|-------|
| Hero/page title | 1.35rem | 700 | line-height: 1.2 |
| Section title | 0.75rem | 700 | UPPERCASE + left border |
| Body/values | 0.875rem | 400–500 | |
| Detail label | 0.75rem | 600 | UPPERCASE, gray-400 |
| Table header | 0.625rem | 600 | UPPERCASE, gray-400 |

---

## Player Display (Player Lockup)

**CRITICAL:** Whenever displaying players in a list, card, or table, use the standard **Player Lockup** pattern for consistency:

### Layout
| Left | Right (Row 1) | Right (Row 2) |
|------|---------------|---------------|
| Circular headshot (spans 2 rows) | **Player name** (bold) | NFL team logo (16px) + Position |

### Astro Component
Use `PlayerCell.astro` — it handles DEF position, team code normalization, and optional modal support automatically:

```astro
import PlayerCell from '../components/theleague/PlayerCell.astro';

<!-- Basic usage (DEF handling is automatic) -->
<PlayerCell name={player.name} headshot={player.headshot} position={player.position} nflTeam={player.nflTeam} />

<!-- Compact size -->
<PlayerCell name={player.name} headshot={player.headshot} position={player.position} nflTeam={player.nflTeam} size="compact" />

<!-- With modal support + badges -->
<PlayerCell name={player.name} headshot={player.headshot} position={player.position} nflTeam={player.nflTeam} playerData={playerObj}>
  <span slot="after-name" class="injury-badge">Q</span>
</PlayerCell>
```

### JS Utility (for client-side rendering)
Use `buildPlayerCellHTML()` when building HTML strings in `<script>` tags:

```typescript
import { buildPlayerCellHTML } from '../utils/player-cell-html';
import { initPlayerModalTrigger } from '../utils/player-modal-trigger';

const html = buildPlayerCellHTML({ name, headshot, position, nflTeam, playerData });
// Attach delegated click handler once on the parent container:
initPlayerModalTrigger(tableBody);
```

### DEF Handling
Team defenses (position=DEF) are handled automatically by `PlayerCell` and `buildPlayerCellHTML` — the avatar swaps to the NFL team logo and the meta row hides the duplicate logo. Callers just pass the raw `position` and `nflTeam`.

### Size Variants
- `default`: 40px avatar (36px mobile)
- `compact`: 32px avatar (28px mobile)
- Parents can also override via CSS: `.my-table .player-cell { --player-avatar-size: 2rem; }`

### Key Files
- Component: `src/components/theleague/PlayerCell.astro`
- Shared CSS: `src/styles/player-cell.css`
- JS utility: `src/utils/player-cell-html.ts`
- Modal trigger: `src/utils/player-modal-trigger.ts`
- Code normalization: `src/utils/nfl-logo.ts` → `normalizeTeamCode()`
- Logo assets: `public/assets/nfl-logos/{CODE}.svg`
- Player modal: `src/components/theleague/PlayerDetailsModal.astro`

---

## League Context

Two leagues share this codebase:

| League | Slug | MFL ID | Data Path |
|--------|------|--------|-----------|
| TheLeague | `theleague` | 13522 | `src/data/theleague/` |
| AFL Fantasy | `afl` | 19621 | `data/afl-fantasy/` |

---

## Key Utilities

| Utility | Purpose |
|---------|---------|
| `src/utils/league-year.ts` | Year rollover logic |
| `src/utils/team-names.ts` | Team name display |
| `src/utils/salary-calculations.ts` | Cap math (10% escalation) |
| `src/utils/auth.ts` | Authentication |
| `src/utils/team-preferences.ts` | Cookie-based preferences |
| `src/utils/league-context.ts` | Dual-league support |

---

## AI Insights System

**IMPORTANT:** Before starting any task, read relevant insight files. After completing work, record learnings.

```
docs/claude/insights/
├── domains/           # Cross-cutting knowledge
│   ├── frontend.md        # UI patterns, components
│   ├── design-system.md   # Tokens, CSS variables
│   ├── mfl-api.md         # MFL API quirks
│   └── accessibility.md   # A11y patterns
└── features/          # Feature-specific learnings
    ├── nav-redesign.md    # Navigation drawer
    └── {feature}.md       # New features
```

**Workflow:**
1. **Before task:** Read `domains/` files for relevant domains + `features/{feature}.md` if exists
2. **After task:** Add new insights using the format in `docs/claude/insights/README.md`

---

## Page Directory Registry

**IMPORTANT:** When adding a new page to the site, you MUST also add an entry to `src/data/page-directory.json`.

Each entry requires: `id`, `title`, `description`, `path`, `icon`, `category` (popular | my-team | reports | tools | info), `tags` (10+ synonyms), `visibility` (all | admin), and `popularity` (0-100).

**Write tags generously** — include every word a user might type to find this page:
- Primary function words (what the page does)
- Synonyms (alternative words users might search)
- Data types shown (chart, table, graph, list)
- Actions available (edit, filter, sort, compare)
- Related concepts and slang/casual terms

The test `tests/page-directory-data.test.ts` enforces minimum 10 tags per entry and validates all fields.

---

## What's New Changelog

**IMPORTANT:** When completing a feature that adds a new page, new user-facing feature, or significant enhancement, you MUST update `src/data/whats-new.json`.

### When to Add a New Entry
- New page added to the site (screenshot required)
- New user-facing feature (e.g., a new tool, mode, or interactive element) (screenshot required)
- Major enhancement that changes how an existing feature works (screenshot required)
- Any guest-facing change that affects user behavior (e.g., URL changes, navigation changes)

### When NOT to Add an Entry
- Style tweaks, data syncs, refactors
- Internal tooling or build changes
- Documentation-only changes
- Admin-only features (visibility: "admin" in nav-config.json)
- Unreleased or in-progress features not yet available to all users

### Writing Style (MANDATORY)

Every What's New entry MUST be written in the league's established editorial voice. This is not optional — it applies to every new-page, new-feature, and enhancement entry.

**Voice:** Conversational, witty, self-aware humor. Slightly sarcastic but never mean-spirited. Written like a sports columnist who actually understands the product.

**Structure (2-3 paragraphs in `description`):**
1. **Opening hook** — A humorous problem statement or analogy about the old way / the pain point. Draw from real-world comparisons or fantasy football culture. Examples: "used to require twelve browser tabs, three spreadsheets, and the quiet resignation that you'd never have all the data in one place", "had all the navigability of a phone book".
2. **Feature details** — What it does, explained with specific capabilities in user terms. Technical details wrapped in accessible language. Be concrete about what the user can do.
3. **Closing** — A callback to the opening joke, a wry practical observation, or a nudge to try it. Examples: "Use it before the auction, or don't — and then use it after the auction to figure out where things went wrong", "Trust, but verify."

**Hallmarks to include:**
- Real-world comparisons ("the difference between a folding map and a GPS")
- Fantasy football / sports metaphors ("all the reliability of a rookie quarterback in a snow game")
- League member shoutouts when a feature was built for or inspired by someone ("This one's for Wabbit", "This one started with a simple request from The Dream")
- Light ribbing of league culture and habits
- Personality in the `summary` field too — not just a dry feature description

**Tone calibration:**
- Bug fixes can be shorter and drier, with one good joke
- New pages and features get the full 2-3 paragraph treatment
- Enhancements land somewhere in between
- `excludeFromHero` entries (minor polish) can be more concise

**Anti-patterns (DO NOT):**
- Write dry, corporate-sounding release notes
- Use generic language like "We're excited to announce..."
- Skip the humor — every entry needs personality
- Write the `summary` as a plain feature description without voice

### Entry Format
Add the new entry at the **top** of the array (newest first):
```json
{
  "id": "kebab-case-id",
  "date": "YYYY-MM-DD",
  "title": "Short Title",
  "summary": "One sentence shown in hero banner and cards.",
  "description": ["Full paragraph(s) for the What's New page."],
  "category": "new-page | new-feature | enhancement | bug-fix",
  "link": "/theleague/page-path",
  "linkLabel": "CTA text (e.g., 'Try it now')",
  "icon": "sprite-icon-id",
  "image": "feature-name.webp",
  "imageAlt": "Descriptive alt text for the screenshot"
}
```

### Screenshot Requirement (MANDATORY)

**Every `new-page`, `new-feature`, and `enhancement` entry MUST include a screenshot.** This is enforced by `tests/whats-new-data.test.ts` and the build will fail without it.

**Required fields:**
- `"image"`: Filename relative to `public/assets/whats-new/` (e.g., `"trade-builder.webp"`)
- `"imageAlt"`: Descriptive alt text explaining what the screenshot shows

**How to add a screenshot:**
1. Take a screenshot of the feature at a standard desktop viewport (1280px+ wide)
2. Save as `.webp` format to `public/assets/whats-new/{feature-id}.webp`
3. Use a 16:9 aspect ratio crop when possible
4. Add `"image"` and `"imageAlt"` fields to the JSON entry

**Where screenshots appear:**
- What's New listing page (card thumbnails with browser-frame chrome)
- What's New detail page (hero image)
- Homepage hero banner (when featured)

**Exempt:** `bug-fix` and `league-event` categories do not require screenshots.

### Hero Banner Behavior
- New entries automatically appear in the homepage hero for **7 days**
- If multiple features are within the 7-day window, one is **randomly selected** per page load
- Set `"excludeFromHero": true` for minor enhancements that shouldn't get hero treatment
- After 7 days, the hero falls back to upcoming league events or the default "What's New" promo
- Priority rules are defined in `src/utils/hero-resolver.ts`

### Weekly Bug Fix & Style Tweak Changelog

Bug fixes and style tweaks are tracked throughout the week and compiled into a single What's New rollup entry every Monday at 8pm PT via GitHub Actions (`scripts/weekly-changelog-rollup.mjs`).

**After completing any bug fix or style tweak** that does NOT qualify for its own What's New entry, append an entry to `src/data/weekly-changelog-staging.json`:

```json
{
  "date": "YYYY-MM-DD",
  "type": "bug-fix | style-tweak",
  "summary": "User-facing description of what changed and why it matters",
  "impact": "user | admin",
  "area": "free-agents | rosters | navigation | design-system | homepage | rankings | trade-builder | salary | league-summary | calendar | standings | playoffs | mvp | import-rankings | whats-new | other"
}
```

**Guidelines:**
- Write `summary` as a user-facing improvement, not a code change (e.g., "Defense player avatars are now circular with properly centered logos" NOT "use flex centering with container padding")
- `impact` is `"user"` for anything league members see; `"admin"` for commissioner-only changes
- Do NOT log: data syncs, refactors with no visible effect, test-only changes, or changes that already got their own What's New entry
- The staging file resets automatically every Monday after the rollup runs

### Weekly Rollup Screenshot (MANDATORY)

Each weekly rollup MUST include **one screenshot** of the most noteworthy fix or enhancement from that week.

**How to choose:** Pick the most visually interesting or impactful change from the staging entries. Take a screenshot of the affected page showing the improvement in action. For example, if the best change fixed player headshots, screenshot the player modal showing a working headshot.

**How to add:**
1. Take a screenshot of the affected page at a standard desktop viewport (1280px+ wide) using Playwright CLI:
   ```bash
   # Use Playwright to capture with any needed interactions (click modals, filters, etc.)
   node -e "const {chromium}=require('playwright'); ..."
   # Convert to webp
   cwebp -q 85 /tmp/screenshot.png -o public/assets/whats-new/weekly-rollup-YYYY-MM-DD.webp
   ```
2. Save as `.webp` to `public/assets/whats-new/weekly-rollup-YYYY-MM-DD.webp` (use the Monday date)
3. Set the `featuredImage` field in `src/data/weekly-changelog-staging.json`:
   ```json
   {
     "weekOf": "2026-03-03",
     "featuredImage": "weekly-rollup-2026-03-03.webp",
     "featuredImageAlt": "Descriptive alt text for the screenshot",
     "changes": [...]
   }
   ```

The rollup script will automatically include the `image` and `imageAlt` fields in the generated What's New entry.

---

## Documentation Index

For detailed documentation, see `docs/claude/`:

| Document | Contents |
|----------|----------|
| [build-dev.md](docs/claude/build-dev.md) | Build commands, npm scripts, dev workflow |
| [data-flow.md](docs/claude/data-flow.md) | MFL API, data sources, cache layer |
| [components.md](docs/claude/components.md) | Astro/React patterns, layouts, styling |
| [testing.md](docs/claude/testing.md) | Vitest, test patterns, coverage |
| [auth.md](docs/claude/auth.md) | Authentication, sessions, cookies |
| [code-standards.md](docs/claude/code-standards.md) | TypeScript, imports, naming conventions |
| [troubleshooting.md](docs/claude/troubleshooting.md) | Common issues, debug techniques |
| [critical-assumptions.md](docs/claude/critical-assumptions.md) | Hardcoded values ($45M cap, 10% escalation) |
| [league-rules.md](docs/claude/league-rules.md) | TheLeague rules, scoring, roster config |
| [afl-rules.md](docs/claude/afl-rules.md) | AFL Fantasy rules, scoring, roster config |
| [insights/](docs/claude/insights/) | AI learnings by domain and feature |

### Feature Documentation

Feature docs live in `docs/features/`:

| Document | Contents |
|----------|----------|
| [auction-predictor-design.md](docs/features/auction-predictor-design.md) | System architecture, algorithms |
| [auction-predictor-tasks.md](docs/features/auction-predictor-tasks.md) | Implementation tasks |
| [custom-rankings.md](docs/features/custom-rankings.md) | Custom rankings & tier system |
| [mfl-api.md](docs/features/mfl-api.md) | MFL API reference |
| [personalization.md](docs/features/personalization.md) | Team preference cookie system |

---

## Communication Rules

- **Always make links clickable** — when providing URLs, GitHub links, PR links, or any other references, format them as markdown hyperlinks (e.g., `[link text](url)`) so they are clickable in the terminal.

---

## Quick Reference

### Critical Constants
```typescript
SALARY_CAP = 45_000_000       // $45M
ROSTER_LIMIT = 28             // 28 players
ESCALATION_RATE = 1.10        // 10% annual
```

### Common Commands
```bash
pnpm dev          # Start dev server
pnpm build        # Production build
pnpm test         # Run all tests
pnpm sync:all     # Sync data from MFL
```

---

## Team Strategy (Fantasy Football)

This section describes the dynasty fantasy football strategy that informs feature priorities and analysis tools.

**Primary Goal:** Sign as many long-term contracts as possible by targeting **young, inexpensive players** to build sustained dynasty dominance.

**Secondary Goal:** Acquire good short-term contracts (1-2 years) that provide trade asset value, roster depth, and plug-and-play starters.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/braven112) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
