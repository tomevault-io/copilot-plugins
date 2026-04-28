## webpolimorfica

> The site supports **Spanish (ES)** and **English (EN)**. Every new page or feature must include translations for both languages.

# WebPolimorfica — Developer Guide

## i18n (Internationalization)

The site supports **Spanish (ES)** and **English (EN)**. Every new page or feature must include translations for both languages.

### Architecture

| File | Purpose |
|------|---------|
| `lib/i18n/translations.js` | All translation strings for ES and EN |
| `lib/i18n/server.js` | Server-side helpers: `getLocale()`, `serverT(section)` |
| `components/LocaleProvider.js` | React context + `useLocale()` and `useT(section)` hooks |
| `components/LanguageSwitcher.js` | ES/EN toggle buttons in the header |

### Server Components

```js
import { serverT } from '../lib/i18n/server';

export default function MyPage() {
  const t = serverT('mySection');
  return <h1>{t.title}</h1>;
}
```

### Client Components

```js
'use client';
import { useT } from '../../components/LocaleProvider';

export default function MyComponent() {
  const t = useT('mySection');
  return <button>{t.label}</button>;
}
```

### Adding a New Section

1. Add your translation keys to **both** `es` and `en` objects in `lib/i18n/translations.js`:

```js
// In translations.js
es: {
  // ...existing sections...
  myNewSection: {
    title: 'Mi Nueva Sección',
    description: 'Descripción en español.',
    button: 'Hacer algo',
  },
},
en: {
  // ...existing sections...
  myNewSection: {
    title: 'My New Section',
    description: 'Description in English.',
    button: 'Do something',
  },
},
```

2. Use `serverT('myNewSection')` in server components or `useT('myNewSection')` in client components.

### Function Values (Dynamic Strings)

For strings that include variable data, use functions:

```js
// In translations.js
es: {
  mySection: {
    found: (n) => `${n} resultado${n > 1 ? 's' : ''} encontrado${n > 1 ? 's' : ''}`,
  },
},
en: {
  mySection: {
    found: (n) => `${n} result${n > 1 ? 's' : ''} found`,
  },
},

// Usage in component:
<span>{t.found(results.length)}</span>
```

### How Locale Switching Works

- Locale is stored in a cookie (`locale=es` or `locale=en`), persisted for 1 year.
- `LanguageSwitcher` updates the cookie and calls `router.refresh()` to re-render server components.
- Default locale is `'es'` (Spanish).
- The `<html lang={locale}>` attribute is set dynamically in `app/layout.js`.

### Existing Sections

| Section key | Used in |
|-------------|---------|
| `nav` | `app/layout.js` |
| `home` | `app/page.js` |
| `juegos` | `app/juegos/page.js` |
| `herramientas` | `app/herramientas/page.js` |
| `solicitud` | `app/components/SolicitudCard.js` |
| `buscaminas` | `app/juegos/buscaminas/page.js` |
| `ajedrez` | `app/juegos/ajedrez/page.js` |
| `tresEnRaya` | `app/juegos/tres-en-raya/page.js` |
| `cuatroEnRaya` | `app/juegos/cuatro-en-raya/page.js` |
| `tetris` | `app/juegos/tetris/page.js` |
| `quitarFondo` | `app/herramientas/quitar-fondo/page.js` |
| `youtubeMp3` | `app/herramientas/youtube-mp3/page.js` |
| `buscarEmails` | `app/herramientas/buscar-emails/page.js` |
| `buscarPersonaEmail` | `app/herramientas/buscar-persona-email/page.js` |
| `curriculum` | `app/curriculum/page.js` |

## Project Structure

```
app/
  layout.js          # Root layout — header, nav, footer, LocaleProvider
  page.js            # Home page
  curriculum/        # CV page
  juegos/            # Games index + individual game pages
  herramientas/      # Tools index + individual tool pages
  api/               # Server-side API routes
  components/        # Shared app-level components (SolicitudCard, NavLink)
components/          # Root-level shared components (LocaleProvider, LanguageSwitcher)
lib/i18n/            # i18n system (translations.js, server.js)
public/              # Static assets
```

## Tech Stack

- **Next.js 14.2.3** (App Router)
- **React** with client components where needed
- **CSS** via `globals.css` (no Tailwind)
- **Deployment**: DeployFast (Docker + GitHub auto-deploy)

## Checklist for New Features

- [ ] Add translation keys to `lib/i18n/translations.js` for both `es` and `en`
- [ ] Use `serverT` in server components, `useT` in client components
- [ ] Test with both locales using the language switcher in the header
- [ ] Run `npm run build` to verify no TypeScript/build errors before committing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zeuscabanas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
