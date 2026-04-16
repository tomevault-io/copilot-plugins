## the-body-electric

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**The Body Electric** is an integrated health & fitness platform that provides a unified dashboard for tracking various health metrics through specialized mini-apps. Part of the Quarterly Systems ecosystem.

**Target URL**: `body.quarterly.systems` (Cloudflare Pages)

## Architecture

### Repository Structure

```
the-body-electric/
├── index.html              # Main landing/coming soon page
├── apps/                   # Mini-apps directory
│   ├── mini-apps/          # Mini-apps registry and docs
│   │   ├── index.json      # App metadata and registry
│   │   └── README.md       # Mini-apps documentation
│   └── weight-tracker/     # Weight tracking mini-app (placeholder)
├── CLAUDE.md               # This file
└── .git/                   # Git repository
```

### Mini-Apps System

Each mini-app is a self-contained application within the `/apps` directory:

- **Registry**: `/apps/mini-apps/index.json` contains metadata for all apps
- **Categories**: body-composition, activity, nutrition, biometrics
- **Data Export**: Each app exposes a standardized data endpoint
- **Independent**: Apps can use different tech stacks as needed

### Current Mini-Apps

**Weight Tracker** (Active)
- EWMA-based weight trend analysis
- 7-day slope calculation
- Calorie delta estimation
- Location: `/apps/weight-tracker/` (to be implemented)

**Future Mini-Apps** (Planned)
- Activity Tracker - Movement, workouts, exercise
- Nutrition Log - Diet, calories, macros
- Biometrics - Heart rate, blood pressure, sleep

## Main Landing Page

**File**: `index.html`

Simple static HTML page showing:
- The Body Electric branding
- List of available mini-apps with status (Active/Soon)
- Links to Quarterly Systems
- No build process required

**Deployment**:
- Cloudflare Pages serves `/index.html` directly
- No build command needed
- Auto-deploys on push to `main` branch

## Development Workflow

### Adding a New Mini-App

1. Create directory: `/apps/[app-name]/`
2. Update `/apps/mini-apps/index.json` with app metadata
3. Build the app with its own tech stack
4. Implement data export endpoint following standardized format
5. Update main landing page to link to new app

### Data Export Standard

Each mini-app should expose a data endpoint returning:

```json
{
  "appId": "app-name",
  "lastUpdated": "2025-10-03T12:00:00Z",
  "summary": {
    "currentValue": 82.5,
    "unit": "kg",
    "trend": "decreasing",
    "changeRate": -0.2
  },
  "recentData": [...],
  "meta": {...}
}
```

## Deployment

### Main Landing Page
```bash
# No build required - static HTML
git add . && git commit -m "..."
git push  # Auto-deploys to body.quarterly.systems
```

### Mini-Apps
Each mini-app may have its own build process:
```bash
cd apps/[app-name]
npm run build  # or app-specific build command
# Deploy to subdirectory or subdomain as needed
```

## Integration with Quarterly Systems

- **Branding**: Cyan/blue gradients, dark theme, "a K5M company" footer
- **Navigation**: Links to https://quarterly.systems
- **Ecosystem**: Part of Quarterly Systems platform alongside VibeCode, Office, Status, Knowledge Base
- **Listing**: Update `quarterly-systems-landing/src/pages/apps.astro` when adding major features

## Tech Stack

### Main Landing Page
- Static HTML/CSS (no framework)
- Gradient styling matching Quarterly Systems branding
- Responsive design

### Mini-Apps (Independent)
Each mini-app chooses its own stack. Example (Weight Tracker):
- React 18 + TypeScript
- Vite build system
- Fireproof (local-first database)
  - Docs: https://use-fireproof.com/docs/welcome/
  - Database API: https://use-fireproof.com/docs/database-api/documents
  - Important: IPLD cannot encode JavaScript `undefined` values - all document fields must have valid values
- Recharts, Tailwind CSS, date-fns

## Development Guidelines

### Testing Weight Tracker
- The weight tracker (`/apps/weight-tracker/index.html`) can be opened directly in the browser
- **No dev server needed** - just open the file:// path in Chrome
- All dependencies are loaded via CDN (React, Recharts, Fireproof, etc.)
- Data persists in browser IndexedDB

### Debugging
- **NEVER add debugging code** to production files (no console.log, debug UI elements, etc.)
- Use browser DevTools console for debugging
- Check browser console for errors instead of adding debug output to code

### Chrome DevTools MCP
This project has the Chrome DevTools MCP server installed, which allows Claude to:
- **Browser automation**: Click, fill forms, navigate pages, take screenshots
- **Performance analysis**: Run performance traces, analyze Core Web Vitals
- **Network inspection**: View and analyze network requests
- **Console access**: Read console messages and errors
- **Emulation**: Test with CPU/network throttling

Use this for testing web pages, debugging issues, and analyzing performance without manual intervention.

## Future Enhancements

- **Dashboard**: Unified view aggregating data from all mini-apps
- **Data Sync**: Cross-app data sharing and insights
- **PWA Support**: Offline capability for mini-apps
- **Authentication**: Optional user accounts for cloud sync
- **Mobile Apps**: Native iOS/Android versions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kmikeym) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
