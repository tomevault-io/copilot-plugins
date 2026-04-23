## domain-volume-control

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Package

**Windows:**
```bash
package.bat
```

**Linux/WSL (if zip is installed):**
```bash
zip -q domain-volume-control-v{VERSION}.xpi manifest.json background.js content.js popup.html popup.js icon.svg
```

**Important:** Update the version number in `manifest.json` before creating a new package.

The build process creates an `.xpi` file (Firefox extension package) by zipping the 6 core files together.

## Architecture

This is a Firefox WebExtension with 3 main components:

### 1. content.js (Content Script)
The heart of volume control. Runs on every page (except excluded domains).

**Key mechanisms:**
- **Property interception**: Overrides `HTMLMediaElement.prototype.volume` setter to intercept all volume changes
- **Web Audio API interception**: Overrides `AudioContext.createGain()` to control Web Audio API volumes (critical for Facebook)
- **Timestamp-based change detection**: Uses 200ms windows to distinguish extension changes from external changes
- **Multi-layered monitoring**:
  - MutationObserver: Real-time DOM watching for new media elements
  - setInterval (250ms): Periodic scan for missed elements
  - Event listeners: 10+ media events (volumechange, playing, loadedmetadata, etc.)
  - timeupdate polling (250ms in sticky mode): Detects and corrects volume drift

**Sticky Volume Mode:**
When enabled (default for facebook.com), aggressively blocks external volume changes and enforces the saved volume. When disabled, only sets initial volume but allows user/site to change it.

**Storage keys:**
- `{domain}`: Volume level (0-100)
- `{domain}_sticky`: Boolean for sticky mode
- `excludedDomains`: Array of excluded domains

### 2. background.js (Background Script)
Minimal service worker that handles:
- Cross-origin iframe domain resolution (when content script can't access `window.top.location`)

### 3. popup.js + popup.html (Browser Action)
User interface for:
- Per-domain volume slider (0-100%)
- Sticky volume checkbox
- Domain exclusion management
- Collapsible excluded domains list

## Critical Implementation Details

### Volume Control Strategy
The extension uses **defensive layers** because sites like Facebook aggressively control their own volume:

1. **Instant detection**: Property setters and `volumechange` event catch changes immediately
2. **Periodic enforcement** (250ms): Corrects any drift during playback
3. **Event-based enforcement**: Re-applies on 10+ media lifecycle events
4. **Multiple API coverage**: Both HTML5 Media Elements and Web Audio API

### Iframe Handling
- Same-origin iframes: Use top-level domain for storage
- Cross-origin iframes: Ask background script for tab's top-level domain
- This ensures videos in embedded iframes use the parent page's volume settings

### Default Exclusions
YouTube is excluded by default to avoid conflicts with its native controls.

## Version History Notes
- v1.0-1.1: Initial release, basic volume control
- v1.2: Added sticky volume feature, iframe support
- v1.3: Improved enforcement with 250ms intervals and additional event listeners to eliminate volume spikes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/auc0le) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
