## king-brewing

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains two web projects from the 2009-2013 era:

1. **King Brewing Company** - Static brewery website (root level and `/kingbrewing/`)
2. **DealingTree** - Location-based deals application (`/kb3/`) featuring HTML5 technologies

The primary active project is DealingTree, which showcases cutting-edge web technologies for its time, including WebSockets, geolocation, offline support, and progressive enhancement.

## Development Commands

### Running the WebSocket Server

The DealingTree application uses a PHP WebSocket server for real-time counter updates:

```bash
# Start the WebSocket server (listens on port 33850)
php kb3/count-server.php

# Or run in background
nohup php kb3/count-server.php &
```

**Note:** On IIS with ASP.NET, the server auto-starts via `kb3/Global.asax` when the application starts.

### Serving the Application

This is a legacy PHP application designed for Apache/IIS:

```bash
# Using PHP's built-in server (for development)
php -S localhost:8000

# Then access: http://localhost:8000/kb3/
```

The application expects Apache with `.htaccess` support for HTTPS redirection.

### Testing WebSocket Connectivity

```bash
# Check if WebSocket server is listening on port 33850
netstat -an | grep 33850
```

## High-Level Architecture

### Progressive Enhancement Pattern

DealingTree uses **feature detection** (not browser detection) via Modernizr to conditionally load features:

- `kb3/initialize-home-page.js` is the entry point that loads features based on browser capabilities
- Uses `Modernizr.load()` to conditionally include:
  - Geolocation polyfills
  - JSON/localStorage polyfills
  - Drag-and-drop UI (if supported)
  - WebSocket real-time counter (if supported)
  - Web Workers for background processing (if supported)

### Key Architectural Components

**Client-Side Storage:**
- Deal inventory stored in `localStorage`
- Managed by `kb3/js/inventory-manager.js` and `kb3/js/deal-storage.js`

**Location-Based Features:**
- Geolocation API for user location
- Haversine formula (`haversine-miles.js`) for distance calculations
- Market selection based on proximity

**Real-Time Updates:**
- WebSocket connection to `count-server.php` on port 33850
- Displays live membership counter
- Custom PHP WebSocket v13 implementation

**Offline Support:**
- HTML5 App Cache manifest: `kb3/dealingtree.appcache`
- Caches core files for offline browsing
- Fallback to `offline.html` when network unavailable

**Geographic Structure:**
- Each city has its own directory: `/kb3/{City}/`
- Deal pages follow pattern: `/kb3/{City}/sku##/index.html`
- Markets: Atlanta, Chicago, Toronto, New York, Los Angeles, London, Paris, Tokyo, Stockholm, Washington, Ottawa

## Critical Files

### Entry Points
- `kb3/index.html` - Main application page
- `kb3/index.php` - Server-side form/cookie handling, market selection
- `kb3/initialize-home-page.js` - Feature detection and conditional module loading

### Server Components
- `kb3/count-server.php` - WebSocket server for real-time counter (port 33850)
- `kb3/Global.asax` - ASP.NET application startup (auto-launches WebSocket server)
- `.htaccess` - Apache rewrite rules for HTTPS enforcement

### Core JavaScript Modules
- `kb3/js/inventory-manager.js` - Purchase and inventory tracking
- `kb3/js/deal-storage.js` - localStorage persistence layer
- `kb3/market-selection.js` - City/market selection logic
- `kb3/market-geolocation.js` - Geolocation integration
- `kb3/haversine-miles.js` - Distance calculations between coordinates
- `kb3/websockets-counter.js` - WebSocket client for real-time counter
- `kb3/deal-dragndrop.js` - Drag-and-drop UI for deal management

### Polyfills and Compatibility
- `kb3/js-webshim/` - HTML5 polyfills library (WebShims Polyfiller)
- `kb3/js/modernizr.js` - Feature detection
- `kb3/js/jquery.js` - DOM manipulation (93KB minified)
- `kb3/js/xmlhttprequest.js` - AJAX fallback for older browsers

### Styling
- `kb3/css/stylesheet.css` - Main styles
- `kb3/css/general.css`, `kb3/css/borders-etc.css`, `kb3/css/3d-effects.css` - Modular CSS
- `kb3/css/media-queries.css` - Responsive design
- `kb3/css/ie.css`, `kb3/css/ie6.css`, `kb3/css/ie7.css` - Legacy IE support

### Configuration
- `kb3/dealingtree.appcache` - HTML5 cache manifest for offline support
- `kb3/cloud-data.txt` - Application data file

## Technology Stack Notes

### No Modern Build System
- No npm, webpack, or bundler
- Scripts loaded via `<script>` tags and `Modernizr.load()`
- CSS concatenation done manually
- Assets served directly without transpilation

### Legacy Browser Support
- IE6, IE7, IE8 compatibility via polyfills
- Conditional CSS files for each IE version
- WebShims Polyfiller provides HTML5 API support

### Server-Side Technologies
- PHP for server logic and WebSocket implementation
- ASP.NET for application lifecycle (IIS hosting)
- CGI scripts in `/cgi/` for form handling

### HTML5 APIs Used
- Geolocation API
- Web Storage (localStorage)
- Web Workers
- WebSockets
- Drag and Drop API
- App Cache (deprecated in modern browsers)

### Feature Detection Strategy
Always use `Modernizr.test` to check for features before use. The codebase follows the pattern:
```javascript
{
  test: Modernizr.feature,
  yep: 'feature-implementation.js',
  nope: 'feature-polyfill.js'
}
```

## Working with This Codebase

### When Adding Features
- Follow the progressive enhancement pattern
- Add feature detection before using new APIs
- Update `initialize-home-page.js` to conditionally load your module
- Test in both modern browsers and with features disabled

### When Modifying the WebSocket Server
- Remember clients expect WebSocket v13 protocol
- Server checks for existing instance before starting (port 33850)
- Broadcasts to all connected clients simultaneously
- Currently ignores client messages (one-way communication)

### When Working with Geographic Data
- Deal pages are statically generated in city directories
- Each market has its own coordinate data for distance calculations
- Use Haversine formula for lat/long distance calculations

### When Updating Offline Support
- Edit `kb3/dealingtree.appcache` to add/remove cached files
- Increment the version number in the manifest comment
- App Cache requires proper MIME type configuration on server

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheiDude)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TheiDude)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
