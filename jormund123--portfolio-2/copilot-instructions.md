## portfolio-2

> Hisabkitab is Nepal's first independent, automated government accountability tracker. It monitors political commitments made by the government and individual MPs, cross-references them with real-time news data, and ascribes progress scores using a multi-stage AI pipeline.

# Hisabkitab (Pratipakchya) - Technical Overview

Hisabkitab is Nepal's first independent, automated government accountability tracker. It monitors political commitments made by the government and individual MPs, cross-references them with real-time news data, and ascribes progress scores using a multi-stage AI pipeline.

## 🏗 Pipeline Architecture

The "Intelligence" layer is the core of the project, located in the `scripts/` directory. It operates as a 5-stage automated pipeline:

1.  **Ingestion (scrapers/)**: Fetches raw data from RSS feeds and news sites into the `documents` table. Includes deduplication logic to ensure unique coverage.
2.  **Screening (classifier/prefilter.ts)**: Uses lightweight keyword-based matching to find "candidate commitments" for each document, significantly reducing expensive LLM calls.
3.  **Intelligence (classifier/llm.ts & pipeline.ts)**: Sends candidate matches to LLMs (Gemini/Groq) to determine if the news `SUPPORTS` or `REFUTES` specific commitment criteria.
4.  **Evidence Logging**: Valid matches (Confidence > 0.6) are logged into the `evidence` table with AI reasoning and supporting quotes.
5.  **Scoring (scoring/engine.ts)**: Aggregates evidence to calculate 0-100% progress scores and update commitment statuses (In Progress, Completed, Late).

---

## 📂 Directory Structure

### 🧠 Core Scripts (`scripts/`)
- **`classifier/`**: The heart of the AI logic.
    - `pipeline.ts`: The main orchestrator that coordinates filtering and LLM calls.
    - `llm.ts`: Handles communication with Gemini/Groq, manages rate limits and retries.
    - `prefilter.ts`: Fast screening logic using keyword overlap.
- **`scoring/`**:
    - `engine.ts`: Calculates progress percentages based on evidence specificity and weights.
- **`scrapers/`**: Base classes and specific parsers for Nepali news sources.
- **`controversy/`**: Analysis of "controversial" news to generate daily interactive polls.
- **`seed/`**: Contains `commitments.json` and `district_commitments.json` (the source of truth for promises).

### 🎨 Frontend (`app/` & `components/`)
- **`app/hisabkitab/`**: Routing, page logic, and scoped CSS for the tracker dashboard.
  - `layout.tsx`: Sets page background (`#E2DAC8`), applies `rt-root` grain once for the whole page.
  - `tokens.css`: Single source of truth for all design tokens (`--paper`, `--ink`, `--terracotta`, etc.) and `--layout-bg`.
  - `hero-section.css`: Static `.hh-*` styles for HeroSection (blob, chip row, ticker, dropdown, typography).
  - `province-metrics.css`, `commitment-card.css`, `commitment-detail.css`, `commitments-list.css`: Per-component scoped styles.

- **`components/pratipakchya/`**:
    - `HeroSection.tsx`: Landing hero — blue blob left panel, chip row (progress/vote/location/language), countdown, status ticker. Includes `LocationDropdown` sub-component and `PollModal` trigger. Inline styles limited to dynamic values only (progress width, urgency gradient, pending opacity, SVG dashoffset).
    - `CommitmentsList.tsx`: Searchable/filterable list of tracked promises.
    - `CommitmentDetail.tsx`: Evidence timeline and criteria checklist for each promise.
    - `CategoryProgress.tsx`: Bar chart by category with hover popover for sources; dialog modal for category drill-down.
    - `PerformanceChart.tsx`: Historical progress visualization.

### 🛠 Library (`lib/`)
- **`pratipakchya/`**:
    - `queries.ts`: Supabase/PostgreSQL queries used across the app and scripts.
    - `constants.ts`: Sorted district lists and shared metadata.

---

## 🎨 Design System

- **Background**: `#E2DAC8` (`--layout-bg`) applied at layout level. `rt-root` grain overlay applied once on the layout wrapper; `position: relative; zIndex: 1` on children to lift above the `z-index: 0` pseudo-elements.
- **Tokens**: All colours reference CSS custom properties from `tokens.css`. Never hardcode hex values in component styles — use `var(--ink)`, `var(--terracotta)`, etc.
- **Blob blue palette**: `--hero-blob-1` through `--hero-blob-4` (slate-blue range, HeroSection only).
- **Chip row pattern**: Container holds the outer border (`1px solid var(--ink)`); each chip has `border: none` with only `border-right` for internal separators. Language chips use a visible `hh-lang-sep` rule before them.
- **CSS conventions**: Static styles go in scoped `.css` files as `.hh-*` / `.pk-*` classes. Dynamic JS-driven values (widths, conditionally-selected gradients, open/active state backgrounds) stay inline.

---

## ⌨️ Common Commands

### Development
- `npm run dev`: Start Next.js development server.
- `npx shadcn@latest add [component]`: Add UI components.

### Pipeline Operations
- `npx tsx scripts/scrapers/run.ts`: Run news scrapers.
- `npx tsx scripts/classifier/pipeline.ts`: Run AI classification (Link news to commitments).
- `npx tsx scripts/scoring/engine.ts`: Recalculate progress scores.
- `npx tsx scripts/controversy/score.ts`: Generate today's controversy polls.

## 🗄 Database Schema
Managed via **Supabase**. Key tables:
- `commitments`: The promises being tracked.
- `documents`: Raw news articles.
- `evidence`: The link between a document and a commitment, with AI classification stats.
- `polls`: Daily controversial topics generated for user engagement.

---
> Source: [Jormund123/portfolio-2](https://github.com/Jormund123/portfolio-2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
