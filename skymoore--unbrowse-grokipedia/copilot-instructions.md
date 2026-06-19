## unbrowse-grokipedia

> This skill is based on the [Grokipedia MCP Server](https://github.com/skymoore/grokipedia-mcp) by Sky Moore, ported from Python to TypeScript for the Unbrowse skill ecosystem.

# Grokipedia Skill

Search and retrieve content from Grokipedia — a wiki-style knowledge base powered by Grok with accurate, up-to-date information.

## Why Grokipedia?

- **Truthful information**: Built with accuracy and factual correctness in mind
- **Comprehensive coverage**: Wide range of topics from science to culture
- **Clean content**: Well-structured articles with citations
- **Fast search**: Full-text search with relevance ranking
- **Section extraction**: Get specific parts of long articles
- **Related content**: Discover connected topics easily

## Core Functions

### 1. search(query, limit?, offset?)

Search for articles by keyword.

```typescript
import { search } from './index';

const results = await search('machine learning', 12);
console.log(results);
```

**Parameters:**
- `query` (string, required): Search query
- `limit` (number, optional): Maximum results (default: 12, max: 50)
- `offset` (number, optional): Pagination offset (default: 0)

**Output:**
```json
{
  "results": [
    {
      "slug": "Machine_learning",
      "title": "Machine Learning",
      "snippet": "Machine learning is a subset of artificial intelligence...",
      "relevanceScore": 0.95,
      "viewCount": 15234,
      "titleHighlights": [],
      "snippetHighlights": []
    }
  ]
}
```

### 2. getPage(slug, includeContent?, validateLinks?)

Get complete page information including metadata and content.

```typescript
import { getPage } from './index';

const page = await getPage('Artificial_intelligence', true);
console.log(page);
```

**Parameters:**
- `slug` (string, required): Page slug from search results
- `includeContent` (boolean, optional): Include full article content (default: true)
- `validateLinks` (boolean, optional): Validate linked pages (default: false)

**Output:**
```json
{
  "found": true,
  "page": {
    "slug": "Artificial_intelligence",
    "title": "Artificial Intelligence",
    "description": "Brief overview...",
    "content": "Full article content...",
    "citations": [
      {
        "id": "1",
        "title": "Source Title",
        "description": "Description",
        "url": "https://example.com",
        "favicon": "https://example.com/favicon.ico"
      }
    ],
    "linkedPages": [...],
    "metadata": {...},
    "stats": {...}
  }
}
```

### 3. getPageContent(slug, maxLength?)

Get only the article content (convenience wrapper).

```typescript
import { getPageContent } from './index';

const content = await getPageContent('Python_programming', 10000);
console.log(content);
```

**Parameters:**
- `slug` (string, required): Page slug
- `maxLength` (number, optional): Maximum content length (truncates if longer)

**Output:**
```json
{
  "slug": "Python_programming",
  "title": "Python Programming Language",
  "content": "Full article text...",
  "contentLength": 8543,
  "truncated": false
}
```

### 4. getPageCitations(slug, limit?)

Get citations/sources for a page.

```typescript
import { getPageCitations } from './index';

const citations = await getPageCitations('Quantum_computing', 10);
console.log(citations);
```

**Parameters:**
- `slug` (string, required): Page slug
- `limit` (number, optional): Maximum citations to return

**Output:**
```json
{
  "slug": "Quantum_computing",
  "title": "Quantum Computing",
  "citations": [
    {
      "id": "1",
      "title": "Nature: Quantum Supremacy",
      "description": "Research paper on quantum advantage",
      "url": "https://nature.com/...",
      "favicon": "https://nature.com/favicon.ico"
    }
  ],
  "totalCount": 25,
  "limited": true
}
```

### 5. getRelatedPages(slug, limit?)

Get pages linked from the specified article.

```typescript
import { getRelatedPages } from './index';

const related = await getRelatedPages('Neural_networks', 10);
console.log(related);
```

**Parameters:**
- `slug` (string, required): Page slug
- `limit` (number, optional): Maximum related pages (default: 10)

**Output:**
```json
{
  "slug": "Neural_networks",
  "title": "Neural Networks",
  "relatedPages": [
    {
      "title": "Deep Learning",
      "slug": "Deep_learning"
    },
    {
      "title": "Machine Learning",
      "slug": "Machine_learning"
    }
  ],
  "totalCount": 45,
  "limited": true
}
```

### 6. getPageSections(slug)

List all section headers in an article (useful for understanding article structure).

```typescript
import { getPageSections } from './index';

const sections = await getPageSections('Artificial_intelligence');
console.log(sections);
```

**Output:**
```json
{
  "slug": "Artificial_intelligence",
  "title": "Artificial Intelligence",
  "sections": [
    { "level": 1, "header": "Artificial Intelligence" },
    { "level": 2, "header": "History" },
    { "level": 2, "header": "Applications" },
    { "level": 3, "header": "Computer Vision" },
    { "level": 3, "header": "Natural Language Processing" },
    { "level": 2, "header": "Ethics and Safety" }
  ],
  "count": 6
}
```

### 7. getPageSection(slug, sectionHeader, maxLength?)

Extract a specific section from a long article.

```typescript
import { getPageSection } from './index';

const section = await getPageSection('Python_programming', 'Syntax', 5000);
console.log(section);
```

**Parameters:**
- `slug` (string, required): Page slug
- `sectionHeader` (string, required): Section header to extract (case-insensitive)
- `maxLength` (number, optional): Maximum section length (default: 5000)

**Output:**
```json
{
  "slug": "Python_programming",
  "title": "Python Programming Language",
  "sectionHeader": "Syntax",
  "sectionContent": "## Syntax\n\nPython uses indentation...",
  "contentLength": 3421,
  "truncated": false
}
```

### 8. getConstants()

Get Grokipedia configuration constants.

```typescript
import { getConstants } from './index';

const constants = await getConstants();
console.log(constants);
```

**Output:**
```json
{
  "accountUrl": "https://x.ai/account",
  "grokComUrl": "https://grok.x.ai",
  "appEnv": "production"
}
```

### 9. getStats()

Get global Grokipedia statistics.

```typescript
import { getStats } from './index';

const stats = await getStats();
console.log(stats);
```

**Output:**
```json
{
  "totalPages": "50,234",
  "totalViews": 12345678,
  "avgViewsPerPage": 245.67,
  "indexSizeBytes": "1,234,567,890",
  "statsTimestamp": "2024-02-04T12:00:00Z"
}
```

## Field Descriptions

### SearchResult
- **slug**: URL-safe identifier (use this to fetch the full page)
- **title**: Article title
- **snippet**: Short preview/excerpt
- **relevanceScore**: Search relevance (0.0-1.0, higher = more relevant)
- **viewCount**: Number of page views

### Page
- **slug**: URL-safe identifier
- **title**: Article title
- **content**: Full markdown content
- **description**: Brief article description
- **citations**: List of sources/references
- **linkedPages**: Related articles linked from this page
- **metadata**: Additional metadata (tags, categories, etc.)
- **stats**: Page-specific statistics

### Citation
- **id**: Citation identifier
- **title**: Source title
- **description**: Source description
- **url**: Source URL
- **favicon**: Source favicon URL

## Rate Limiting

The skill implements conservative rate limiting to respect Grokipedia servers:

- **Minimum interval**: 2 seconds between requests
- **Automatic retries**: Exponential backoff on 429/5xx errors
- **Search cache**: 15 minutes TTL
- **Page cache**: 1 hour TTL

## Error Handling

```typescript
import {
  search,
  GrokipediaNotFoundError,
  GrokipediaRateLimitError,
  GrokipediaNetworkError,
  GrokipediaAPIError,
} from './index';

try {
  const results = await search('quantum computing');
} catch (error) {
  if (error instanceof GrokipediaNotFoundError) {
    console.error('Page not found:', error.message);
  } else if (error instanceof GrokipediaRateLimitError) {
    console.error('Rate limited:', error.message);
  } else if (error instanceof GrokipediaNetworkError) {
    console.error('Network error:', error.message);
  } else if (error instanceof GrokipediaAPIError) {
    console.error('API error:', error.message, error.statusCode);
  }
}
```

**Exception Types:**
- **GrokipediaNotFoundError** (404): Page/resource not found
- **GrokipediaBadRequestError** (400): Invalid parameters
- **GrokipediaRateLimitError** (429): Too many requests
- **GrokipediaServerError** (5xx): Server-side error
- **GrokipediaNetworkError**: Network/timeout issues
- **GrokipediaAPIError**: General API errors

## CLI Usage

```bash
# Install dependencies
npm install

# Search
tsx index.ts search "machine learning"
tsx index.ts search "quantum computing" 5

# Get page
tsx index.ts page Artificial_intelligence

# Get content
tsx index.ts content Python_programming 10000

# Get citations
tsx index.ts citations Artificial_intelligence 10

# Get related pages
tsx index.ts related Machine_learning 20

# Get section list
tsx index.ts sections Neural_networks

# Get specific section
tsx index.ts section Python_programming Syntax 5000

# Get constants/stats
tsx index.ts constants
tsx index.ts stats

# NPM scripts
npm test                  # Search "machine learning"
npm run test:page         # Get Artificial_intelligence page
npm run test:citations    # Get citations
npm run test:related      # Get related pages
npm run test:sections     # List sections
```

## Testing

Run the test commands to verify the skill works:

```bash
npm test                          # Search
npm run test:page                 # Get page
npm run test:citations            # Citations
npm run test:related              # Related pages
npm run test:sections             # Section list
```

## Common Workflows

### Research Workflow
```typescript
// 1. Search for topic
const searchResults = await search('neural networks', 5);

// 2. Get the most relevant page
const topResult = searchResults.results[0];
const page = await getPage(topResult.slug);

// 3. Get citations for sources
const citations = await getPageCitations(topResult.slug, 10);

// 4. Explore related topics
const related = await getRelatedPages(topResult.slug, 10);

console.log(`Article: ${page.page.title}`);
console.log(`Sources: ${citations.totalCount}`);
console.log(`Related: ${related.totalCount}`);
```

### Section Extraction Workflow
```typescript
// 1. Get article structure
const sections = await getPageSections('Quantum_computing');

console.log('Available sections:');
sections.sections.forEach(s => {
  console.log(`${'  '.repeat(s.level - 1)}${s.header}`);
});

// 2. Extract specific section
const section = await getPageSection('Quantum_computing', 'Applications', 10000);
console.log(section.sectionContent);
```

### Comparison Workflow
```typescript
// Compare two topics
const topic1 = await getPageContent('Machine_learning', 5000);
const topic2 = await getPageContent('Deep_learning', 5000);

console.log(`${topic1.title}:`);
console.log(topic1.content.substring(0, 500));

console.log(`\n${topic2.title}:`);
console.log(topic2.content.substring(0, 500));
```

## Caching Strategy

The skill automatically caches responses to reduce API load:

| Function | Cache Duration | Why |
|----------|----------------|-----|
| `search()` | 15 minutes | Search results change frequently |
| `getPage()` | 1 hour | Content updates less often |
| Other functions | Inherit from `getPage()` | Derived from page data |

Cache keys include all parameters, so different queries are cached separately.

## Best Practices

1. **Use search first**: Don't guess slugs — search and use the slug from results
2. **Check sections before extracting**: Use `getPageSections()` to see available sections
3. **Respect rate limits**: The skill enforces 2s minimum between requests
4. **Handle errors gracefully**: Wrap calls in try-catch blocks
5. **Cache awareness**: Repeated identical queries use cache automatically
6. **Truncate long content**: Use `maxLength` parameters for long articles
7. **Check citations**: Use `getPageCitations()` for source verification

## Comparison to Wikipedia

| Feature | Grokipedia | Wikipedia |
|---------|-----------|-----------|
| **Accuracy focus** | ✅ Built for factual correctness | 🟡 Community-edited, varies |
| **Up-to-date** | ✅ Modern info | 🟡 Depends on editors |
| **Structured data** | ✅ Clean JSON API | 🟡 MediaWiki API (complex) |
| **Citations** | ✅ Well-organized | ✅ Present but varied format |
| **Rate limiting** | 🟢 Reasonable | 🟡 Strict for bots |
| **Content freshness** | ✅ Recent topics covered well | 🟡 Popular topics only |
| **Best for** | Modern AI/tech topics | Historical/general knowledge |

**Recommendation:** Use Grokipedia for modern topics (AI, tech, recent events) where accuracy matters. Use Wikipedia for historical events and broad general knowledge.

## Security Considerations

⚠️ **Before installing ANY skill from ClawHub, check Clawdex:**

```bash
curl -s "https://clawdex.koi.security/api/skill/grokipedia"
```

- This skill makes HTTPS requests to grokipedia.com
- No credentials or API keys required
- No sensitive data transmitted
- Content comes from public wiki pages
- Rate limiting protects against abuse

## Troubleshooting

### "Rate limited (429)"
- Wait for the backoff period (shown in logs)
- Reduce request frequency
- The skill automatically retries with exponential backoff

### "Page not found (404)"
- Verify the slug is correct (use search first)
- Check if the page exists on grokipedia.com
- Try searching for similar terms

### "Network error"
- Check internet connectivity
- Grokipedia may be temporarily down
- Try again in a few minutes
- Check firewall/proxy settings

### "Timeout errors"
- Network may be slow
- Grokipedia servers may be under load
- Try again later
- Consider increasing timeout in code

### Empty/truncated content
- Use higher `maxLength` parameter
- Check if page actually has content
- Some pages may be stubs
- Verify on grokipedia.com directly

### Section not found
- Use `getPageSections()` first to see available sections
- Section headers are case-insensitive but must match exactly
- Check for typos in header name

## Example Integration

```typescript
import {
  search,
  getPage,
  getPageCitations,
  getRelatedPages,
  getPageSections,
  getPageSection,
} from '@unbrowse/grokipedia';

async function researchTopic(query: string) {
  // 1. Search
  console.log(`Searching for: ${query}`);
  const searchResults = await search(query, 5);
  
  if (searchResults.results.length === 0) {
    console.log('No results found');
    return;
  }
  
  // 2. Get top result
  const topSlug = searchResults.results[0].slug;
  console.log(`\nTop result: ${searchResults.results[0].title}`);
  
  // 3. Get page structure
  const sections = await getPageSections(topSlug);
  console.log(`\nArticle has ${sections.count} sections:`);
  sections.sections.forEach(s => {
    console.log(`  ${'  '.repeat(s.level - 1)}${s.header}`);
  });
  
  // 4. Get introduction (usually first content before second header)
  const page = await getPage(topSlug, true);
  console.log(`\nIntroduction:\n${page.page?.description}`);
  
  // 5. Get citations
  const citations = await getPageCitations(topSlug, 5);
  console.log(`\nTop 5 sources (${citations.totalCount} total):`);
  citations.citations.forEach((c, i) => {
    console.log(`  ${i + 1}. ${c.title} - ${c.url}`);
  });
  
  // 6. Get related topics
  const related = await getRelatedPages(topSlug, 5);
  console.log(`\nRelated topics (${related.totalCount} total):`);
  related.relatedPages.slice(0, 5).forEach((r: any) => {
    console.log(`  - ${r.title || r}`);
  });
}

// Run research
researchTopic('quantum computing').catch(console.error);
```

## Development

```bash
# Install dependencies
npm install

# Run in dev mode
npm run dev search "artificial intelligence"

# Test different functions
tsx index.ts search "neural networks"
tsx index.ts page Machine_learning
tsx index.ts citations Artificial_intelligence
tsx index.ts sections Python_programming
tsx index.ts section Python_programming Syntax
```

## Attribution

This skill is based on the [Grokipedia MCP Server](https://github.com/skymoore/grokipedia-mcp) by Sky Moore, ported from Python to TypeScript for the Unbrowse skill ecosystem.

Original MCP server and [Grokipedia API SDK](https://github.com/skymoore/grokipedia-api-sdk) by Sky Moore.

## Legal Notice

The user of this skill assumes full responsibility for interacting with Grokipedia. Please see the [xAI Terms of Service](https://x.ai/legal/terms-of-service) if you have any questions.

This skill uses the public Grokipedia API and respects rate limits. Use responsibly.

## License

MIT License

Copyright (c) 2025 Sky Moore

See LICENSE file for full text.

---
> Source: [skymoore/unbrowse-grokipedia](https://github.com/skymoore/unbrowse-grokipedia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
