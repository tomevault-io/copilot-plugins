## torbjorn-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **General rules:** See `C:\Dev\spitakolus\CLAUDE.md` for shared standards (documentation, assets, Meta Ads, infrastructure).

## Repository Identity

**This is torbjorn-ai** - Personal portfolio/documentation website for Torbjörn Sandblad.

- **Purpose:** Showcase AI-integrated work processes
- **Tech stack:** Next.js 15, TypeScript, Tailwind CSS, Keystatic
- **Production:** https://www.torbjornsai.site
- **Tech stack:** Next.js 15, TypeScript, Tailwind CSS, Keystatic CMS

**Related repos:**
- `spitakolus` → shared documentation (general rules)
- `flocken-website` → flocken.info product
- `nastahem` → nastahem.com product

---

## Commands

```bash
npm install          # Install dependencies
npm run dev          # Dev server (localhost:3000)
npm run build        # Production build
npm run lint         # ESLint
```

**Keystatic Editor:** Visit `localhost:3000/keystatic` to access the visual content editor.

---

## Project Structure

```
torbjorn-ai/
├── app/
│   ├── page.tsx              # Start / Om arbetet
│   ├── arbete/
│   │   ├── page.tsx          # Process list
│   │   └── [slug]/page.tsx   # Individual process
│   ├── riktning/page.tsx     # Riktning (static)
│   ├── keystatic/            # Keystatic admin UI
│   └── api/keystatic/        # Keystatic API routes
├── components/
│   ├── Header.tsx
│   └── Footer.tsx
├── content/
│   └── processer/            # MDX/Markdoc process files
├── public/assets/images/     # Images and placeholders
├── keystatic.config.ts       # Content schema
└── tailwind.config.js        # Design tokens
```

---

## Content Structure

### Process files (content/processer/)

Each process follows this exact structure:
1. Sammanfattning (in frontmatter)
2. Syfte & avgränsning
3. AI-first angreppssätt
4. Verktyg & system
5. Resultat / nuläge
6. Lärdomar
7. Fortsättning (with status)
8. Uppdateringar (timestamped)

### Ämneskluster (categories)

- `tracking-data-analys` – Tracking, data & analys
- `content-kreativ-produktion` – Content & kreativ produktion
- `automation-arbetsfloden` – Automation & arbetsflöden
- `beslutsstod-prioritering` – Beslutsstöd & prioritering
- `foretagsbyggande-ai` – Företagsbyggande med AI

### Status values

- `aktiv` – Currently being worked on
- `parkerad` – On hold, not abandoned
- `avslutad` – Completed

---

## Design System

### Colors (Tailwind)

```css
base: #FAFAF8         /* Warm off-white background */
text: #1A1A1A         /* Primary text */
text-muted: #6B7280   /* Secondary text */
accent: #3D6B5C       /* Primary accent (muted green) */
accent-hover: #4A7C6F /* Hover state */
border: #E5E7EB       /* Borders */
```

### Typography

- **Display/Headlines:** Georgia (serif) – personality, substance
- **Body:** Inter (sans-serif) – clean, readable
- **Code:** JetBrains Mono – technical elements

### Design Principles

- Text first, always
- Generous whitespace
- Portfolio feel, wiki simplicity
- Same template everywhere
- Professional, human, trustworthy

---

## Visuell stil och bildproduktion

Tre dokument styr bildarbetet:

- **`docs/visuell-stil.md`** — det estetiska språket: palett, halftone, referensbibliotek (Ref A, Ref B, Miljö 1-4 i `public/assets/references/`), filnamnskonventioner.
- **`docs/bildgenerering.md`** — produktionsflöde och Nano Banana-lärdomar: scen-prompt-mall, reference-usage-rules-block, brand-filter-strategier, edit vs fresh-generate, "exactly ONE"-mönstret.

Hero-bilder för artiklar sparas i `public/assets/images/` med filnamnet `{artikel-slug}-hero.jpg` — detekteras automatiskt av artikel- och listsidorna.

Referensbiblioteket i `public/assets/references/` används i alla framtida bildgenereringar. Bifoga alltid Ref A för karaktärslikhet + en miljöreferens för stil. Lägg till Ref B om scenen har en pratbubbla.

---

## Skrivguide

Den aktiva skrivguiden ligger alltid i `docs/skrivguide.md`. Det är ett levande dokument – ingen versionsarkivering, git-historiken är versionshanteringen.

**Uppdatera guiden:**
1. Redigera `docs/skrivguide.md` direkt
2. Commita med `git commit -m "Docs: update skrivguide – [kort beskrivning av ändringen]"`
3. Pusha till båda remotes

**Guiden täcker:** ton & röst, artikelstruktur, tooltip-syntax, NextThreshold-komponenten, formatregler och titelprincipen.

---

## Deployment

### Setup (hur det är konfigurerat)

Vercel Hobby-planen tillåter bara ett GitHub-konto per team. Torbjörns Vercel-team
är kopplat till **RaquelSandblad**-kontot. Därför deployas via ett privat spegelrepo:

- `origin` → `tbinho/torbjorn-ai` (backup/källkod)
- `raquel` → `RaquelSandblad/torbjorn-ai` (Vercel deployas härifrån)

Vercel är anslutet till `RaquelSandblad/torbjorn-ai` och deployas automatiskt vid push.

### Deploy-kommando

```bash
git add .
git commit -m "Description"
git push origin main   # backup
git push raquel main   # triggar Vercel-deploy
```

### Viktigt: commit-author MÅSTE vara RaquelSandblad

Vercel Hobby-planen tillåter inte collaborators på privata repos. Det betyder
att commits med en annan author (t.ex. `tbinho`) blockas av Vercel med:

> "Deployment blocked: The commit author did not have contributing access to
> the project on Vercel."

Repo-lokal git-config är därför satt till Raquel:

```bash
git config user.name "RaquelSandblad"
git config user.email "raquel.sandblad@hotmail.com"
```

Verifiera med `git config user.name` innan första committen i en ny session.
Om någon commit råkat bli författad av annan person, korrigera med:

```bash
git rebase <senast-fungerande-commit> --exec \
  'git commit --amend --reset-author --no-edit --allow-empty'
git push --force <remote> main
```

### Credentials för deploy (non-interactive)

Bakgrundsterminaler kan inte visa GUI-credential-dialogen. Använd inbäddade tokens direkt i URL:en:

```powershell
# Hämta cachade tokens från Windows Credential Manager:
"protocol=https`nhost=github.com`nusername=tbinho`n" | git credential fill
"protocol=https`nhost=github.com`nusername=RaquelSandblad`n" | git credential fill

# Pusha med inbäddad token:
git push "https://tbinho:<TOKEN>@github.com/tbinho/torbjorn-ai.git" main
git push "https://RaquelSandblad:<TOKEN>@github.com/RaquelSandblad/torbjorn-ai.git" main
```

Tokens cachas i Windows Credential Manager och hämtas med `git credential fill` ovan.

### Domän & DNS

- **Domän:** `torbjornsai.site` (registrerad hos Loopia)
- **Namnservrar:** Bytta till Vercels (`ns1.vercel-dns.com`, `ns2.vercel-dns.com`)
- **DNS hanteras av Vercel** (inte Loopia DNS-editor)
- **Production URL:** https://www.torbjornsai.site

---

## Future: Nano Banana Integration

Planned integration for AI image generation. API endpoint TBD.

---

## Company Info

- **Company:** Spitakolus AB (Org.nr: 559554-6101)
- **Email:** support@spitakolus.com

---
> Source: [tbinho/torbjorn-ai](https://github.com/tbinho/torbjorn-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
