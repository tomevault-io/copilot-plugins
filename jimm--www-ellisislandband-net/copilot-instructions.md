## www-ellisislandband-net

> Jekyll site for https://ellisislandrock.com. Generated HTML is in `_site` (ignore it).

# Ellis Island Band Website

Jekyll site for https://ellisislandrock.com. Generated HTML is in `_site` (ignore it).

## Site Structure

- `_config.yml` - Jekyll config, kramdown processor, jekyll-sitemap plugin
- `_layouts/` - default.html (with image modal), page.html, post.html
- `_includes/` - head.html, header.html, footer.html, social links, content helpers
- Root `.md` files - Pages (index, about, schedule, song-list, gallery, etc.)

## React Architecture

### Structure (src/)
- `schedule/` - Schedule page components (ScheduleApp → ScheduleList → ScheduleItem → Date/Info/Text/PosterImage)
- `song-list/` - Song list components (SongListApp → SongTable → SongRow)
- `gallery/` - Gallery components (ParkCityGallery, RandomIndexImage)
- `shared/` - Common utilities and components
  - `components/ImageModal.jsx` - Full-screen image modal (replaces js/image_modal.js)
  - `hooks/useDataFetcher.js` - Generic fetch hook (local file → API fallback)
  - `utils/dataFetcher.js` - fetchFromSource function
  - `utils/htmlHelpers.js` - HTML entity unescaping (with optional newline conversion)
  - `index.jsx` - Entry point for shared components (mounts ImageModal)

### Key Details

**BandHelper Integration:**
- Schedule: Fetches calendar feed, marks gigs as private (either name == "Private Event" or some gig fields are not present)
- Song list: Fetches smart list, filters out "Learning" songs, sortable columns
- Both fall back to local JSON files (updated hourly by cron)
- Custom fields: `custom_cC99h9` (description), `custom_CCMx5n` (poster URL), `custom_Kz3bz0` (poster alt)

**Integration with Legacy Code:**
- Uses `window.marked.parseInline()` for Markdown in descriptions
- Exposes `window.modal_image()` and `window.openImageModal()` from ImageModal component
- Uses existing CSS classes (no React-specific styles)

**Build (Vite):**
- Output: `schedule.bundle.js`, `song-list.bundle.js`, `image-modal.bundle.js`, `park-city-gallery.bundle.js`, `random-index-image.bundle.js`, `assets/client-*.js`
- Format: ES modules (type="module")

## Development Workflow

**Critical: BandHelper uses snake_case** (`date_display`, `date_start`). Never convert to camelCase.

**Setup:**
1. `make server` (Jekyll on localhost:4000)
2. `npm run watch` (auto-rebuild React on file changes)
3. Edit `src/schedule/`, `src/song-list/`, `src/gallery/`, or `src/shared/`

**Adding Features:**
- New component → `src/{schedule|song-list}/components/`
- App-specific utility → `utils/[name]Helpers.js`
- Shared utility → `src/shared/utils/` or `src/shared/hooks/`
- Check shared utilities before creating duplicates

**Vite Config:**
- Entries: `src/schedule/index.jsx`, `src/song-list/index.jsx`, `src/shared/index.jsx`, `src/gallery/park-city-index.jsx`, `src/gallery/random-index.jsx`
- Output: ES modules to `js/dist/`
- `process.env.NODE_ENV` defined as 'production'

**Common Issues:**
- "React is not defined" → Ensure Vite bundles React (not external), rebuild
- "process is not defined" → Check Vite `define` config, rebuild
- Changes not showing → Verify `npm run watch` running, hard refresh browser
- Data not loading → Check Network tab, verify JSON file or API accessible

**Deployment:** `make publish` runs `npm run build` + Jekyll build + rsync to server

## External JavaScript

- jQuery 3.6.4 (minimal/legacy usage)
- marked.min.js (Markdown parsing in descriptions)

Note: All custom JavaScript replaced by React.

## Build & Dependencies

**Makefile:**
- `make server` - Jekyll dev server
- `make build` - npm build + Jekyll build
- `make publish` - Build + rsync to server

**Dependencies:**
- Ruby/bundler (Jekyll + plugins)
- Node.js/npm (React, ReactDOM, Vite)

**Documentation:** `docs/` contains planning documents. See `REACT_MIGRATION_PLAN.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jimm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
