## hki-pw-frontend

> Frontend application for HKI 2050, a cyberpunk pen-and-paper tabletop RPG set in Helsinki in 2050.

# HKI 2050 - Cyberpunk PnP Frontend

Frontend application for HKI 2050, a cyberpunk pen-and-paper tabletop RPG set in Helsinki in 2050.

## Tech Stack

- **Framework:** Vue 3 (Composition API with `<script setup>`)
- **Build Tool:** Vite 5
- **Language:** TypeScript 5
- **Styling:** SCSS + Bootstrap 5 (dark theme)
- **State Management:** Pinia
- **Routing:** Vue Router 4
- **Icons:** Heroicons Vue
- **Markdown:** Marked

## Project Structure

```
src/
├── assets/          # Images, SVGs, backgrounds
├── components/      # Reusable Vue components
├── router/          # Vue Router configuration
├── scss/            # Global styles and theme
├── stores/          # Pinia state stores
├── types/           # TypeScript interfaces
├── views/           # Page components (routes)
├── App.vue          # Root component
└── main.ts          # Entry point
```

## Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server
npm run build        # Production build
npm run preview      # Preview production build
```

## Key Patterns

### Component Conventions
- Use Composition API with `<script setup lang="ts">`
- Props typed with `defineProps<T>()`
- Emits typed with `defineEmits<T>()`
- Scoped SCSS styling

### API Calls
- Base URL: `https://hki2050.com/api/`
- Auth: Bearer token in `Authorization` header
- API key: `X-API-Key` header from `VITE_PW_APIKEY` env var
- Direct fetch calls in components (no abstraction layer)

### State Management
- `userStore`: JWT authentication state
- `thingsStore`: Items/weapons with 5-minute caching

### Styling
- Theme colors: magenta (`#ff047b`), cyan (`#00f0ff`)
- Glitch effects for cyberpunk aesthetic
- Clip-path for angled corners
- Bootstrap grid system

### Types
- `Listable`: Base interface (id, name, created_at, image_url)
- `Character`: Extends Listable with stats, attributes, inventory, portrait
- `Thing`: Items/weapons with effects
- `Article`: Worldbuilding lore content

## Routes

| Path | Auth | Purpose |
|------|------|---------|
| `/` | No | Landing page |
| `/login` | No | Authentication |
| `/characters` | Yes | Character list |
| `/characters/:id` | Yes | Character detail/create |
| `/articles` | Yes | Article list |
| `/articles/:id` | Yes | Article detail |
| `/things` | Yes | Item browser |
| `/things/:id` | Yes | Item detail |

Base path is `/app/` for all routes.

## Key Components

- **CharacterForm.vue**: Complex character editor with skill calculations, inventory management, stat visualization, portrait upload
- **SearchableDropdown.vue**: Reusable filtered item selector
- **MasonryListing.vue**: Grid layout for cards
- **ThingListing.vue**: Item grid with effect badges

## Portrait Upload

Characters support portrait images via base64 encoding:
- Accepts: jpeg, png, gif, webp
- File is converted to base64 data URI in `CharacterForm.vue`
- Stored temporarily in `uploadImage` field, sent as `images` array in API payload
- API response includes `images` Record (URLs keyed by filename)

## Configuration

- **vite.config.ts**: Base path `/app/`, SCSS global imports, `@` alias
- **.prettierrc**: Tabs, no semicolons, 120 char width
- **.env**: `VITE_PW_APIKEY` for API authentication

## Known Limitations

- API URLs partially hardcoded (consider env vars)
- No automated tests
- Limited error feedback to users
- JWT stored in localStorage
- No route-level code splitting

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ninjapiraatti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
