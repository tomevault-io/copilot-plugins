## mra-frontend

> This is a **multi-tenant motorsport website skeleton** powered by **Strapi CMS**. The core concept is:

# Motorsport Multi-Tenant CMS Project Rules

## Project Overview
This is a **multi-tenant motorsport website skeleton** powered by **Strapi CMS**. The core concept is:
- **Single codebase** serves multiple motorsport event websites
- **Content is dynamically loaded** from Strapi based on `SITE_SLUG` environment variable
- **Simply change the site slug** and the entire website transforms (branding, content, events, news, etc.)
- **No code changes needed** to deploy different event sites

## Architecture Philosophy

### Core Principle: **Data-Driven Multi-Tenancy**
```
SITE_SLUG=wakefield300 → Wakefield 300 racing event site
SITE_SLUG=gtworldchallenge → GT World Challenge site  
SITE_SLUG=gt4australia → GT4 Australia site
```

### Content Flow
```
Strapi CMS → API Routes → Components → Dynamic Site
```

1. **Strapi** stores all content (events, news, sponsors, documents, branding)
2. **API routes** (`/api/config`, `/api/news`, `/api/events`) fetch site-specific data
3. **Components** render content dynamically based on fetched data
4. **Site transforms** completely based on `SITE_SLUG`

## Key Components & Responsibilities

### Configuration System
- **`/api/config`**: Primary API that fetches site configuration from Strapi
- **Fallback**: Local JSON configs in `/src/config/` for development
- **Dynamic branding**: Colors, logos, site title, navigation all from Strapi

### Content APIs
- **`/api/news`**: Site-specific news articles with image URL normalization
- **`/api/events`**: Event listings (Strapi + RaceReady integration)
- **`/api/menu`**: Dynamic navigation items

### Page Structure
- **`/`**: Homepage with Hero, Latest News, Event Documents, Sponsors
- **`/events`**: Event listings page
- **`/events/[id]`**: Detailed event page with documents, entry lists, schedule
- **`/event/[id]`**: RaceReady-powered event page with countdown
- **`/event-info`**: General event information page
- **`/news/[slug]`**: Individual news article pages

## Development Guidelines

### When Adding Features
1. **Make it data-driven**: New features should pull from Strapi, not hardcode content
2. **Consider multi-tenancy**: Will this work for different event types/sites?
3. **Fallback gracefully**: Always have fallbacks for missing Strapi data
4. **Normalize URLs**: Always convert relative Strapi media URLs to absolute

### Content Management
- **All content lives in Strapi**: Events, news, sponsors, documents, branding
- **Images**: Strapi media fields → normalized to absolute URLs in API transforms
- **Branding**: Primary colors, logos, site titles from Strapi site configuration
- **Navigation**: Menu items and website links from Strapi

### URL Handling Rules
- **Always link internally**: Convert external Strapi URLs to internal pages
- **News**: External URLs → `/news/[slug]` internal pages
- **Events**: External event URLs → `/event/[id]` or `/events/[id]` pages
- **Documents**: Link to actual document URLs (PDFs, etc.)

### API Response Transformation
```javascript
// Always normalize Strapi media URLs
const strapiUrl = process.env.STRAPI_URL || "";
image: item.image?.url 
  ? (item.image.url.startsWith('http') ? item.image.url : `${strapiUrl}${item.image.url}`)
  : null
```

### Component Patterns
- **Use `useConfig()`**: Get site configuration in client components
- **Server-side fetching**: Use `/api/config` in server components and pages
- **Graceful degradation**: Handle missing/invalid data elegantly
- **URL validation**: Always validate and sanitize URLs before using in `next/link` or `next/image`

## Environment Variables
```env
SITE_SLUG=wakefield300           # Determines which site content to load
STRAPI_URL=https://cms.domain.com # Strapi backend URL
STRAPI_API_TOKEN=your_token      # Strapi API authentication
SITE_DOMAIN=https://your-site.com # Frontend domain for API calls
```

## File Structure Logic

### Static Structure (Skeleton)
- Components, layouts, UI elements
- Page templates and routing
- API route handlers
- Styling and configuration

### Dynamic Content (From Strapi)
- Site branding (colors, logos, titles)
- Navigation menus and website links
- News articles and images
- Event information and documents
- Sponsor logos and links
- Hero content and buttons

### Config Fallbacks
- `/src/config/site-config.json` - Default/fallback config
- `/src/config/site-config-[slug].json` - Site-specific fallbacks

## Integration Points

### Strapi CMS Structure
Expected Strapi content types:
- **Sites**: Main site configuration (slug, branding, hero, navigation)
- **News**: Articles with images, slugs, content
- **Events**: Event data, documents, schedules
- **Sponsors**: Logos and links
- **Documents**: Event-related files

### RaceReady Integration
- `/api/raceready/event/[id]` - External event data integration
- Used as fallback when Strapi events are unavailable
- Provides entry lists, detailed schedules, live timing

## Deployment Strategy
1. **Single deployment** serves all sites
2. **Environment variables** determine site identity
3. **Strapi content** drives all differences
4. **No code changes** needed for new event sites

## Best Practices

### URL Safety
- Always validate URLs before passing to Next.js components
- Normalize external URLs to internal routes when possible
- Use try-catch around URL construction

### Content Validation
- Check for required fields before rendering
- Provide sensible fallbacks for missing content
- Handle both v4 and v5 Strapi response formats

### Performance
- Use appropriate caching strategies (`no-store` for dynamic content)
- Optimize images through Next.js Image component
- Lazy load non-critical content

### SEO
- Generate dynamic metadata based on site configuration
- Use proper OpenGraph and Twitter meta tags
- Implement structured data for events and articles

## Common Patterns

### Fetching Site Config
```javascript
// Server component
const config = await fetch(`${baseUrl}/api/config`).then(r => r.json());

// Client component  
const config = useConfig();
```

### Internal Link Conversion
```javascript
// Convert external URLs to internal routes
const internalHref = externalUrl.includes('event') ? '/events' : '/news';
```

### Media URL Normalization
```javascript
// Ensure all media URLs are absolute
const normalizedUrl = url.startsWith('http') ? url : `${strapiUrl}${url}`;
```

## Goals
- **Zero-touch deployment**: Change slug → new site
- **Content editor friendly**: All changes through Strapi CMS
- **Developer efficient**: Single codebase for multiple sites
- **Performance optimized**: Fast, modern Next.js architecture
- **SEO ready**: Dynamic metadata and structured data

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mmeguizo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mmeguizo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
