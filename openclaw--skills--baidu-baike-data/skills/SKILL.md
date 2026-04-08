---
name: baidu-baike-data
description: Use when working with the Baidu Baike Component is a knowledge service tool designed to query authoritative encyclopedia explanations for various nouns. Its core function is given a specific "noun" (object, person, location, concept, event, etc.) provided by the user, it returns a standardized, detailed entry explanation sourced from Baidu Baike.
metadata:
  author: openclaw
---

# Baidu Baike

Query encyclopedia entries from Baidu Baike.

## Two Usage Scenarios

### Scenario 1: Direct Search
Get default matching entry for a keyword.
```bash
python3 scripts/baidu_baike.py --search_type=lemmaTitle --search_key="keyword"
```

### Scenario 2: Homonym Resolution
When term has multiple entries, list them and select by ID.
```bash
# List entries with same name
python3 scripts/baidu_baike.py --search_type=lemmaList --search_key="keyword" --top_k=5

# Get specific entry by ID
python3 scripts/baidu_baike.py --search_type=lemmaId --search_key="entry_id"
```

## API
- LemmaList: List entries with same title
- LemmaContent: Get entry details by title or ID

## Setup
```bash
export BAIDU_API_KEY="your_api_key"
```

## Workflow
1. Extract noun from query
2. For ambiguous terms, call LemmaList first
3. User selects entry from list
4. Call LemmaContent with selected ID
5. Return structured data

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openclaw)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/openclaw)
<!-- tomevault:4.0:skill_md:2026-04-08 -->
