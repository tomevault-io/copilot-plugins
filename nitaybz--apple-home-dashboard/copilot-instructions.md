## apple-home-dashboard

> Custom Home Assistant dashboard strategy that recreates the Apple Home app experience. A Lovelace plugin distributed via HACS that auto-generates dashboards from HA entities.

# Apple Home Dashboard - Project Instructions

## Project Overview
Custom Home Assistant dashboard strategy that recreates the Apple Home app experience. A Lovelace plugin distributed via HACS that auto-generates dashboards from HA entities.

## Tech Stack
- **Language:** TypeScript (strict mode, ES2020 target)
- **Architecture:** Native Web Components (Custom Elements + Shadow DOM) - NO React/Vue/Angular
- **Build:** Webpack 5 with ts-loader, single bundle output (`dist/apple-home-dashboard.js`)
- **Styling:** CSS-in-JS via Shadow DOM (template literals), container queries for responsive layout
- **Drag & Drop:** SortableJS
- **Platform:** Home Assistant Lovelace Strategy API
- **Distribution:** HACS (Home Assistant Community Store)

## Project Structure
```
src/
  apple-home-strategy.ts    # Entry point - strategy registration
  components/               # Core web components (AppleHomeCard, AppleHomeView)
  config/                   # Device groups, entity data logic
  pages/                    # Page renderers (Home, Room, Group, Scenes, Cameras)
  sections/                 # UI sections (Header, Chips, Favorites, Status, Area)
  utils/                    # Manager singletons (Edit, DragDrop, Customization, Snapshot, etc.)
  types/                    # TypeScript interfaces
  translations/             # i18n language files
```

## Key Patterns & Conventions
- **Web Components:** All UI elements are Custom Elements with Shadow DOM. No external UI framework.
- **Manager Pattern:** Singleton managers for cross-cutting concerns (CustomizationManager, SnapshotManager, EditModeManager, etc.)
- **Strategy Pattern:** `generateLovelaceDashboard()` returns pure config objects for HA to render.
- **Responsive Design:** Container queries (not media queries) with 5 breakpoints (XL/Desktop/Tablet/Mobile/XS).
- **RTL Support:** Full right-to-left layout support via RTLHelper.
- **i18n:** LocalizationService with translation files, auto-detects HA language.
- **Storage:** User customizations persisted via Home Assistant storage API (`hass.connection`).
- **Single Bundle:** Webpack configured with `LimitChunkCountPlugin(maxChunks: 1)` - everything in one file.

## Development Commands
- `npm run build` - Production build
- `npm run build:dev` - Dev build with watch mode
- `npm run lint` - ESLint
- `npm run format` - Prettier
- `npm run bump` - Version bump (updates package.json, README, strategy source)
- `npm run deploy` - Deploy via deploy.sh

## Skill Configuration

### Complementary Skills (Work Together with CC10x)

| When task involves... | Invoke |
|-----------------------|--------|
| UI components, layouts, styling, visual design | `frontend-design:frontend-design` |
| TypeScript web components, Shadow DOM, CSS patterns | `cc10x:frontend-patterns` |
| New entity support, page/section creation | `cc10x:code-generation` |
| Strategy architecture, manager patterns | `cc10x:architecture-patterns` |
| HA integration bugs, rendering issues | `cc10x:debugging-patterns` |
| Pre-commit code quality checks | `cc10x:code-review-patterns` |
| Feature planning, multi-step implementation | `cc10x:planning-patterns` |
| Git commits and PRs | `commit-commands:commit` or `commit-commands:commit-push-pr` |

### Skills NOT relevant to this project
- `mongodb-*` - No database layer
- `react-best-practices` - Uses native Web Components, not React

## Deployment
- After `npm run build`, SCP the `dist/apple-home-dashboard.js` to the HA server and update the hacstag file for cache busting
- **No need to restart Home Assistant** - this is a frontend resource. A browser hard-refresh (or hacstag update) is enough to pick up the new JS.

## Important Notes
- All CSS is scoped via Shadow DOM - global styles don't work here
- The `hass` object is the primary data source (entities, config, services, connection)
- Entity domains each have specific rendering logic in `DashboardConfig.ts`
- Cards support two design types: `REGULAR` and `TALL` (controlled by user customization)
- Camera snapshots use a dual-image crossfade technique for smooth updates
- Drag-and-drop uses SortableJS with custom touch handling
- Always test RTL layout when modifying CSS
- The bundle must remain a single file for HACS compatibility

---
> Source: [nitaybz/apple-home-dashboard](https://github.com/nitaybz/apple-home-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
