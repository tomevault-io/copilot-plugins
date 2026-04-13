## beehiiv-newsletter-advisor-site

> When the user provides a URL to add as a new product:

# Project Instructions for Claude

## Adding New Products

When the user provides a URL to add as a new product:

1. **ALWAYS fetch the URL immediately** - never ask for details, just fetch and extract (title, description, author)

2. **Add entry to `src/data.ts`** with this format:
```typescript
{
  id: 'url-friendly-slug',        // lowercase, hyphens, based on title
  title: 'Product Title',
  category: 'Digital Products',   // always use this
  description: 'Brief description of the product.',
  author: 'Creator/Author Name',
  url: 'https://the-provided-url.com',
  tags: ['Relevant', 'Tags'],     // pick from existing or add new
  date: 'YYYY-MM-DD',             // use current date
},
```

3. **Tags** should be relevant to the product type (e.g., Newsletter, Playbook, Ebook, Course, Template, Founders, etc.)

4. **The product will automatically**:
   - Appear on home page and browse page
   - Have its own detail page at `/products/{id}`
   - Show iframe preview (scaled like thumbnail)
   - Have clickable tags that filter browse page
   - Have clickable author linking to author page

## Tech Stack
- React 18 + TypeScript + Vite
- Tailwind CSS (via CDN)
- Running on localhost:3005

## Key Files
- `src/data.ts` - Product data
- `src/pages/Detail.tsx` - Product detail page
- `src/components/ProductCard.tsx` - Product card with iframe preview
- `src/components/Header.tsx` - Nav with dynamic tags from data

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/beehiiv)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/beehiiv)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
