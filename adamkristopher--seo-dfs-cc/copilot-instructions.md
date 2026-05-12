## seo-dfs-cc

> This is a Python toolkit for keyword research using the DataForSEO API. Claude executes research functions on your behalf and saves structured results to JSON files, then creates human-readable summary documents for decision-making.

# DataForSEO Keyword Research Toolkit

## Overview

This is a Python toolkit for keyword research using the DataForSEO API. Claude executes research functions on your behalf and saves structured results to JSON files, then creates human-readable summary documents for decision-making.

## Project Structure

```
/Users/adamcarter/Desktop/SEO/
├── .env                      # API credentials (DO NOT COMMIT)
├── main.py                   # High-level convenience functions
├── config/
│   └── settings.py           # Configuration and defaults
├── core/
│   ├── client.py             # DataForSEO API client (singleton)
│   └── storage.py            # Auto-save results to JSON
├── api/
│   ├── keywords_data.py      # Search volume, CPC, competition
│   ├── labs.py               # Suggestions, difficulty, intent
│   ├── serp.py               # Google/YouTube rankings
│   └── trends.py             # Google Trends data
└── results/
    ├── keywords_data/        # Raw JSON from Keywords Data API
    ├── labs/                 # Raw JSON from Labs API
    ├── serp/                 # Raw JSON from SERP API
    ├── trends/               # Raw JSON from Trends API
    └── summary/              # Human-readable markdown summaries
```

## How to Use (Prompting Claude)

### Basic Research Requests

Just tell Claude what you want to research:

- "Research keywords for [topic]"
- "Get YouTube keyword data for [video idea]"
- "Find keyword suggestions for [seed keyword]"
- "Analyze competitor [domain.com]"
- "What's trending in [category]"

### Specific Function Requests

For more control, request specific functions:

- "Run `get_keyword_suggestions` for 'AI website builders'"
- "Get `get_youtube_serp` for these keywords: [list]"
- "Use `get_bulk_keyword_difficulty` on [keywords]"

### Summary Requests

After running research, ask for summaries:

- "Create a summary of those results in /results/summary"
- "Put the results in a document named [specific-name].md"
- "Summarize the data so I can make decisions"

## Workflow Pattern

### 1. Research Phase
Claude runs API functions and saves raw JSON to `/results/{category}/`

Example files created:
```
results/labs/20251204_092522__keyword_suggestions__AI_website_builders__limit_30.json
results/serp/20251204_093715__youtube_competition__WordPress_is_dead__video_analysis.json
```

### 2. Summary Phase
Claude reads the JSON files and creates markdown summaries in `/results/summary/`

Example summaries:
```
results/summary/wordpress-youtube-video-results.md
results/summary/youtube-competition-serp-analysis.md
```

### 3. Decision Phase
You review the summaries which contain:
- Data tables with volumes, CPC, competition
- Ranked lists of opportunities
- Strategic recommendations
- Links to source data

## Available Functions

### High-Level (main.py)
| Function | Purpose |
|----------|---------|
| `keyword_research(keyword)` | Full keyword analysis |
| `youtube_keyword_research(keyword)` | YouTube-focused research |
| `landing_page_keyword_research(keywords, competitor)` | SEO optimization |
| `full_keyword_analysis(keywords)` | Comprehensive analysis |
| `competitor_analysis(domain)` | Competitor keyword intel |

### Keywords Data API
| Function | Purpose | Limit |
|----------|---------|-------|
| `get_search_volume(keywords)` | Volume, CPC, competition | 700 |
| `get_keywords_for_site(domain)` | Keywords for a domain | - |
| `get_keywords_for_keywords(keywords)` | Keyword expansion | 20 |

### Labs API
| Function | Purpose | Limit |
|----------|---------|-------|
| `get_keyword_overview(keywords)` | Comprehensive metrics | 700 |
| `get_keyword_suggestions(keyword)` | Long-tail variations | 1 seed |
| `get_keyword_ideas(keywords)` | Category expansion | 200 |
| `get_related_keywords(keyword)` | "Searches related to" | 1 seed |
| `get_bulk_keyword_difficulty(keywords)` | Difficulty scores | 1000 |
| `get_historical_search_volume(keywords)` | Monthly trends | 700 |
| `get_search_intent(keywords)` | Intent classification | 1000 |
| `get_domain_keywords(domain)` | Domain rankings | - |
| `get_competitors(keywords)` | Find competitors | - |

### SERP API
| Function | Purpose |
|----------|---------|
| `get_google_serp(keyword)` | Google organic results |
| `get_google_maps_serp(keyword)` | Local results |
| `get_google_news_serp(keyword)` | News results |
| `get_google_images_serp(keyword)` | Image results |

### Trends API
| Function | Purpose |
|----------|---------|
| `get_trends_explore(keywords)` | Full trend data |
| `get_youtube_trends(keywords)` | YouTube trends |
| `get_news_trends(keywords)` | News trends |
| `get_trending_now()` | Currently trending |

## Session History

### Session: 2025-12-04

**Research conducted:**

1. **YouTube Keyword Research - AI WordPress Automation Video**
   - Seeds: "AI replacing web developers", "WordPress AI automation", "tech jobs declining 2025", "WordPress developers future", "freelance web design dying", "AI website builders"
   - Functions used: `get_keyword_suggestions`, `get_keyword_overview`, `get_google_serp`, `get_youtube_trends`
   - Summary: `results/summary/wordpress-youtube-video-results.md`
   - Key finding: "AI website builders" has 40,500 monthly searches

2. **YouTube Competition Analysis**
   - Seeds: "AI is killing web development", "WordPress is dead", "tech jobs 2025", "web developers replaced by AI"
   - Functions used: `get_google_serp` (video analysis)
   - Summary: `results/summary/youtube-competition-serp-analysis.md`
   - Key finding: All keywords saturated with fresh 2025 videos

## Summary Document Format

When Claude creates summaries, they follow this structure:

```markdown
# [Research Topic]
Generated: [timestamp]

## Executive Summary
- Best keyword and why
- Key opportunities

## 1. Search Volume & Metrics
| Keyword | Volume | CPC | Competition | Difficulty |
[data table]

## 2. Keyword Suggestions
[grouped by seed keyword]

## 3. SERP Analysis
[what's currently ranking]

## 4. Recommendations
- Title suggestions
- Tags to use
- Content strategy
```

## Tips for Best Results

1. **Be specific** - "Get keyword suggestions for 'AI website builders'" works better than "research AI stuff"

2. **Request summaries** - Always ask for a summary document after research, named specifically

3. **Batch related keywords** - Give Claude multiple related keywords at once for comparison

4. **Specify the goal** - "for a YouTube video" vs "for a landing page" changes what data is most relevant

5. **Ask for competition analysis** - "Show me what videos are ranking" helps identify opportunities

## Credentials

Located in `.env` (not committed to git):
```
DATAFORSEO_LOGIN=your_email
DATAFORSEO_PASSWORD=your_api_password
```

Get credentials from: https://app.dataforseo.com/api-access

---
> Source: [adamkristopher/seo-dfs-cc](https://github.com/adamkristopher/seo-dfs-cc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
