## mindful-saver

> Chrome extension MVP for combating impulsive online spending using behavioral psychology interventions. Built with Vite, Material Design 3, and modular architecture (platform-agnostic core + Chrome platform code) designed for future Android translation.

# Mindful Saver - Session Start Guide

## Project Overview
Chrome extension MVP for combating impulsive online spending using behavioral psychology interventions. Built with Vite, Material Design 3, and modular architecture (platform-agnostic core + Chrome platform code) designed for future Android translation.

---

## Current Status (v0.1.1)

### ✅ What Works
- Extension loads and runs without errors
- Detects "Add to Cart" buttons on Amazon (both main page and secondary)
- Shows intervention modal with Material Design 3 components
- Modal interactions work (categorization, wishlist suggestion, etc.)
- Purchase completion works for **secondary buttons** (frequently bought together section)
- No infinite loop issues (MutationObserver properly debounced)

### ❌ Critical Blocker
**Main Amazon product page "Add to Cart" button doesn't complete purchase after intervention**
- Modal appears ✅
- User clicks "proceed anyway" ✅
- Modal closes ✅
- Item is NOT added to cart ❌

**Element Structure:**
```html
<input type="submit" id="add-to-cart-button" name="submit.add-to-cart">
```
Inside a `<form>` element.

**Current Approach (Failing):**
```javascript
const form = pendingPurchaseElement.closest('form');
if (form) {
  form.submit();  // ← This doesn't work
}
```

---

## Immediate Task

**Fix purchase completion for Amazon main product page button.**

### Debugging Steps
1. **Add console logging** to verify code execution:
   - Is `completePurchase()` being called?
   - Is the form element found?
   - Are there any JS errors in console?

2. **Try alternative submission methods:**
   - `HTMLFormElement.prototype.submit.call(form)` (bypass potential overrides)
   - Dispatch submit event: `form.dispatchEvent(new Event('submit', {cancelable: true, bubbles: true}))`
   - Delay submission: `setTimeout(() => form.submit(), 100)`
   - Programmatic button click instead: `pendingPurchaseElement.click()`

3. **Investigate Amazon's form:**
   - Check for hidden fields
   - Look for custom submit handlers
   - Test if `formaction` attribute needs special handling

4. **Alternative approach:**
   - Don't prevent default, just show modal and return early on second click
   - Use a flag system instead of `preventDefault()`

---

## Key Files to Read First

### **MUST READ** (in order):
1. **`docs/handoffs/Handoff_10-18-2025_22-59-48_EST.md`** - Latest session handoff with detailed debugging notes
2. **`docs/Master_Handoff.md`** - Quick summary of all sessions
3. **`extension/chrome/content-script.js`** - Contains the failing purchase completion logic
   - Lines 92-94: Global variables for pending purchase
   - Lines 169-188: `completePurchase()` function (THE BUG IS HERE)
   - Lines 216-253: Click event handler with form detection

### Helpful Context:
4. **`docs/Master_Index.md`** - Complete file catalog
5. **`plan.md`** - Original project specification
6. **`README.md`** - Build and testing instructions

---

## How to Test

### Setup
```bash
npm run build          # Build the extension
# Load dist/ folder in chrome://extensions/
```

### Test Scenario
1. Go to any Amazon product page (e.g., search "blink camera")
2. Click the main "Add to Cart" button (right sidebar)
3. Intervention modal should appear
4. Click "I want to buy it now anyway" (or go through full flow)
5. **Expected:** Item added to cart
6. **Actual:** Modal closes, nothing happens

### Secondary Test (This Works)
1. Scroll down to "Frequently bought together" section
2. Click any "Add to cart" button there
3. Same modal flow
4. **Result:** Item IS added to cart ✅

---

## Project Structure

```
extension/
  core/               # Platform-agnostic logic (future Android reuse)
    models.js         # Data models
    intervention-logic.js  # Decision engine
  chrome/             # Chrome-specific code
    background.js     # Service worker
    content-script.js # ← BUG IS HERE (purchase completion)
  ui/                 # Modal and popup
```

---

## Build System
- **Vite** with `@crxjs/vite-plugin`
- Material Web Components bundled locally (not CDN)
- `npm run build` → outputs to `dist/`
- Always reload extension in Chrome after rebuild

---

## Testing Expectations

**User is the tester.** Build features, let user test, wait for confirmation before moving forward. Don't test yourself unless debugging requires it.

---

## End Session Protocol

When user says "end session" or similar:
1. Create detailed handoff file: `docs/handoffs/Handoff_MM-DD-YYYY_HH-MM-SS_EST.md`
2. Update `docs/Master_Handoff.md` with 1-2 sentence summary
3. Update `docs/Master_Index.md` if new files created
4. Commit changes with descriptive message
5. Create git tag (semantic versioning: v0.1.x for patches, v0.2.x for features)
6. Push to GitHub
7. Create GitHub release with notes

---

## Important Reminders

- **Keep files under 500 lines** (current longest: content-script.js at 276 lines)
- **Modular architecture** - no Chrome-specific code in `core/`
- **User confirms functionality** - don't assume it works without testing
- **References folder gitignored** - screenshots go in `references/`
- **Commit often** - after each working feature

---

## GitHub
- **Repo:** https://github.com/haithemobeidi/mindful-saver (private)
- **Latest:** v0.1.1
- **Branch:** master

---

## Quick Start Checklist

When starting a new session:
- [ ] Read latest handoff file
- [ ] Review the critical blocker section above
- [ ] Read `extension/chrome/content-script.js` lines 169-188
- [ ] Add debugging console logs
- [ ] Try alternative form submission methods
- [ ] Build and have user test
- [ ] Iterate until main button works

**Goal:** Get Amazon main "Add to Cart" button working after intervention modal.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haithemobeidi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/haithemobeidi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
