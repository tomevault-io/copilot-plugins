## burs-platform

> - Frontend: Next.js 15 (App Router)

# Burs Platform - Next.js 15 + Supabase

## Proje Yapısı
- Frontend: Next.js 15 (App Router)
- Database: Supabase (PostgreSQL)
- AI Scraper: OpenAI GPT-4o-mini
- Styling: Tailwind CSS

## Dosya Organizasyonu
/app
  /api/scholarships/route.ts - API endpoint (Supabase sorguları)
  /page.tsx - Ana sayfa (burs listesi + filtreler)
  
/components
  /ScholarshipFilters.tsx - Filtre sidebar component

/lib
  /supabase.ts - Supabase client + types

/scripts
  /universal_scraper.py - AI destekli web scraper (OpenAI)
  /tubitak_manual.py - Manuel TÜBİTAK burs verisi

## Database Schema (Supabase)
Table: scholarships
- id (uuid)
- title (text)
- organization (text)
- description (text)
- amount (numeric)
- amount_type (text) - "aylık", "yıllık", "tek seferlik"
- deadline (date)
- type (text) - "akademik", "ihtiyaç", "engelli"
- education_level (text) - "lise", "lisans", "yükseklisans", "doktora"
- application_url (text)
- is_active (boolean)

## API Endpoints
GET /api/scholarships
  Query params: search, type, education_level, min_amount, max_amount, days_left

## Önemli Notlar
- Python scraper için: pip3 install openai beautifulsoup4 supabase
- Environment variables: .env.local (SUPABASE_URL, SUPABASE_ANON_KEY, OPENAI_API_KEY)
- Pagination: 10 burs per page
- Filtreleme: Client-side + server-side

## Teknoloji Stack
- Next.js 15.3.3
- React 19
- TypeScript
- Tailwind CSS
- Supabase 2.0.3
- OpenAI API (GPT-4o-mini)
- Python 3.13

## Geliştirme Komutları
npm run dev - Development server
python3 scripts/universal_scraper.py - AI scraper çalıştır

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/serdarozerman-debug) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
