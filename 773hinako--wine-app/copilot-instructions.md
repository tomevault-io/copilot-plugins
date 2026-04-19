## wine-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**PWA-focused wine tasting record app** with offline support.

This project has been restructured to focus exclusively on **Progressive Web App (PWA)** development. Previous iOS and Standalone versions have been archived for reference.

Core technology: Vanilla JavaScript + IndexedDB for complete offline functionality.

## Project Structure

```
wine/
├── src/                    # Source code
│   ├── js/                # JavaScript modules
│   │   ├── app.js        # Main application logic & UI
│   │   ├── db.js         # IndexedDB wrapper (WineDB class)
│   │   ├── features.js   # Extended features (sort, filter, favorites, stats, dark mode)
│   │   └── auto-backup.js # Auto-backup to LocalStorage
│   ├── css/              # Stylesheets
│   │   └── styles.css    # Main stylesheet with dark mode support
│   └── workers/          # Service Worker
│       └── service-worker.js
│
├── public/               # Static files (deployed as-is)
│   ├── index.html       # Single-page app HTML
│   ├── manifest.json    # PWA manifest
│   └── icons/           # PWA icons
│       ├── icon-192.png
│       └── icon-512.png
│
├── archive/              # Archived versions (iOS, Standalone)
├── docs/                 # Documentation
├── scripts/              # Development scripts
├── netlify.toml         # Netlify deployment config
├── GIT-GUIDE.md         # Git workflow documentation
└── DATA-SAFETY.md       # Data persistence documentation
```

## Common Commands

### Development Server

```bash
# Quick start with provided scripts
./scripts/start-server.bat        # Windows
./scripts/start-server.sh         # Mac/Linux

# Manual start
python -m http.server 8000
```

### Icon Generation

```bash
# PWA icons (192x192, 512x512)
python scripts/generate-icons.py

# Browser-based icon generation
# Open scripts/create-icons.html in browser
```

## Architecture

### Screen Management

The app uses a single-page architecture with three screens managed by JavaScript:
- **Home Screen** (`#home-screen`): Wine list with search functionality
- **Edit Screen** (`#edit-screen`): Add/edit wine form with camera integration
- **Detail Screen** (`#detail-screen`): View full wine details

Screen transitions are handled by `showScreen()` function in [src/js/app.js](src/js/app.js).

### Module Architecture

The app is split into focused modules:

1. **[src/js/db.js](src/js/db.js)** - Data persistence layer
   - `WineDB` class wraps IndexedDB operations
   - Handles CRUD operations and search
   - Export/import functionality

2. **[src/js/features.js](src/js/features.js)** - Extended features
   - Sort/filter utilities (`sortWines`, `filterWines`)
   - Favorite toggle (`toggleFavorite`)
   - Statistics calculation (`calculateStatistics`)
   - Dark mode (`initDarkMode`, `toggleDarkMode`)
   - Tutorial system (`showTutorial`)
   - Swipe gestures (`initSwipeGestures`)

3. **[src/js/auto-backup.js](src/js/auto-backup.js)** - Data safety
   - Auto-backup to LocalStorage every 5 minutes
   - Maintains 3 generations of backups
   - Backup on page unload

4. **[src/js/app.js](src/js/app.js)** - Application controller
   - Screen navigation (`showScreen`)
   - Event handling
   - UI rendering and updates
   - Coordinates other modules

### Data Flow

1. User interactions → Event handlers in [src/js/app.js](src/js/app.js)
2. App logic → IndexedDB operations via [src/js/db.js](src/js/db.js) WineDB class
3. Data stored in IndexedDB + auto-backup to LocalStorage
4. UI updated with new data
5. Service Worker caches app files for offline use

### IndexedDB Schema

**Object Store**: `wines`
- **Key**: `id` (auto-increment)
- **Indexes**: `name`, `region`, `variety`, `date`

**Wine Object Structure**:
```javascript
{
  id: number,              // Auto-generated
  name: string,           // Required
  producer: string,
  region: string,
  variety: string,
  vintage: number,
  date: string,           // ISO date format
  rating: number,         // 0-5
  photo: string,          // Base64 data URL (compressed, max 1200px)
  thumbnail: string,      // Base64 thumbnail (400px, for list view)
  favorite: boolean,      // Favorite flag
  tasting: {              // Structured tasting notes
    wineType: string,     // 'red', 'white', 'rose', 'sparkling'
    appearanceColor: string,
    aromas: string[],
    firstAroma: string,   // Primary aroma (from grape variety)
    secondAroma: string,  // Secondary aroma (from fermentation)
    thirdAroma: string,   // Tertiary aroma/bouquet (from aging)
    oakIntensity: string, // Oak/barrel influence intensity
    sweetness: string,
    acidity: string,
    tannin: string,       // For red wines
    body: string,
    finish: string,
    additionalNotes: string
  },
  notes: string,          // Legacy notes field
  createdAt: string,      // ISO timestamp
  updatedAt: string       // ISO timestamp
}
```

