## erp-spiritoalchemico

> - Frontend: SvelteKit 2 + Svelte 5 + TailwindCSS 4

# SpiritoAlchemico ERP - Cursor Rules

## Stack
- Frontend: SvelteKit 2 + Svelte 5 + TailwindCSS 4
- Backend/DB: PocketBase (SQLite embedded)
- SDK: pocketbase (npm package)
- Language: TypeScript strict
- Deployment: VPS Linux, Nginx reverse proxy

## Architecture
- PocketBase runs on the server at http://127.0.0.1:8090
- Frontend communicates via PocketBase JS SDK
- All auth goes through PocketBase Auth collections
- Use SvelteKit server-side load functions for data fetching
- Use SvelteKit form actions for mutations

## Code Style
- Use TypeScript strict mode
- Use early returns for readability
- Prefix event handlers with "handle" (handleClick, handleSubmit)
- Components in PascalCase: ProductCard.svelte
- Use $lib/components for shared components
- Use $lib/types for TypeScript interfaces
- Use $lib/utils for utility functions
- Use $lib/pocketbase.ts for PB client initialization

## PocketBase Conventions
- Collection names: lowercase, snake_case (e.g., order_items)
- Relation fields: use expand for nested data
- Always handle PocketBase errors with try/catch
- Use realtime subscriptions only where truly needed

## Design System (MANDATORY - follow strictly)

### Colors
- Page background: gradient from #E8E4DE to #F5F0E8 with warm yellow hint #FFF8E7
- Cards: white #FFFFFF, border-radius: 24px, box-shadow: 0 1px 3px rgba(0,0,0,0.04), no visible borders
- Accent cards: soft yellow #FFF3CD or #FFFDE7 background
- Dark cards: #1A1A1A / #2D2D2D background with white text (for contrast widgets like notifications)
- Primary accent: yellow #F5D547 / #FFD60A (active pills, badges, progress bars, secondary buttons)
- Primary action: black #1A1A1A (primary buttons, active navbar pill)
- Text primary: #1A1A1A
- Text secondary: #6B7280
- Danger: soft red for alerts
- Row hover in tables: #FFFDE7

### Typography
- Font family: Inter (import from Google Fonts)
- Page titles: 28-36px, font-weight 600-700, #1A1A1A
- KPI numbers: 36-48px, font-weight 700, #1A1A1A
- Body text: 14-16px, font-weight 400-500
- Secondary text: 14px, #6B7280

### Layout
- NO sidebar navigation. Use a TOP horizontal navbar with pill/tab navigation
- Active tab: black background (#1A1A1A) with white text, border-radius: 20px
- Inactive tab: transparent with gray text
- Page content uses CSS Grid "bento box" style: cards of varying heights, max 3-4 columns
- Generous spacing: 16-24px gap between cards, 24-32px internal card padding
- Mobile: stack to single column

### Components
- Cards: always rounded-3xl (24px), subtle shadow, no borders
- Buttons primary: bg-[#1A1A1A] text-white rounded-2xl px-6 py-3
- Buttons secondary: bg-[#F5D547] text-[#1A1A1A] rounded-2xl px-6 py-3
- Badges/pills: rounded-full, small font, colored background
- Tables: no grid borders, row hover #FFFDE7, checkbox left, status as colored pills
- Charts: minimal style, rounded bar tops, thin line charts, soft donut segments
- Avatars: circular 32-40px with subtle border
- Inputs: rounded-2xl, subtle border, focus ring yellow
- Modals/drawers: rounded-3xl with backdrop blur

### General
- TailwindCSS for all styling, no <style> blocks
- Mobile-first responsive design
- Italian language for all UI labels
- Smooth transitions: transition-all duration-200
- Icons: use Lucide icons (lucide-svelte package)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VittorinoMarco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
