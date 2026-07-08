## djcylow-react

> De volledige werkwijze voor Claude staat in [`workflow/workflow-CLAUDE.md`](workflow/workflow-CLAUDE.md)

# CLAUDE.md — DJ Cylow Website

De volledige werkwijze voor Claude staat in [`workflow/workflow-CLAUDE.md`](workflow/workflow-CLAUDE.md)
en wordt hieronder automatisch ingeladen. Een mens-vriendelijke versie van dezelfde workflow (voor
handmatige uitvoering door Dave) staat in [`workflow/workflow-HUMAN.md`](workflow/workflow-HUMAN.md).

@workflow/workflow-CLAUDE.md

---

## Project Quick Reference

### Key commands

```bash
npm run dev      # dev server → http://localhost:3000
npm run build    # static export → .next/
npm run lint     # ESLint + TypeScript check
```

### Hulpscripts

| Commando | Script | Wat het doet |
|---|---|---|
| `npm run mix:add` | `scripts/add-mix.js` | Voeg interactief een nieuwe mix toe aan het juiste JSON bestand |
| `npm run images:webp` | `scripts/convert-to-webp.js` | Converteer alle `.jpg` in `public/images/` naar `.webp` en verwijder de originelen |
| `npm run images:webp:dry` | `scripts/convert-to-webp.js --dry-run` | Preview: laat zien welke bestanden geconverteerd zouden worden |

Release-workflow scripts (`scripts/release/`) staan beschreven in `workflow/workflow-CLAUDE.md` → "Scripts".

**Workflow nieuwe mix toevoegen:**
1. `npm run mix:add` — vul alle gegevens in, script genereert afgeleide velden automatisch
2. Afbeeldingen neerzetten in `public/images/{power}/{color}/`
3. `npm run images:webp` — als je `.jpg` afbeeldingen hebt aangeleverd
4. Controleer het JSON bestand in de editor
5. Commit + push via de release workflow (zie `workflow/workflow-CLAUDE.md`)

### Critical constraints

- **Static export**: `output: 'export'` in `next.config.ts` — no server-side rendering, no Next.js API routes. Contact form runs via Netlify Functions.
- **Images unoptimized**: `images: { unoptimized: true }` — required for static export. Do not remove.
- **Bilingual (EN/NL) via next-intl**: the site is fully bilingual. All user-facing strings belong in `messages/en.json` and `messages/nl.json` — never hardcoded in components. Use `useTranslations()` (client) or `getTranslations()` (server). The domain is `djcylow.com`; default locale is `en`.
- **No inline CSS**: do not use `style={{}}` in JSX. All CSS belongs in SCSS files under `src/styles/`. Exception: truly dynamic runtime values (e.g. `backgroundImage: url(${src})`, progress bar width percentages).
- **Tailwind v4 + SCSS**: both are used side by side. SCSS lives in `src/styles/`, Tailwind as utility classes in components.

### Where content lives

| What | Where |
|---|---|
| UI strings (buttons, labels, errors) | `messages/en.json` + `messages/nl.json` |
| Mix metadata & tracklists | `src/data/mixes/[power]-[color].json` |
| Home page text | `src/content/home.ts` |
| Services text | `src/content/diensten.ts` |
| Music Mood Colours text | `src/content/musicmoodcolours.ts` |
| Testimonials | `src/content/referenties.ts` |
| Breakpoints | `src/constants/design.ts` |
| In-progress changes (unreleased) | `CHANGELOG.md` → `[Unreleased]` (bevuld via per-branch entry-bestanden, zie workflow) |

### Audio storage

Active Cloudflare R2 bucket: `https://pub-4fa4c2c1f9a644c4878cba29a7926443.r2.dev/`

---

## Mix JSON Rules (src/data/mixes/)

See `src/data/mixes/README.md` for the full schema spec.

Quick rules:
- `color` field: capitalized (`"Red"`, not `"red"`)
- `date` field: ISO format `"YYYY-MM-DD"` (not empty)
- `description`: unique per mix, 120–160 chars, Dutch, no dashes (`-` or `—`), no artist names (artists go in `top_artists`)
- `tracklist` time format: `"HH:MM:SS"` with leading zeros
- New mixes go at the **top** of the array

---
> Source: [DaveKJohn/djcylow-react](https://github.com/DaveKJohn/djcylow-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
