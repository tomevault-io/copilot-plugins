## pingintripv2

> B2C travel & tour web application targeting travelers and families visiting **Lombok** and **Bali**. The unique selling point is **custom tour building** — travelers pick destinations and we generate a personalized itinerary.

# Travel & Tour Web App — Project Context

## Overview
B2C travel & tour web application targeting travelers and families visiting **Lombok** and **Bali**. The unique selling point is **custom tour building** — travelers pick destinations and we generate a personalized itinerary.

## Brand Philosophy
- **NOT luxury-focused** — Pingintrip is about **authentic experiences, friendship, and local soul**
- Founded by Reza, a local guide who believes great trips are about **connection, stories, and genuine human bonds** — not luxury packages
- Tagline: "Authentic journeys, real friendships, local soul. Since 2013."
- Team: **Reza** (Founder & Lead Guide) and **Galang Ardian** (Co-Founder & Operations)
- All copy/content must reflect this philosophy — avoid words like "luxury", "premium", "exclusive"

## Business Context
- **Model**: Direct selling (no marketplace, no commission)
- **Target**: Individual travelers & families planning Lombok/Bali trips
- **Core Feature**: Custom tour builder — user selects destinations → system generates itinerary with scheduling
- **Payment**: Xendit integration
- **Authentication**: Google OAuth

## Tech Stack
| Layer | Technology |
|-------|-----------|
| Framework | Next.js (App Router) |
| Styling | Tailwind CSS + DaisyUI |
| Language | TypeScript |
| Database | PostgreSQL (orders only) |
| Auth | Google OAuth (NextAuth.js) |
| Payment | Xendit |
| Deployment | Vercel |
| Design | Mobile-first, responsive |

## Architecture Decisions

### Frontend-Heavy MVP
- All tour data (packages, destinations, itineraries, reviews) stored as **JSON constants** in `src/data/` folder
- No backend CRUD for content — all content is static data for MVP
- APIs: **Order API** (Next.js API Routes → PostgreSQL) + **Enquiry API** (Google Sheets + Telegram)

### Enquiry Submission System
When a user submits the enquiry form (`/enquiry`), the API route (`/api/enquiry`) does two things in parallel:
1. **Appends a row to Google Sheets** — 18 columns including destinations, budget, traveler count, Google sign-in status
2. **Sends a Telegram notification** — formatted MarkdownV2 message to a group chat

Required env vars:
- `GOOGLE_SERVICE_ACCOUNT_KEY` — JSON from Google Cloud Service Account
- `GOOGLE_SPREADSHEET_ID` — from spreadsheet URL
- `GOOGLE_SHEET_NAME` — default: "Enquiries"
- `TELEGRAM_BOT_TOKEN` — from BotFather
- `TELEGRAM_CHAT_ID` — group chat ID (negative number)

### Data Strategy
All static data files live in `src/data/`:
```
src/data/
├── destinations.ts    # All destination data (Lombok & Bali)
├── packages.ts        # Pre-built tour packages
├── itineraries.ts     # Sample itineraries & activities
├── reviews.ts         # Customer reviews/testimonials
├── categories.ts      # Tour categories (adventure, culture, beach, etc.)
└── index.ts           # Barrel export
```

### Folder Structure
```
src/
├── app/                    # Next.js App Router pages
│   ├── [locale]/(main)/    # Public pages (with main layout: navbar, footer, floating cart)
│   │   ├── page.tsx        # Landing/Home page
│   │   ├── destinations/   # "Explore & Plan" — browse & build custom itinerary
│   │   ├── about/          # About page (story, philosophy, team, gallery)
│   │   ├── enquiry/        # Enquiry form (trip details + user details → Sheets + Telegram)
│   │   ├── packages/       # "Open Trips" — shared/group tour packages
│   │   ├── booking/        # Booking & checkout flow
│   │   └── layout.tsx      # Main layout (navbar, footer, FloatingCart)
│   ├── api/                # API routes
│   │   ├── enquiry/        # POST: submit enquiry → Google Sheets + Telegram
│   │   ├── orders/         # Order CRUD
│   │   ├── payment/        # Xendit webhook & callbacks
│   │   └── auth/           # NextAuth routes
│   ├── layout.tsx          # Root layout
│   └── globals.css         # Global styles
├── components/             # Reusable UI components
│   ├── ui/                 # Atomic/base components (cards, buttons, modals)
│   │   └── optimized-image.tsx  # ⚠️ WAJIB dipakai untuk semua image
│   ├── sections/           # Page sections (hero, features, testimonials)
│   └── layout/             # Layout components (navbar, footer, FloatingCart)
├── data/                   # Static JSON/TS data constants
├── hooks/                  # Custom React hooks
├── lib/                    # Utilities & configurations
│   ├── db.ts               # Database connection (Prisma/Drizzle)
│   ├── sheets.ts           # Google Sheets API helper (enquiry)
│   ├── telegram.ts         # Telegram Bot API helper (enquiry notifications)
│   ├── xendit.ts           # Xendit client config
│   └── utils.ts            # Helper functions
├── types/                  # TypeScript type definitions
└── styles/                 # Additional style files if needed
```

