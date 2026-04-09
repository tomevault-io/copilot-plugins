## unicornstudio

> AI-gestützter Page Builder mit WordPress-Integration.

# Unicorn Studio

AI-gestützter Page Builder mit WordPress-Integration.

## Tech Stack
- **Frontend**: Next.js 16, React, TypeScript, Tailwind CSS v4
- **Backend**: Supabase (PostgreSQL + Auth + Storage)
- **AI**: Anthropic Claude, Google GenAI
- **Deployment**: Vercel
- **CMS**: WordPress Plugin für Sync

## Projektstruktur
```
app/
  (auth)/          # Login/Signup
  (editor)/        # Page Editor
  api/
    ai/            # AI Generation
    v1/sites/      # REST API
    internal/      # WordPress Webhooks
  dashboard/       # Site Management
components/        # React Components
lib/
  ai/              # AI Prompts & Operations
  supabase/        # DB Client & Queries
stores/            # Zustand Store (editor-store.ts)
types/             # TypeScript Types
plugins/wordpress/ # WP Plugin Source
supabase/migrations/ # DB Schema
```

## Datenbank (Supabase)

### Wichtige Tabellen
- `organizations` - Workspaces/Kunden
- `profiles` - User (verlinkt mit auth.users)
- `sites` - Websites mit `global_header_id`, `global_footer_id`
- `pages` - Seiten mit `html_content`, `content` JSON
- `components` - Header/Footer mit `html`, `css`, `js`, `position`
- `cms_components` - Wiederverwendbare UI Components
- `templates` - Seitenvorlagen
- `content_types` - CMS Content Types
- `entries` - CMS Einträge
- `design_variables` - Design Tokens

### Supabase CLI
```bash
# Lokal
npx supabase start
npx supabase db reset

# Migration erstellen
npx supabase migration new <name>

# Remote (Production)
npx supabase db push --linked
npx supabase db pull --linked
```

## WordPress Integration

### Push Flow
1. `app/api/v1/sites/[siteId]/wordpress/push/route.ts` - Push zu WP
2. Sendet Webhooks an WP Plugin
3. WP Plugin speichert Content lokal

### CSS Export
`app/api/v1/sites/[siteId]/export/css/route.ts`
- Scannt: `pages`, `cms_components`, `templates`, `components`
- **WICHTIG**: Header/Footer über `site.global_header_id` / `global_footer_id` verlinkt
- Kompiliert Tailwind v4 CSS für alle gefundenen Klassen

### WP Plugin Pfad
`plugins/wordpress/unicorn-studio-connect/`
- `includes/class-css-manager.php` - CSS Sync
- `includes/class-global-components.php` - Header/Footer
- `themes/unicorn-studio-blank/` - Blank Theme

## Development

```bash
npm run dev      # Dev Server
npm run build    # Production Build
```

## Git & Deploy
- Remote: GitHub
- Auto-Deploy via Vercel auf Push zu `main`
- Nach Push: WordPress "CSS aktualisieren" klicken

## Häufige Probleme

### CSS fehlt in WordPress
1. Prüfen ob Klassen im HTML der Components sind
2. Components werden via `global_header_id`/`global_footer_id` geladen
3. CSS neu synchronisieren in WP Admin

### Tailwind Klassen leer
Tailwind v4 generiert manchmal leere Regeln für Opacity-Farben.
Post-Processing oder Fallback in `export/css/route.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Felixzink96)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Felixzink96)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
