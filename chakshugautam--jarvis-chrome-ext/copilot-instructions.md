## jarvis-chrome-ext

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JARVIS is a Chrome extension (Manifest V3) that applies intelligent performance optimizations to specific websites using monkey-patching techniques. Currently supports:
- **Hasgeek.com**: Performance optimizations (lazy loading, redirect fixes, image optimization)
- **The Sun (thesun.co.uk)**: Ad blocking and content cleanup

## Development Commands

### Installing the Extension

1. Open Chrome: `chrome://extensions/`
2. Enable "Developer mode" (top-right toggle)
3. Click "Load unpacked"
4. Select directory: `/Users/__chaks__/jarvis-extension`

### Testing

**Test on Hasgeek:**
```
Visit: https://hasgeek.com/rootconf/
Open DevTools → Console
Filter logs by: [JARVIS]
```

**Test on The Sun:**
```
Visit: https://www.thesun.co.uk/
Check console for blocked ads/scripts
```

### Debugging

- Console logs are prefixed with `[JARVIS]`
- Stats are tracked per-session (reset on reload)
- Check extension errors: `chrome://extensions/` → "Errors" button

## Architecture

### Core Components

**manifest.json**
- Defines permissions, content scripts, and background service worker
- Content scripts run at `document_start` for early DOM interception
- Uses declarativeNetRequest for network-level blocking

**background.js** (Service Worker)
- Manages extension lifecycle and settings
- Controls declarativeNetRequest rule enabling/disabling
- Handles message passing between popup and content scripts
- Temporarily unblocks Razorpay when payment buttons clicked

**popup.html + popup.js**
- UI for toggling features and viewing stats
- Site-specific panels (Hasgeek vs The Sun vs unsupported)
- Real-time stats updates via message passing

**Content Scripts**
- `scripts/hasgeek-patches.js`: Hasgeek optimizations
- `scripts/thesun-patches.js`: The Sun ad blocking

**rules.json**
- Declarative network request rules for blocking scripts
- Blocks Razorpay, ad networks, tracking scripts

### Monkey-Patching Techniques

The extension uses aggressive runtime patching to intercept browser behavior:

**1. createElement Override**
```javascript
document.createElement = function(tagName) {
  // Intercept script tags to block Razorpay
}
```

**2. Property Descriptor Patching**
```javascript
Object.defineProperty(HTMLImageElement.prototype, 'src', {
  set: function(value) {
    // Rewrite image URLs to bypass redirects
  }
})
```

**3. MutationObserver**
- Watches for dynamically added scripts/images/ads
- Removes blocked elements before they execute/load

**4. fetch() Override**
```javascript
window.fetch = function(...args) {
  // Block OpenStreetMap tiles until venue visible
}
```

**5. IntersectionObserver**
- Detects when venue section enters viewport
- Triggers map loading only when needed

## Hasgeek Optimizations

### Patch 1: Lazy Load Razorpay (~450KB savings)
- Blocks all Razorpay scripts via declarativeNetRequest rules
- Overrides `createElement` to catch dynamic script injection
- When payment button clicked: unblocks Razorpay, injects checkout script, re-triggers click
- Background worker re-enables blocking after 60 seconds

### Patch 2: Fix Image Redirects (~150ms per image)
- Detects `images.hasgeek.com/embed/file/{hash}` URLs
- Extracts hash and size, rewrites to direct S3 URL: `imgee.s3.amazonaws.com/imgee/{hash}_w{size}_h{size}.jpeg`
- Patches both `img.src` setter and `setAttribute`

### Patch 3: Lazy Load Images
- Adds `loading="lazy"` attribute to images (skips first 3)
- MutationObserver catches dynamically added images

### Patch 4: Lazy Load Maps (~200KB savings)
- Overrides `fetch()` to block `openstreetmap.org` tiles
- IntersectionObserver detects when venue section visible
- Restores original `fetch()` when map needed

## The Sun Optimizations

### Patch 1: Block Ad Scripts
- MutationObserver removes script tags from 20+ ad/tracking domains
- Includes Google Ads, DoubleClick, Facebook, TikTok, analytics

### Patch 2: Remove Ad Containers
- Removes iframes and divs matching ad patterns
- Runs on load + every 3 seconds (catches late-loading ads)
- Checks: ad-related IDs/classes, Google query IDs, large empty containers

### Patch 3: Clean Reading Mode
- Removes recommendation sections ("READ MORE", "MOST READ", etc.)
- Removes sidebars, footers, widget sections
- Removes injected promotional boxes (`.article-boxout`)
- Removes newsletter signup forms (`.inline-module--newsletter`)
- Removes promotional paragraphs (casino/betting affiliate links)
- Preserves main article content and navigation
- Specific selectors: `.digital-personalisation`, `.single__sidebar`, etc.

### Patch 4: Remove Images (Optional)
- Removes all `<img>` and `<picture>` elements from article content
- Scoped to `.article__content` to avoid breaking layout
- MutationObserver watches for dynamically added images

### Patch 5: Remove Videos (Optional)
- Removes `<video>` elements and Brightcove players
- Removes video containers by class name matching
- MutationObserver watches for dynamically added videos

## Hasgeek Additional Patches

### Patch 5: Remove Images (Optional)
- Removes all `<img>` and `<picture>` elements from entire page
- MutationObserver watches for dynamically added images

### Patch 6: Remove Videos (Optional)
- Removes `<video>` elements
- Removes YouTube/Vimeo iframes
- Removes video containers by class name matching
- MutationObserver watches for dynamically added videos

## Settings Storage

