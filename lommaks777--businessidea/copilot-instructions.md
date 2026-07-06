## businessidea

> You are an expert Full Stack Developer building a tool to mine business ideas from Google Play reviews.

# Project Context: ReviewMiner (Local Version)

You are an expert Full Stack Developer building a tool to mine business ideas from Google Play reviews.
- **Environment:** Node.js (Local/Railway). NO Vercel Serverless limitations (timeouts are irrelevant).
- **Stack:** Next.js 14 (App Router), TypeScript, Tailwind CSS, Shadcn/UI.
- **Database:** SQLite with Prisma ORM (local file `dev.db`).
- **Scraping:** Apify SDK (`apify-client`).
- **Logic:**
  1. User inputs niches (keywords).
  2. System searches apps (10k-500k installs) via Apify.
  3. System scrapes reviews (Spanish language).
  4. System analyzes reviews for "Pain Points" (Negative) and "Feature Requests" (Positive with specific keywords).
  5. Results are saved to SQLite and exported to Google Sheets.

**Coding Standards:**
- Use strict TypeScript.
- Use Server Actions or API Routes with Node.js Streams for long-running processes.
- Handle errors gracefully (don't crash the whole loop if one app fails).
- Use `zod` for validation.

---
> Source: [lommaks777/businessidea](https://github.com/lommaks777/businessidea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
