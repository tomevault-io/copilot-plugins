## relentless-rigor

> This project uses next-intl for internationalization. ALL user-facing text MUST be translated.

# Internationalization (i18n) Rules

This project uses next-intl for internationalization. ALL user-facing text MUST be translated.

## Mandatory Requirements

1. **Never hardcode user-facing strings** - Use translation keys from `messages/{locale}.json`
2. **When adding new text**, add keys to BOTH `messages/en.json` AND `messages/es.json`
3. **Component text** - Use `useTranslations()` hook in client components, `getTranslations()` in server components
4. **Article content** - Create separate MDX files per locale (e.g., `page.en.mdx`, `page.es.mdx`)
5. **Data files** - Use `descriptionKey` pattern (see `src/lib/projects.ts`) instead of hardcoded descriptions

## File Patterns

- Translation files: `messages/en.json`, `messages/es.json`
- Locale config: `i18n.ts`
- Request config: `src/i18n/request.ts`
- All pages under: `src/app/[locale]/`

## Adding New Content

### New Page
1. Create under `src/app/[locale]/your-page/page.tsx`
2. Add translation namespace to `messages/*.json`
3. Use `getTranslations('namespace')` for server components

### New Article
1. Create folder: `src/app/[locale]/articles/article-slug/`
2. Create `page.en.mdx` and `page.es.mdx`
3. Update `src/lib/articles.ts` to load locale-specific file

### New Component with Text
1. Import: `import { useTranslations } from 'next-intl'`
2. Get translator: `const t = useTranslations('namespace')`
3. Use: `{t('keyName')}`

## Translation Key Naming Conventions

- Use camelCase for keys: `readArticle`, `noArticles`
- Group by feature/page: `articles.title`, `home.newsletter.title`
- Keep keys descriptive but concise
- Avoid abbreviations

## Common Patterns

### Server Component
```tsx
import { getTranslations } from 'next-intl/server'

export default async function Page() {
  const t = await getTranslations('namespace')
  return <h1>{t('title')}</h1>
}
```

### Client Component
```tsx
'use client'
import { useTranslations } from 'next-intl'

export function Component() {
  const t = useTranslations('namespace')
  return <button>{t('buttonText')}</button>
}
```

### Data with Translation Keys
```typescript
// In lib file
export const items = [
  { name: 'Item', descriptionKey: 'itemDescription' }
]

// In component
const t = useTranslations('items.descriptions')
items.map(item => <p>{t(item.descriptionKey)}</p>)
```

## Checklist Before Committing

- [ ] All new user-facing strings use translation keys
- [ ] Keys added to both en.json and es.json
- [ ] No hardcoded strings in JSX
- [ ] Article content has both locale versions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/diegogallovich) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