Uses `chrome.storage.sync` with these keys:
- `enabled`: Master toggle (global)
- `lazyRazorpay`: Toggle Razorpay lazy loading (Hasgeek)
- `fixRedirects`: Toggle image redirect fixes (Hasgeek)
- `lazyImages`: Toggle lazy image loading (Hasgeek)
- `lazyMaps`: Toggle map lazy loading (Hasgeek)
- `blockAds`: Toggle ad blocking (The Sun)
- `removeImages`: Toggle image removal (both sites)
- `removeVideos`: Toggle video removal (both sites)

Settings changes trigger page reload via `chrome.tabs.reload()`.

## Message Passing

**Content Script → Background:**
```javascript
{ action: 'statsUpdate', stats: {...} }
{ action: 'allowRazorpay' }
```

**Popup → Content Script:**
```javascript
{ action: 'getStats' }  // Response: { stats: {...} }
```

**Background → Popup:**
```javascript
{ action: 'statsUpdate', site: 'thesun', stats: {...} }
```

## Important Constraints

1. **document_start Timing**: Content scripts MUST run at `document_start` to intercept resources before they load
2. **Monkey-Patch Order**: Override native functions before page scripts execute
3. **Razorpay Flow**: Block → User clicks → Unblock → Inject → Re-click → Re-block (60s)
4. **Stats Tracking**: Only counts actions, doesn't verify actual bandwidth savings
5. **Site Detection**: Based on URL hostname matching (`includes('hasgeek.com')`)

## Performance Metrics

**Hasgeek Event Page:**
- Before: 86+ requests, ~2.1MB, ~3.8s load time
- After: 40-50 requests, ~1.4MB, ~2.2s load time
- Improvement: ~42% faster

## Adding Support for New Sites

### Traditional Approach (popup.html + popup.js)

The current production files use hardcoded HTML and site detection. To add a site:

1. Add host permission in `manifest.json`
2. Create `scripts/newsite-patches.js` content script
3. Register in `manifest.json` content_scripts
4. Add hardcoded HTML sections to `popup.html` (50+ lines)
5. Update `popup.js` detectSite() if/else chain
6. Add blocking rules to `rules.json` if needed

### Self-Describing Module Approach (popup-dynamic.html + popup-dynamic.js)

**NEW**: An alternative scalable architecture using self-describing modules:

**To add a new site:**

1. Create `scripts/newsite-patches.js` with SITE_CONFIG:
   ```javascript
   // Site configuration for dynamic UI generation
   window.JARVIS_SITE_CONFIG = {
     id: 'newsite',
     name: 'New Site',
     emoji: '🌐',
     matches: ['newsite.com'],

     features: [
       { id: 'blockAds', label: 'Block Ads', emoji: '🚫', default: true },
       { id: 'removeImages', label: 'Remove Images', emoji: '🖼️', default: false }
     ],

     stats: [
       { id: 'blockedAds', label: 'Ads Blocked' },
       { id: 'savedBytes', label: 'Bandwidth Saved',
         format: (stats) => `~${stats.savedBytes} KB` }
     ]
   };

   // Your site-specific optimization code
   (async function() {
     // ...
   })();
   ```

2. Add entry to `sites.json`:
   ```json
   {
     "id": "newsite",
     "script": "scripts/newsite-patches.js"
   }
   ```

3. Add host permission and content script to `manifest.json`

4. **That's it!** The popup UI generates automatically from SITE_CONFIG.

**Benefits:**
- No popup HTML/JS changes needed
- Each site is completely self-contained
- Easy to maintain 100+ sites
- Sites can be loaded from CDN (during build time)

## Known Limitations

- Image redirect fix assumes `.jpeg` extension (some images are `.png`)
- Razorpay unblock creates 1-2s delay on first payment attempt
- Map lazy loading relies on text matching "Venue" heading
- Stats are per-session only (no persistence)
- The Sun cleanup may remove legitimate content if it has ad-like patterns

## Architecture Comparison

### Traditional (popup.html + popup.js) - Current Production

**Files:**
- `popup.html` - Hardcoded HTML for each site (400+ lines)
- `popup.js` - if/else chains for site detection (150+ lines)

**Pros:**
- Simple and straightforward
- No build step needed
- Easy to debug

**Cons:**
- Adding a site requires 50+ lines of HTML + JS changes
- Settings namespace can collide (two sites can't both use `blockAds`)
- Scales poorly beyond 5 sites

**When to use:** Small projects with 2-5 sites that rarely change

---

### Self-Describing Modules (popup-dynamic.html + popup-dynamic.js) - New Alternative

**Files:**
- `popup-dynamic.html` - Minimal template (200 lines, never changes)
- `popup-dynamic.js` - Generic UI generator (200 lines, never changes)
- `sites.json` - Registry of sites (grows linearly)
- Each site script has `JARVIS_SITE_CONFIG` metadata

**Pros:**
- Adding a site = just edit the site script + add 3 lines to sites.json
- Scales to 100+ sites easily
- Each site is completely independent
- Zero popup code changes for new sites
- Settings naturally namespaced by site ID

**Cons:**
- Slightly more complex initial setup
- Config parsing adds minimal overhead

**When to use:** Projects that will grow to 6+ sites, or want CDN-based site distribution

---

### Recommendation

**Current state:** Traditional approach is fine for Hasgeek + The Sun

**If adding 3+ more sites:** Switch to self-describing modules

**Migration path:**
1. Rename `popup.html` → `popup-legacy.html`
2. Rename `popup-dynamic.html` → `popup.html`
3. Update manifest.json to use new popup.html
4. Existing site scripts already have SITE_CONFIG exported

---
> Source: [ChakshuGautam/jarvis-chrome-ext](https://github.com/ChakshuGautam/jarvis-chrome-ext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
