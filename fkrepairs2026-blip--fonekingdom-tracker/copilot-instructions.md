## fonekingdom-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fonekingdom Repair Tracker is a mobile phone repair shop management system for a Philippine-based business. It's a single-page web app using vanilla JavaScript (no frameworks), Firebase Realtime Database, and deployed to GitHub Pages.

**Live URL:** https://fkrepairs2026-blip.github.io/fonekingdom-tracker/

## Development Workflow

No build step required - edit files directly and push to main branch for GitHub Pages auto-deployment.

**Local testing:** Open `index.html` in browser with a live server (required for ES modules). Firebase config points to production database.

## Architecture

### Global State Pattern
All state lives on `window` object for cross-module access:
```javascript
window.currentUser        // Firebase auth user
window.currentUserData    // User data from DB (role, displayName, etc.)
window.allRepairs         // Live repairs array
window.allInventoryItems  // Inventory items
```

### Real-time Data Flow
```
Firebase listener updates → Global array populated → window.currentTabRefresh() called → UI updates
```

Never query Firebase directly from UI code - always use global arrays.

### Tab Refresh Pattern (Critical)
Each tab sets its own refresh callback first, then renders:
```javascript
function buildMyTab(container) {
    window.currentTabRefresh = () => buildMyTab(document.getElementById('myTabId'));
    container.innerHTML = generateHTML();
}
```

### Function Export Pattern (Mandatory)
All functions used in HTML onclick must be exported:
```javascript
function myFunction() { }
window.myFunction = myFunction;  // At end of file
```

### Loading State Pattern
```javascript
try {
    utils.showLoading(true);
    await firebaseOperation();
    utils.showLoading(false);  // MUST hide on success
    if (window.currentTabRefresh) window.currentTabRefresh();
} catch (error) {
    utils.showLoading(false);  // MUST hide on error
    alert('Error: ' + error.message);
}
```

## Role-Based Access Control

Roles: `admin`, `manager`, `cashier`, `technician`. Check `window.currentUserData.role` before sensitive operations.

| Action | Admin | Manager | Cashier | Technician |
|--------|-------|---------|---------|------------|
| Receive Devices | ✅ | ✅ | ✅ | ✅ |
| Accept Repairs | ✅ | ✅ | ❌ | ✅ |
| Set Pricing | ✅ | ✅ | ❌ | ✅ |
| Record Payments | ✅ | ✅ | ✅ | ✅ (needs remittance) |
| Verify Payments | ✅ | ✅ | ❌ | ❌ |
| User Management | ✅ | ❌ | ❌ | ❌ |

## Firebase Operations

**Create:**
```javascript
const newRef = await db.ref('repairs').push({ ...data, createdAt: new Date().toISOString() });
const id = newRef.key;
```

**Update:**
```javascript
await db.ref(`repairs/${repairId}`).update({
    status: newStatus,
    lastUpdated: new Date().toISOString(),
    lastUpdatedBy: window.currentUserData.displayName
});
```

**Soft Delete (preferred):**
```javascript
await db.ref(`repairs/${repairId}`).update({
    deleted: true,
    deletedAt: new Date().toISOString(),
    deletedBy: window.currentUserData.displayName
});
```

## File Structure

- `index.html` - Single-page app shell with inline modals
- `js/app.js` - Initialization orchestrator
- `js/auth.js` - Login/logout, sets `window.currentUser/currentUserData`
- `js/repairs.js` - Firebase operations, business logic (largest file)
- `js/ui.js` - Tab builders, all UI rendering (second largest)
- `js/utils.js` - Utilities: date formatting, loading indicator, image compression
- `js/inventory.js` - Inventory operations
- `js/analytics.js` - Stats and reports
- `css/styles.css` - All styling

## Common Pitfalls

**Missing comma in utils object:**
```javascript
// WRONG - syntax error
daysAgo: function() { }
timeAgo: function() { }

// CORRECT
daysAgo: function() { },
timeAgo: function() { },
```

**Forgetting to export functions:** Results in `Uncaught ReferenceError: functionName is not defined`

**Not hiding loading on error:** UI becomes frozen with spinner overlay

**Querying Firebase instead of using global state:** Bypasses real-time sync

## Localization Notes

- Currency: Philippine Peso (₱)
- Phone format: 09XX or +639XX (11 digits)
- Language: English + Tagalog support
- Timezone: Manila (Asia/Manila) for auto-finalization

## Date Handling

```javascript
// Always store as ISO format
const now = new Date().toISOString();

// Display helpers
utils.formatDateTime(now);  // "Dec 27, 2025, 10:30 AM"
utils.formatDate(now);      // "Dec 27, 2025"
utils.daysAgo(now);         // "2 days ago"
```

## Additional Documentation

Detailed documentation available in `.cursor/` folder:
- `PROJECT.md` - Full architecture, DB schema
- `FEATURES.md` - Feature specifications
- `CODING_STANDARDS.md` - Detailed code standards
- `FILE_STRUCTURE.md` - Code organization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fkrepairs2026-blip)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fkrepairs2026-blip)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
