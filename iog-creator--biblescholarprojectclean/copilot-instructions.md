## biblescholarprojectclean

> This file was automatically restored or completed.


--- 
type: always 
title: Public Domain Bible Processing 
description: Guidelines for processing public domain Bible translations (KJV, ASV, WEB) 
globs: 
  - "load_kjv_bible.py" 
  - "load_asv_bible.py" 
  - "load_public_domain_bibles.py" 
  - "src/etl/public_domain_*.py" 
alwaysApply: false 
--- 
 

# Public Domain Bible Processing Guidelines

## Glob Patterns
- src/etl/etl_english_bible.py
- src/api/*/english_*.py
- tests/unit/test_*kjv*.py
- tests/unit/test_*asv*.py
- tests/integration/test_*kjv*.py
- tests/integration/test_*asv*.py
- load_*_bible.py

## Public Domain Bible Processing Overview

### Supported Public Domain Translations
1. **King James Version (KJV)** - Complete Bible, 31,100 verses
2. **American Standard Version (ASV)** - Complete Bible, 31,103 verses
3. **World English Bible (WEB)** - Optional, modern public domain translation

### Data Sources
Public domain Bible translations can be loaded from the following sources:

1. **Direct from GitHub repositories**:
   - KJV: `https://raw.githubusercontent.com/thiagobodruk/bible/master/json/en_kjv.json`
   - ASV: `https://raw.githubusercontent.com/bibleapi/bibleapi-bibles-json/master/asv.json`

2. **Local files**:
   - If the translations are downloaded locally, they should be stored in the `data/raw` directory.

### Database Storage

All public domain Bible verses should be stored in the `bible.verses` table with:

```sql
CREATE TABLE IF NOT EXISTS bible.verses (
  id SERIAL PRIMARY KEY,
  book_name VARCHAR(50) NOT NULL,
  chapter_num INTEGER NOT NULL,
  verse_num INTEGER NOT NULL,
  verse_text TEXT NOT NULL,
  translation_source VARCHAR(20) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(book_name, chapter_num, verse_num, translation_source)
);
```

### Expected Verse Counts

| Translation | Expected Verse Count |
|-------------|---------------------|
| KJV         | 31,100              |
| ASV         | 31,103              |

## Processing Requirements

### KJV Bible Processing

1. **Format**: The KJV JSON file is structured as an array of books, each containing chapters, and each chapter an array of verses.
2. **Book Names**: Convert book abbreviations or numbers to standard full book names.
3. **Translation Code**: Use 'KJV' as the translation_source.
4. **Error Handling**: Include robust error handling for file format issues.

### ASV Bible Processing

1. **Format**: The ASV JSON file from bibleapi.com has a special structure:
   - Data is contained in `resultset.row` array
   - Each verse record has a `field` array with 5 elements:
     - Reference ID (1001001 = book 1, chapter 1, verse 1)
     - Book number (1-66)
     - Testament number (1 = OT, 2 = NT)
     - Verse number
     - Verse text
2. **Chapter Extraction**: 
   - For single-digit book numbers (e.g., Genesis = 1): Extract chapter from positions 1-3 of the reference ID
   - For double-digit book numbers: Extract chapter from positions 2-4 of the reference ID
3. **Translation Code**: Use 'ASV' as the translation_source.

## Implementation Guidelines

### Loading Process

1. **Download**: Retrieve the Bible JSON data from the appropriate source.
2. **Parse**: Convert the JSON into verse objects with proper book, chapter, verse structure.
3. **Deduplicate**: Check for existing verses before insertion to avoid duplicates.
4. **Batch Processing**: Load verses in batches (1000 verses) to optimize performance.
5. **Verification**: After loading, verify verse counts match the expected totals.

### API Access

All public domain translations should be accessible through the standard verse API:
```
/api/verses?translation=KJV&book=BookName&chapter=N&verse=N
/api/verses?translation=ASV&book=BookName&chapter=N&verse=N
```

### Search Functionality

1. Enable full-text search across all public domain translations.
2. Allow users to specify which translations to include in search results.

## Testing Requirements

1. **Unit Tests**:
   - Verify correct parsing of each translation's specific format
   - Test handling of edge cases (missing fields, special characters)
   
2. **Integration Tests**:
   - Verify complete database loading
   - Confirm verse retrieval through API endpoints
   - Test cross-translation verse comparison

## Copyright and Attribution

Unlike restricted translations, public domain Bibles do not require special copyright notices. However, it's good practice to include attribution:

1. **KJV**: "King James Version (KJV), public domain."
2. **ASV**: "American Standard Version (ASV), 1901, public domain."

## Recommended Loading Scripts

1. **KJV**: Use `load_kjv_bible.py` for downloading and loading KJV verses.
2. **ASV**: Use `direct_asv_download.py` for downloading and processing ASV verses.

## Verification Queries

```sql
-- Verify KJV loading
SELECT COUNT(*) FROM bible.verses WHERE translation_source = 'KJV';
-- Should return 31,100

-- Verify ASV loading
SELECT COUNT(*) FROM bible.verses WHERE translation_source = 'ASV';
-- Should return 31,103

-- Compare John 3:16 across translations
SELECT translation_source, verse_text 
FROM bible.verses 
WHERE book_name = 'John' AND chapter_num = 3 AND verse_num = 16
ORDER BY translation_source;
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iog-creator) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