### Photo Handling & Optimization

Photos are automatically compressed and stored as Base64-encoded data URLs in IndexedDB:

**Image Processing Pipeline**:
1. User selects image → `handlePhotoSelect()` in [src/js/app.js](src/js/app.js)
2. File size validation (max 10MB)
3. Auto-compression via `compressImage()`:
   - Full photo: max 1200px, quality 85%
   - Thumbnail: 400px, quality 80% (for list view)
4. Stored in `wine.photo` and `wine.thumbnail` fields

**Benefits**:
- Complete offline functionality
- No external file storage needed
- Automatic optimization reduces storage impact

**Considerations**:
- Large photo collections may consume significant browser storage
- Use thumbnails in list views for performance

### Service Worker Strategy

Uses **Network First** with cache fallback:
1. Tries to fetch from network
2. Updates cache with successful responses
3. Falls back to cache on network failure

This ensures users see the latest version while maintaining offline functionality.

## Key Features & Functions

### Core Features (src/js/app.js)
- **Screen Navigation**: `showScreen(screenName)` - 4 screens: home, edit, detail, stats
- **Wine List**: `loadWineList()` - Loads, filters, sorts, and displays wines with thumbnails
- **Search**: `searchWines(query)` - Real-time search across name, region, variety, notes
- **Form Handling**: `handleFormSubmit()` - Validates and saves wine data
- **Image Processing**: `compressImage(file, maxWidth, quality)` - Canvas-based compression
- **Toast Notifications**: `showToast(message, type)` - User feedback (success/error/info)
- **Loading States**: `showLoading(message)` / `hideLoading()` - Async operation feedback

### Extended Features (src/js/features.js)
- **Favorites**: `toggleFavorite(wineId)` - Star/unstar wines
- **Sorting**: `sortWines(wines, sortBy)` - date, rating, name (asc/desc)
- **Filtering**: `filterWines(wines, filterType)` - all, favorites, wine types
- **Statistics**: `calculateStatistics(wines)` - Rating distribution, type breakdown, top regions/varieties
- **Dark Mode**: `toggleDarkMode()` - Persisted to LocalStorage
- **Tutorial**: `showTutorial()` - First-time user onboarding
- **Swipe Gestures**: `initSwipeGestures()` - Touch interactions for favorites

### Database Operations (src/js/db.js - WineDB class)
- `init()` - Initialize IndexedDB with schema
- `addWine(wine)` - Create new wine record
- `updateWine(id, wine)` - Update existing record
- `deleteWine(id)` - Delete record
- `getWine(id)` - Retrieve single wine
- `getAllWines()` - Retrieve all wines (sorted by date descending)
- `searchWines(query)` - Full-text search across multiple fields
- `exportData()` - Export all data as JSON
- `importData(jsonData)` - Import data from JSON

### Auto-Backup System (src/js/auto-backup.js)
- `AutoBackup.start()` - Initialize auto-backup (5-minute interval)
- `AutoBackup.createBackup()` - Save to LocalStorage
- `AutoBackup.restore(backupIndex)` - Restore from backup
- Maintains 3 generations of backups

## File Paths Reference

### HTML Entry Point
- Main HTML: [public/index.html](public/index.html)
- Script loading order (critical):
  1. `../src/js/db.js` (database layer)
  2. `../src/js/features.js` (extended features)
  3. `../src/js/auto-backup.js` (backup system)
  4. `../src/js/app.js` (application controller - must load last)
- CSS: `../src/css/styles.css`
- Icons: `icons/icon-192.png`, `icons/icon-512.png`
- Manifest: `manifest.json`

### Service Worker
- Location: [src/workers/service-worker.js](src/workers/service-worker.js)
- Registration: `/src/workers/service-worker.js` (absolute path)
- Cache strategy: Network-first with fallback
- Cached resources:
  - `/public/index.html`
  - `/src/css/styles.css`
  - `/src/js/*.js`
  - `/public/manifest.json`

### PWA Manifest
- Location: [public/manifest.json](public/manifest.json)
- App name: "ワイン記録"
- Theme color: `#FFB6D9`
- Display: `standalone`
- Icons: 192x192, 512x512

