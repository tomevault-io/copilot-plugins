## crawler

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

```bash
# Install dependencies and Playwright browsers
npm install

# Run development server
npm run dev

# Build for production
npm run build

# Run production server
npm start

# Run linting
npm run lint
```

## Architecture Overview

This is a Next.js application that implements an advanced web crawler using Playwright. The architecture consists of:

### Backend (API Routes)
- `/app/api/crawler/start/route.ts` - Handles crawl initiation and status checking
- Uses Playwright for browser automation with stealth mode to avoid detection
- Implements human-like behaviors (random delays, mouse movements, typing patterns)
- Stores crawl results as JSON files in `crawl-results/` directory

### Core Crawler Logic (`/lib/crawler/`)
- `browser-manager.ts` - Manages Playwright browser instances with human-like behavior
- `crawler-engine.ts` - Main crawling logic including:
  - URL queue management with rate limiting
  - Link discovery and relationship tracking
  - robots.txt compliance
  - Session management and login handling
  - Screenshot capture

### Frontend Components
- Main UI in `/app/page.tsx` using React hooks for state management
- Crawler configuration form with advanced options
- Real-time progress display (currently simulated, WebSocket support planned)
- Results visualization with tabs for summary, pages, and errors

### Type System
All TypeScript types are defined in `/lib/types/crawler.ts` including:
- `CrawlConfig` - Configuration options
- `CrawlResult` - Complete crawl output structure
- `LinkRelationship` - Tracks how pages are connected via HTML elements

## Key Implementation Details

1. **Human-like Behavior**: The crawler simulates human interactions including:
   - Random delays between actions (200ms-2000ms)
   - Natural mouse movements with curved paths
   - Typing with variable speed and occasional typos
   - Random scrolling patterns

2. **Link Discovery**: Extracts links from:
   - Anchor tags (`<a href>`)
   - Buttons with onclick handlers
   - Tracks exact CSS selectors and element positions

3. **Rate Limiting**: Uses p-queue to limit concurrent requests and respect server resources

4. **Session Management**: Maintains cookies and handles login forms automatically

## Future Improvements

- Implement real WebSocket support for live progress updates
- Add CAPTCHA solving integration
- Implement visual site map generation with D3/React Flow
- Add proxy rotation support
- Create Docker configuration for deployment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rodenp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rodenp)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
