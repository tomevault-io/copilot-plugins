## bookmarkletincludes

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BookmarkletIncludes is a reusable JavaScript/CSS library for creating bookmarklets. It provides a unified `BMS` (Bookmarklet Suite) namespace with world-class UI components, sophisticated animations, and utilities to simplify bookmarklet creation and reduce file sizes.

### Component System

The project includes a comprehensive components library (`components/` directory) with:
- **Core Components**: Enhanced panels, modals with dark theme by default
- **Data Display**: Advanced tables with sorting, filtering, pagination
- **Interactive**: Sophisticated animation effects (fade, slide, scale, bounce, ripple, etc.)
- **Templates**: Pre-built combinations like Data Viewer

All components:
- Use dark theme by default for reduced eye strain
- Include smooth animations with `prefers-reduced-motion` support
- Work independently or can be composed together
- Are loaded dynamically from GitHub raw URLs

## Architecture

The project follows a modular architecture with zero external dependencies:

- **Core Library**: `JavaScript/bookmarklet-suite.js` - Provides the BMS namespace with UI, DOM, and Utils modules
- **CSS Styles**: `CSS/bookmarklet-suite.css` - All styles use `.bms-` prefix for isolation
- **Refactored Bookmarklets**: Individual bookmarklets in `JavaScript/*-refactored.js` that depend on BMS
- **Loaders**: Minimal bookmarklet code in `Bookmarklets/loaders/*.js` that inject BMS and the bookmarklet script

## Key Development Patterns

### BMS Namespace Structure
```javascript
window.BMS = {
  config: {},           // Configuration options
  init(options),       // Initialize BMS and inject CSS
  UI: {               // UI Components
    createPanel(),    // Draggable, resizable panels with tabs
    createModal(),    // Modal dialogs
    showSpinner(),    // Loading indicators
    updateStatus()    // Status messages (auto-dismiss)
  },
  DOM: {              // DOM utilities
    select(),         // querySelector wrapper
    selectAll(),      // querySelectorAll wrapper
    setHTML()        // Safe innerHTML (strips scripts)
  },
  Utils: {            // Utility functions
    generateId(),     // Unique ID generation
    copyToClipboard(),// Clipboard operations
    throttle(),       // Rate limiting
    debounce(),       // Delayed execution
    parseEngagementCount() // Parse K/M/B notation
  }
}
```

### Creating a New Bookmarklet

1. Create `JavaScript/MyBookmarklet-refactored.js`:
```javascript
BMS.runMyBookmarklet = async function() {
  if (!window.BMS) { alert("BMS not loaded"); return; }

  // Load required components
  try {
    await BMS.loadComponents([
      'core/panel/panel',
      'data-display/table/table',
      'interactive/animation-effects/animations'
    ]);
  } catch (error) {
    console.error('Failed to load components:', error);
  }

  // Use enhanced components if available
  if (BMS.UI.Components && BMS.UI.Components.Panel) {
    const panel = BMS.UI.Components.Panel.create({
      title: '🚀 My Bookmarklet',
      content: 'Your content here',
      position: { top: 50, left: 50 },
      size: { width: 600, height: 400 },
      animation: true,
      theme: 'dark'
    });

    if (BMS.UI.Animations) {
      BMS.UI.Animations.scale(panel, { duration: 400 });
    }
  } else {
    // Fallback to basic panel
    BMS.UI.createPanel({
      title: 'My Bookmarklet',
      content: 'Your content here'
    });
  }
};
BMS.runMyBookmarklet();
```

