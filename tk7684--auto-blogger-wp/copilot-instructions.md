## auto-blogger-wp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Auto-Blogging WordPress is an automated content generation system that:
- Fetches trending topics from multiple sources (Twitter, NewsAPI, local JSON)
- Generates SEO-optimized articles using Google Gemini AI
- Creates featured images using Gemini 3 Pro Image / DALL-E 3 / Hugging Face FLUX
- Publishes to WordPress with Yoast SEO integration
- Supports multilingual content (English/Thai)

## Common Commands

### Run Daily Content Generation
```bash
python main.py
# or explicitly:
python main.py daily
```

### Run with Manual Topic
```bash
python main.py "your topic here"
```

### Run Maintenance/Auditor
```bash
python main.py maintenance 10  # audits 10 posts
```

### Run Tests
```bash
pytest
pytest tests/test_integration.py  # specific test file
pytest -v  # verbose output
```

### Test Image Generation
```bash
python test_image_gen.py
```

## Environment Variables

Required variables in `.env`:
```
WP_URL=https://your-site.com
WP_USER=your@email.com
WP_APP_PASSWORD=your-app-password
GEMINI_API_KEY=your-gemini-key
SITE_URL=https://your-site.com
SITE_NAME=YourSiteName
```

Optional variables:
```
GEMINI_SERVICE_ACCOUNT_KEY_FILE=path/to/service-account.json  # For Vertex AI with region rotation
HUGGINGFACE_API_KEY=your-hf-key  # Fallback image generation
OPENAI_API_KEY=your-openai-key  # DALL-E 3 fallback
TWITTER_BEARER_TOKEN=your-twitter-token  # Trending topics
NEWSAPI_KEY=your-newsapi-key  # Trending topics
SEO_ENABLED=true
IMAGE_GENERATION_ENABLED=true
YOAST_INTEGRATION_ENABLED=true
SCHEMA_MARKUP_ENABLED=true
```

## Architecture

### Entry Points

- `main.py` - Root entry point, delegates to `src/main.py`
- `src/main.py` - Core orchestration: `initialize_system()`, `run_content_generation()`, `run_maintenance()`

### Core Components

**Clients (`src/clients/`)**
- `GeminiClient` - Centralized Gemini API client with region rotation for Vertex AI
- `WordPressClient` - WordPress REST API client

**Image Generation (`src/image_generator.py`)**
- `ImageGenerator` - Multi-service image generation with fallback chain
- `WordPressMediaUploader` - Upload images to WordPress media library
- `GeminiRateLimiter` - RPM/RPD tracking with midnight Pacific timezone reset
- Priority: Gemini 3 Pro Image → DALL-E 3 → Hugging Face FLUX.1-dev

**Content Pipeline**
1. `trend_sources.py` - Fetch trending topics (Twitter, NewsAPI, JSON fallback)
2. `seo_system.py` - SEO prompt builder, schema markup generator
3. `research_agent.py` - Research augmentation for content
4. `auditor.py` - Content maintenance: updates old posts, fixes links, adds missing images
5. `yoast_seo.py` - Yoast SEO meta field integration

### Content Generation Flow

```
initialize_system()
  │
  ├─> Clients: WordPress, Gemini
  ├─> Components: ImageGenerator, MediaUploader, YoastSEO
  ├─> SEO: SEOPromptBuilder, SchemaMarkupGenerator
  └─> Agents: ResearchAgent, ContentAuditor

run_content_generation()
  │
  ├─> 1. Topic Selection (trend_sources.get_hot_trend)
  ├─> 2. Content Generation (Gemini with SEO schema)
  ├─> 3. Image Generation (with rate limiting)
  ├─> 4. Publishing (WordPress with schema markup)
  └─> 5. Yoast Meta Update
```

### Rate Limiting

**GeminiRateLimiter** implements Google's official rate limit best practices:
- RPM (Requests Per Minute) - rolling 60-second window
- RPD (Requests Per Day) - resets at midnight Pacific Time
- Per-model tracking for image generation limits
- Exponential backoff: 1s → 2s → 4s → ... → 120s max
- Proactive throttling before hitting limits

Reference: https://ai.google.dev/gemini-api/docs/rate-limits

### Gemini Client Authentication

Two modes supported:
1. **API Key** (`GEMINI_API_KEY`) - For Gemini API (generativelanguage.googleapis.com)
   - Required for image generation models (gemini-3-pro-image-preview)
   - Free tier limits apply

2. **Service Account** (`GEMINI_SERVICE_ACCOUNT_KEY_FILE`) - For Vertex AI
   - Enables regional endpoint rotation for quota distribution
   - Regions: us-central1, us-east1, us-west1, europe-west1, asia-southeast1
   - Auto-rotates on rate limits via `rotate_region()`

### SEO System

**SEOPromptBuilder** creates structured prompts for content generation
**SchemaMarkupGenerator** generates JSON-LD structured data for articles

Pydantic schema for article metadata:
```python
class SEOArticleMetadata(BaseModel):
    content: str
    seo_title: str  # max 60 chars
    meta_description: str  # max 160 chars
    focus_keyword: str
    excerpt: str
    suggested_categories: List[str]
    suggested_tags: List[str]
```

### Image Generation Fallback Chain

1. **Gemini 3 Pro Image Preview** (gemini-3-pro-image-preview)
   - Requires API key (not Vertex AI)
   - 4K output, Google Search grounding
   - Config: aspect_ratio="1:1", image_size="4K"

2. **DALL-E 3** (OpenAI)
   - High quality fallback
   - Requires `OPENAI_API_KEY`

3. **Hugging Face FLUX.1-dev**
   - Last resort
   - Requires `HUGGINGFACE_API_KEY`

### Maintenance/Auditor

**ContentAuditor** optimizes existing posts:
- Fetches published posts via WordPress API
- Analyzes content quality and SEO
- Updates outdated information
- Fixes broken links
- Adds missing featured images
- Updates Yoast meta fields

## Testing

- `tests/test_integration.py` - Full content generation pipeline
- `tests/test_main.py` - Main system tests
- `tests/test_research_agent.py` - Research agent tests
- `tests/test_publish.py` - WordPress publishing tests
- `test_image_gen.py` - Standalone image generation test
- `test_diagnostic.py` - Diagnostic utilities

## Key Files

- `src/main.py` - Main orchestration and system initialization
- `src/image_generator.py` - Image generation with rate limiting
- `src/clients/gemini.py` - Gemini client with region rotation
- `src/clients/wordpress.py` - WordPress REST API client
- `src/seo_system.py` - SEO prompt and schema generation
- `src/auditor.py` - Content maintenance and optimization
- `src/yoast_seo.py` - Yoast SEO plugin integration
- `src/trend_sources.py` - Trending topic fetchers

---
> Source: [TK7684/Auto-Blogger-WP](https://github.com/TK7684/Auto-Blogger-WP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
