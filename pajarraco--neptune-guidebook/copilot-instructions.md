## neptune-guidebook

> This project uses `react-i18next` for multi-language support. Translation files are managed through Google Sheets and generated via the `fetch-sheet-data.js` script.

# Internationalization Rules

## Overview

This project uses `react-i18next` for multi-language support. Translation files are managed through Google Sheets and generated via the `fetch-sheet-data.js` script.

## Critical Rules

### 1. Navigation Links Must Be Language-Agnostic

**NEVER** translate section IDs in the `link` property of features.

**Valid Section IDs:**

- `property-info`
- `check-in-out`
- `local-guide`
- `transport`
- `amenities`
- `emergency`

**Correct Example:**

```json
{
  "features": [
    {
      "text": "Código Wi-Fi",
      "link": "property-info"
    }
  ]
}
```

**Incorrect Example:**

```json
{
  "features": [
    {
      "text": "Código Wi-Fi",
      "link": "información de la propiedad"
    }
  ]
}
```

### 2. Link Properties Are English-Only

The `fetch-sheet-data.js` script automatically:

- Includes `link` properties ONLY in `en.json`
- Excludes `link` properties from `es.json`, `fr.json`, `it.json`

This is controlled in the `transformToGuidebookFormat` function:

```javascript
const featureProperties =
  language === "en" ? ["icon", "text", "link"] : ["icon", "text"];
```

### 3. Manual Translation File Editing

When manually editing translation files:

1. **Never add `link` properties to non-English files** - they will be ignored by the component
2. **Only translate user-facing text** - keep all technical identifiers (section IDs, keys) unchanged
3. **Maintain JSON structure** - ensure all objects and arrays match the English version structure

### 4. Adding New Languages

To add a new language:

1. Create a new sheet tab in Google Sheets with the language code (e.g., `de` for German)
2. Add the language code to the `LANGUAGES` environment variable: `LANGUAGES=en,es,fr,it,de`
3. Add the language to `src/components/LanguageSelector.tsx`:
   ```typescript
   const languages = [
     { code: "en", name: "English", flag: "🇬🇧" },
     { code: "es", name: "Español", flag: "🇪🇸" },
     { code: "fr", name: "Français", flag: "🇫🇷" },
     { code: "it", name: "Italiano", flag: "🇮🇹" },
     { code: "de", name: "Deutsch", flag: "🇩🇪" },
   ];
   ```
4. Run `npm run fetch-data` to generate the new translation file

## Language Selector Component

### Location

- **Component**: `src/components/LanguageSelector.tsx`
- **Styles**: `src/components/LanguageSelector.css`
- **Position**: Fixed in top-right corner of the banner

### Features

- Dropdown menu showing current language flag and code
- Click to expand and select from available languages
- Auto-closes after selection
- Responsive (hides language code on mobile devices)
- Uses Material Symbols Outlined icons

### Styling Rules

- Fixed position: `top: 20px; right: 20px;`
- High z-index (1000) to stay above other content
- White background with shadow for visibility
- Rounded corners (25px for toggle, 12px for dropdown)

## Google Sheets Data Fetching

### Script Location

`scripts/fetch-sheet-data.js`

### Key Functions

**`transformToGuidebookFormat(config, language)`**

- Transforms Google Sheets data into the guidebook JSON structure
- `language` parameter determines whether to include `link` properties
- Only includes links when `language === 'en'`

**`getNumberedItems(config, prefix, properties)`**

- Fetches numbered items from the sheet (e.g., `welcome_feature_1_text`, `welcome_feature_2_text`)
- `properties` array determines which fields to include
- For features, properties vary by language (see Rule #2)

### Running the Script

```bash
npm run fetch-data
```

### Environment Variables Required

- `GOOGLE_SHEET_ID` - The Google Sheets document ID
- `GOOGLE_SERVICE_ACCOUNT_KEY` or `GOOGLE_SERVICE_ACCOUNT_PATH` - Authentication credentials
- `LANGUAGES` - Comma-separated language codes (e.g., "en,es,fr,it")

## Common Issues and Solutions

### Issue: Links don't work in translated versions

**Cause**: Link properties contain translated text instead of section IDs  
**Solution**:

1. Check translation files - remove or fix incorrect `link` values
2. Re-run `npm run fetch-data` to regenerate from Google Sheets
3. Ensure Google Sheets only has section IDs in link columns

### Issue: Language selector icon not displaying

**Cause**: Wrong Material Icons class name  
**Solution**: Use `material-symbols-outlined` not `material-icons`

### Issue: New language not appearing

**Cause**: Missing from LanguageSelector component or LANGUAGES env var  
**Solution**:

1. Add to `languages` array in `LanguageSelector.tsx`
2. Add to `LANGUAGES` environment variable
3. Run `npm run fetch-data`

## Best Practices

1. **Always use the fetch script** - Don't manually create translation files
2. **Test all languages** - After changes, verify links work in all languages
3. **Keep structure consistent** - All translation files should have the same JSON structure
4. **Use section IDs** - Never translate technical identifiers
5. **Validate JSON** - Ensure all translation files are valid JSON

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pajarraco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