### Image Optimization System

Project ini menggunakan sistem optimasi image custom. **SEMUA image yang digunakan di project ini WAJIB melalui proses ini.**

#### Workflow Menambahkan Image Baru
1. Siapkan image original (jpg/png/webp)
2. Jalankan converter script:
   ```bash
   npm run convert-image -- ./path/to/image.jpg
   # atau dengan custom nama:
   npm run convert-image -- ./path/to/image.jpg --name hero-banner
   ```
3. Script akan generate folder di `public/assets/<nama>/` berisi:
   - `placeholder.webp` — 20px blur placeholder
   - `sm.webp` — 640px
   - `md.webp` — 1024px
   - `lg.webp` — 1536px
   - `xl.webp` — 1920px
   - `original.webp` — ukuran asli
   - `metadata.json` — info sizes, aspect ratio, dominant color
4. Gunakan `OptimizedImage` component di code:
   ```tsx
   import OptimizedImage from "@/components/ui/optimized-image";

   <OptimizedImage src="/assets/hero-banner" alt="Hero" />
   ```

#### Aturan Image
- **JANGAN** gunakan `<img>` tag biasa atau Next.js `<Image>` untuk asset image project
- **SELALU** gunakan `<OptimizedImage>` component dari `@/components/ui/optimized-image`
- **SELALU** convert image terlebih dahulu dengan `npm run convert-image` sebelum dipakai
- Untuk hero/above-the-fold image, gunakan prop `priority` agar tidak lazy loaded
- Image assets yang sudah di-convert disimpan di `public/assets/`

#### OptimizedImage Quick Reference
```tsx
// Lazy loaded (default)
<OptimizedImage src="/assets/foto" alt="Deskripsi" />

// Priority (hero, above-the-fold)
<OptimizedImage src="/assets/hero" alt="Hero" priority />

// Fixed size
<OptimizedImage src="/assets/thumb" alt="Thumb" width={400} height={300} />

// Fill parent
<OptimizedImage src="/assets/bg" alt="Background" fill objectFit="cover" />
```

## Code Style & Conventions
- **Naming**: PascalCase for components, camelCase for functions/variables, kebab-case for files/folders
- **Components**: Functional components with TypeScript interfaces for props
- **State Management**: React Context + `useState`/`useReducer` (keep it simple for MVP)
- **Imports**: Use `@/` path alias for all imports from `src/`
- **DRY**: Don't repeat yourself — extract shared logic into hooks and utilities
- **Responsiveness**: Always design mobile-first, then add desktop breakpoints
- **Accessibility**: Use semantic HTML, proper ARIA attributes

## Completed Features
1. ✅ **Landing Page** — Hero, featured destinations, popular packages, testimonials, brand story
2. ✅ **Explore & Plan (Destinations)** — Interactive map, browse & filter 78 places, add to custom itinerary via cookies, detail pages per destination (SEO)
3. ✅ **FloatingCart** — Persistent trip cart across pages (in layout), links to enquiry, hidden on enquiry page
4. ✅ **Enquiry Form** — Multi-step form: destinations, dates, travelers, budget (IDR/USD/CNY), user details with Google pre-fill, phone input. Submits to Google Sheets + Telegram
5. ✅ **About Page** — Hero, origin story, philosophy cards, stats, team (Reza + Galang), gallery (placeholder), trust logos (placeholder), CTA
6. ✅ **Authentication** — Google login/signup via NextAuth.js
7. ✅ **i18n** — Full 3-locale support (en, id, cn) across all pages

## TODO — Remaining Pages
1. **FAQ** — Frequently asked questions page
2. **Booking Conditions** — Terms for bookings
3. **Privacy Policy** — Data privacy page
4. **Travel Insurance** — Insurance information
5. **Open Trips (Tour Packages)** — ⚠️ **Paling berat** — shared/group trip listing + detail + connect to enquiry for private trips
6. **Terms & Conditions** — Legal terms page

## Navbar Terminology
- **"Explore & Plan"** (not "Destinations") — browsing + custom itinerary building
- **"Open Trips"** (not "Tour Packages") — shared/group trips; private trips go via enquiry system
- **"Custom Tour"** — links to destinations/explore page
- **"About"** — about page

## Important Rules
- All data constants MUST be in `src/data/` — never scatter data across components
- Mobile-first: always start with mobile layout, enhance for larger screens
- Use DaisyUI component classes first, custom Tailwind only when needed
- Keep API routes minimal — only order-related + enquiry endpoints for MVP
- Every component must be typed with TypeScript — no `any` types
- No testing in MVP phase — focus on shipping features
- **⚠️ SEMUA image WAJIB menggunakan `OptimizedImage` component** — JANGAN pakai `<img>` biasa atau `next/image`
- **⚠️ Image baru WAJIB dikonversi dulu** dengan `npm run convert-image` sebelum dipakai di component
- Image assets yang sudah di-convert disimpan di `public/assets/<nama>/`
- **⚠️ Brand voice: authentic, friendship, local soul** — JANGAN pakai kata "luxury", "premium", "exclusive"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neuthos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/neuthos)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