## Testing the PWA

1. Start local server (port 8000 recommended)
2. Open `http://localhost:8000/public/` in browser
3. Open Chrome DevTools
4. Go to Application tab
5. Check "Service Workers" - should show registered worker
6. Check "Manifest" - should show app info and icons
7. Test offline: Check "Offline" in Network tab, refresh page

## Installing as PWA

### Desktop (Chrome/Edge)
- Click the install icon in the address bar
- Or use browser menu → "Install app"

### Mobile (Chrome/Safari)
- Chrome: Menu → "Add to Home screen"
- Safari: Share → "Add to Home Screen"

## Deployment

### Netlify (Current Deployment)

The app is configured for automatic deployment via [netlify.toml](netlify.toml):

**Configuration**:
- Publish directory: `.` (root)
- Build command: None (static PWA)
- SPA redirect: `/*` → `/public/index.html`

**Cache Headers**:
- Service Worker: `max-age=0, must-revalidate` (always fresh)
- CSS/JS/images: `max-age=31536000, immutable` (1 year)
- Manifest: `max-age=3600` (1 hour)

**Deployment Flow**:
1. Push to GitHub main branch
2. Netlify auto-detects changes
3. Builds and deploys (1-2 minutes)
4. Live at configured domain

**Manual Deploy**:
```bash
# Using Git (auto-deploy)
git add -A
git commit -m "Description"
git push origin main

# Using Netlify CLI
netlify deploy --prod
```

### Data Persistence on Updates

✅ **User data is safe**: Updates to app code do not affect user data
- App code (HTML/CSS/JS) is cached by Service Worker
- User data (IndexedDB) is separate from app code
- Auto-backup to LocalStorage provides additional safety

See [DATA-SAFETY.md](DATA-SAFETY.md) for details.

## Archived Versions

Previous multi-platform versions are archived in [archive/](archive/):
- **archive/ios/** - iOS native app (Capacitor)
- **archive/standalone/** - Single-file HTML version
- **archive/www/** - Capacitor web files

These are kept for reference but are no longer actively maintained.

## Common Issues

### Service Worker Not Updating
- Hard refresh: Ctrl+Shift+R (Windows) or Cmd+Shift+R (Mac)
- Or: DevTools → Application → Service Workers → "Unregister"

### Camera Not Working
- Requires HTTPS or localhost
- Check browser permissions

### Path Issues After Restructure
All file paths have been updated to the new structure:
- HTML uses relative paths (`../src/...`)
- Service Worker uses absolute paths (`/src/...`, `/public/...`)
- Manifest uses relative paths from public/ directory

## Important Development Notes

### Adding New Features

When adding new features that require persistent data:
1. Update wine object structure in `WineDB` schema if needed
2. Add migration logic for existing data
3. Update auto-backup to include new fields
4. Test export/import with new data structure

### Modifying the Database Schema

**IMPORTANT**: IndexedDB schema changes require careful migration:
```javascript
// In db.js, increment version number
const DB_VERSION = 2; // Increment

// Add migration in onupgradeneeded
if (event.oldVersion < 2) {
    // Migration code here
}
```

### Service Worker Updates

After modifying Service Worker:
- Increment `CACHE_VERSION` constant
- Test that old cache is properly cleared
- Verify offline functionality still works

### Script Loading Order

**Critical**: Scripts must load in this exact order:
1. `db.js` - Provides `wineDB` global
2. `features.js` - Uses `wineDB`, provides utility functions
3. `auto-backup.js` - Uses `wineDB`, provides `AutoBackup` global
4. `app.js` - Uses all above modules

Breaking this order will cause "undefined" errors.

### CSS Custom Properties (Dark Mode)

All colors use CSS variables defined in `:root` and overridden in `body.dark-mode`:
```css
:root {
    --background-color: #FFFFFF;
}
body.dark-mode {
    --background-color: #1a1a1a;
}
```
Use variables for all new color styles to maintain dark mode compatibility.

## Key Documentation Files

- [README.md](README.md) - User-facing documentation (Japanese)
- [GIT-GUIDE.md](GIT-GUIDE.md) - Git workflow and commands
- [DATA-SAFETY.md](DATA-SAFETY.md) - Data persistence and backup system
- [PROJECT-STRUCTURE.md](PROJECT-STRUCTURE.md) - Detailed project structure (if exists)
- [DEVELOPMENT-STATUS.md](DEVELOPMENT-STATUS.md) - Feature tracking and status

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/773hinako) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
