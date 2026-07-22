## amazon-variation-price-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Chrome extension (Manifest V3) that scans Amazon product page variations (colors, sizes, etc.) to find the cheapest option and tracks price history over time. The extension uses vanilla JavaScript with no frameworks or build tools.

## Architecture

### Core Components

**Content Script (content.js)**
- Runs on all amazon.com pages (run_at: document_idle)
- Extracts product data (ASIN, title, price, variation name) from the DOM
- Handles variation scanning by programmatically clicking through all color/size options
- Uses multiple CSS selector fallbacks for price extraction due to Amazon's inconsistent DOM structure
- Communicates with popup via chrome.runtime message passing

**Popup UI (popup.js + popup.html)**
- Main user interface shown when clicking the extension icon
- Manages scan operations and displays results
- Handles data persistence using Chrome Storage API
- Stores price history indexed by ASIN (Amazon product ID)
- Maintains up to 50 historical scans per product

**Manifest (manifest.json)**
- Manifest V3 configuration
- Permissions: storage (for price history), activeTab (for current page interaction)
- Host permissions: amazon.com only (not international sites)

### Data Flow

1. User clicks "Scan All Variations" button in popup
2. Popup sends `scanAllVariations` message to content script
3. Content script finds all variation swatches/dropdowns on page
4. For each variation:
   - Clicks the variation element (or selects dropdown option)
   - Waits 1000ms for price to update
   - Extracts price and variation name
   - Sends progress update to popup
5. Results returned to popup, sorted by price
6. Popup saves results to chrome.storage.local indexed by ASIN
7. UI displays variations with cheapest highlighted

### Storage Structure

```javascript
{
  priceHistory: {
    [ASIN]: {
      productTitle: string,
      scans: [
        {
          timestamp: ISO string,
          results: [
            {
              productTitle, currentPrice, currentVariation,
              variationName, timestamp, url, asin
            }
          ]
        }
      ]
    }
  }
}
```

## Development

### Testing the Extension

1. Make code changes
2. Go to `chrome://extensions/`
3. Click refresh icon on the extension card
4. Navigate to an Amazon product page (e.g., https://www.amazon.com/dp/B0CXCTSMJH)
5. Click extension icon to test

### Debugging

- **Popup**: Right-click extension icon → "Inspect popup" (opens DevTools for popup.html)
- **Content script**: F12 on Amazon page → Console tab (content.js logs appear here)
- **Background errors**: chrome://extensions/ → "Errors" button on extension card

### Key Implementation Details

**Price Extraction**: Amazon uses multiple price selectors depending on product type. The extension tries selectors in order:
- `.a-price .a-offscreen` (most common)
- `#corePrice_feature_div .a-price .a-offscreen`
- `#price_inside_buybox`
- `.a-price-whole`
- `#priceblock_ourprice`
- `#priceblock_dealprice`

**Variation Detection**: Supports both swatch-style and dropdown-style variations:
- Swatches: `#variation_color_name li`, `#variation_size_name li`
- Dropdowns: `#variation_color_name select option`, `#variation_size_name select option`
- Filters out unselectable variations via `.unselectable` class check

**Async Message Handling**: Content script uses `return true` in message listener to keep channel open for async responses.

**Price Parsing**: Prices are extracted as strings (e.g., "$19.99"), then cleaned with `replace(/[^0-9.]/g, '')` for numerical comparison.

## Limitations & Known Issues

- Only supports amazon.com (not .ca, .uk, .de, etc.)
- 1-second wait between variation clicks may need adjustment for slow connections
- Amazon's DOM structure changes frequently; selector fallbacks may need updates
- Some product page layouts may not be detected (returns "No variations found")
- Extension requires page to be fully loaded before scanning

## Common Tasks

When modifying selectors due to Amazon DOM changes:
1. Use browser DevTools to inspect current Amazon page structure
2. Add new selectors to the fallback arrays in content.js
3. Test on multiple product types (clothes, electronics, books with variations)

When adding international Amazon support:
1. Update `host_permissions` in manifest.json
2. Add domain checks in popup.js and content.js
3. Account for currency symbol variations in price parsing

---
> Source: [hackn3y/amazon-variation-price-tracker](https://github.com/hackn3y/amazon-variation-price-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
