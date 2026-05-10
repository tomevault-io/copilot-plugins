## swift-poe-search-mcp

> - No comments unless code doesn't explain itself

# Swift Search MCP - AI Guidelines

## Coding Standards

- No comments unless code doesn't explain itself
- Use Swift 6.2
- Use all search/research tools for latest online information
- Use all Swift-related skills available
- ASK QUESTIONS ALWAYS - interview in detail about technical implementation, concerns, tradeoffs

## Parameter Convention

- `showThinking: Bool?` - Shows `<think>...</think>` tags when `true`, strips when `false` (default)

## Poe Model Parameters Reference

Use these parameters when generating tool calls.

### Sonar & Sonar-Pro Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| search_context_size | enum | low | low, medium, high |
| search_mode | enum | default | default, academic, sec |
| search_domain_filter | string | "" | Comma-separated domains. Exclude with '-'. Max 20 |
| search_language_filter | string | "" | ISO 639-1 codes (2 letters). Max 10 |
| search_recency_filter | enum | none | none, day, week, month, year |
| search_after_date | string | "" | Exact publication date |
| search_before_date | string | "" | Exact publication date |
| country | string | "" | ISO 3166-1 alpha-2 |
| region | string | "" | State/Province name |
| city | string | "" | City name |
| latitude | string | "" | Requires longitude + country |
| longitude | string | "" | Requires latitude + country |
| return_images | bool | false | Include images |
| return_videos | bool | false | Include videos |
| image_domain_filter | string | "" | Domains for images. Max 10 |
| image_format_filter | string | "" | gif, jpg, png, webp |

### Deep Research Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| reasoning_effort | enum | low | low, medium, high |
| search_mode | enum | default | Only "default" supported |
| search_domain_filter | string | "" | Comma-separated domains |
| search_after_date_filter | string | "" | Date filter |
| search_before_date_filter | string | "" | Date filter |
| last_updated_after_filter | string | "" | Update date filter |
| last_updated_before_filter | string | "" | Update date filter |

### Exa Search Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| operation | enum | search | search, similar, contents, code |
| search_type | enum | auto | auto, neural, deep, fast |
| num_results | int | 10 | Number of results (1-100) |
| category | enum | "" | company, research paper, news, pdf, github, tweet, etc. |
| show_content | bool | false | Display full page content |
| include_domains | string | "" | Comma-separated domains to include |
| exclude_domains | string | "" | Comma-separated domains to exclude |
| include_text | string | "" | Text that must appear (up to 5 words) |
| exclude_text | string | "" | Text that must NOT appear |
| start_crawl_date | string | "" | ISO 8601 date |
| end_crawl_date | string | "" | ISO 8601 date |
| start_published_date | string | "" | ISO 8601 date |
| end_published_date | string | "" | ISO 8601 date |
| return_text | bool | true | Fetch page text content |
| text_max_chars | string | "" | Limit text length |
| include_html_tags | bool | false | Preserve HTML structure |
| return_highlights | bool | false | AI-selected key snippets |
| highlights_sentences | int | 3 | Sentences per highlight (1-10) |
| highlights_per_url | int | 3 | Highlights per result (1-10) |
| highlights_query | string | "" | Guide highlight selection |
| return_summary | bool | false | AI-generated summaries |
| summary_query | string | "" | Guide summary generation |
| livecrawl | enum | fallback | fallback, never, always, preferred |
| subpages | int | 0 | Linked subpages to fetch (0-10) |
| subpage_target | string | "" | Keyword for subpages |
| code_tokens | enum | dynamic | dynamic, 5000, 10000, 20000 |

### Exa Answer Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| text | bool | false | Show text snippets under citations |

### Exa Research Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| model | enum | exa-research | exa-research, exa-research-pro, exa-research-fast |

### Linkup Search Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| depth | enum | standard | standard (fast) or deep (comprehensive) |

---
> Source: [mehmetbaykar/swift-poe-search-mcp](https://github.com/mehmetbaykar/swift-poe-search-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