2. Create `Bookmarklets/loaders/MyBookmarklet-loader.js` (minified, single-line):
```javascript
javascript:(function(){const SUITE_URL='https://raw.githubusercontent.com/SOELexicon/BookmarkletIncludes/refs/heads/main/JavaScript/bookmarklet-suite.js';const SCRIPT_URL='https://raw.githubusercontent.com/SOELexicon/BookmarkletIncludes/refs/heads/main/JavaScript/MyBookmarklet-refactored.js';if(window.BMS){const script=document.createElement('script');script.src=SCRIPT_URL;document.body.appendChild(script);return;}const suiteScript=document.createElement('script');suiteScript.src=SUITE_URL;suiteScript.onload=()=>{BMS.init();const script=document.createElement('script');script.src=SCRIPT_URL;document.body.appendChild(script);};document.body.appendChild(suiteScript);})();
```

## Testing

No build process required. Test bookmarklets by:

1. Open test HTML files directly in a browser:
   - `test.html` - General BMS component testing
   - `HtmlAnalyser-test.html` - Framework detection testing
   - `JsonExtract-test.html` - JSON-LD extraction testing
   - `TwitterMediaExtractor-test.html` - Media extraction testing

2. Run comprehensive tests via `JavaScript/test-bookmarklet.js` which tests all BMS features

### Content Security Policy (CSP) Limitations

**CRITICAL**: Bookmarklets **CANNOT** run on websites with strict Content Security Policy headers.

**GitHub.com is IMPOSSIBLE for bookmarklets** because it blocks:
- External scripts (`script-src github.githubassets.com` only)
- Inline scripts (no `unsafe-inline`)
- Eval and dynamic imports (no `unsafe-eval`)
- Blob URLs (blocked by script-src)

**Other sites where bookmarklets will NOT work:**
- Banking websites (Chase, Bank of America, etc.)
- Government websites (.gov domains)
- Enterprise applications with security policies
- Any site with `script-src` without `unsafe-inline`

**Sites where bookmarklets WILL work:**
- Twitter/X.com ✓
- Wikipedia.org ✓
- News websites (CNN, BBC, etc.) ✓
- Reddit.com ✓
- YouTube.com ✓
- Most blog platforms ✓
- E-commerce sites ✓
- Personal websites ✓

**DO NOT test bookmarklets on GitHub.com** - use the local HTML test files instead, or test on Twitter/Wikipedia.

**Technical details**: The loaders fetch JavaScript code and inject it as inline scripts using `script.textContent`. This bypasses most CSP restrictions, but sites like GitHub explicitly block inline scripts. This is a fundamental browser security feature, not a bug.

## Important Conventions

- **CSS Classes**: Always prefix with `bms-` to avoid conflicts
- **Private Functions**: Prefix with underscore (e.g., `_makeDraggable`)
- **GitHub URLs**: All production loaders and components use `https://raw.githubusercontent.com/SOELexicon/BookmarkletIncludes/refs/heads/main/` as the base URL
- **Safe HTML**: Use `BMS.DOM.setHTML()` instead of direct innerHTML to prevent script injection
- **Dark Theme**: All components use dark theme by default
- **Async Loading**: Use `await BMS.loadComponents([...])` to load components dynamically
- **Graceful Degradation**: Always provide fallback to basic UI if components fail to load

## Component Loading

Components are loaded dynamically from GitHub:

```javascript
// Load single component
await BMS.loadComponent('core/panel/panel');

// Load multiple components
await BMS.loadComponents([
  'core/panel/panel',
  'data-display/table/table',
  'interactive/animation-effects/animations'
]);
```

Components are cached after first load to improve performance.

## Common Commands

Since this is a plain JavaScript/CSS project with no build system:

- **Test locally**: Open any `*-test.html` file in a browser
- **Deploy**: Push changes to GitHub main branch (bookmarklets load from raw GitHub URLs)
- **Debug**: Check browser console for errors, all BMS operations log there

## File Size Optimization

The refactored approach significantly reduces bookmarklet sizes:
- Original HtmlAnalyser.js: 76KB → Refactored: ~8KB + shared BMS library
- Original twitter-media-analyzer.js: 118KB → Refactored: ~12KB + shared BMS library

The shared BMS library (bookmarklet-suite.js) is cached after first load, making subsequent bookmarklet executions faster.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SOELexicon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SOELexicon)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
