## theoakvilleguide-com

> A Next.js website for The Oakville Guide — an Oakville, Ontario real estate platform. Reads from the same Supabase backend as the React Native mobile app.

# OakvilleGuideWeb — Next.js Website

## What This Is
A Next.js website for The Oakville Guide — an Oakville, Ontario real estate platform. Reads from the same Supabase backend as the React Native mobile app.

## Tech Stack
- **Framework**: Next.js 16+ (App Router, TypeScript)
- **Styling**: Tailwind CSS with custom Oakville theme colors
- **Backend**: Existing Supabase project (shared with mobile app)
- **Data**: PropTX MLS listings cached in `cached_listings` table
- **Hosting**: Vercel
- **Domain**: theoakvilleguide.com

## Key Supabase Details
- Project ID: ixbpbezcgiayzuknliee
- Region: Canada Central
- Auth: Supabase Auth (email/password + Google OAuth)
- The `cached_listings` table is populated by the `sync-listings` Edge Function every 30 min
- VOW (sold) data requires authentication — do not expose without login
- RLS is enabled on all tables

## Styling Conventions
- Page background: #FAFBFC
- Primary dark: #1A2B3C
- Brand blue: #1A5276
- Text primary: #11181C
- Text muted: #687076
- Borders: #E0E0E0
- Cards: white, 12px border-radius, 1px border
- All defined as Tailwind custom colors (oak-dark, oak-blue, etc.)

## TRREB/PropTX Compliance (REQUIRED)
- Every listing display must show brokerage name (ListOfficeName)
- TRREB disclaimer required in footer on every page with listing data
- IDX search results capped at 100 listings
- VOW data (sold listings) requires user to be logged in
- Bona fide interest notice required for VOW access

## Navigation
- / → Homepage (hero + featured listings + neighbourhoods)
- /listings → Browse/search with filters
- /listing/[id] → Individual listing detail page
- /neighbourhoods → Neighbourhood directory

## Important Rules
- Never expose the Supabase service role key in client-side code
- Always use the anon key with RLS
- PropTX API calls go through Supabase Edge Functions, never directly from the browser
- All listing data comes from cached_listings table, not direct API calls

---
> Source: [costarealty/theoakvilleguide.com](https://github.com/costarealty/theoakvilleguide.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
