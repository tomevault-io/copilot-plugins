## dicefloor-preview

> Dice Floor (dicefloor.com) is a career and information website for casino dealers and maritime crew. It covers job hunting, cruise line work, land casino work, industry news, and will eventually include a job agency placement service.

# Dice Floor — Project Briefing

## What This Is
Dice Floor (dicefloor.com) is a career and information website for casino dealers and maritime crew. It covers job hunting, cruise line work, land casino work, industry news, and will eventually include a job agency placement service.

The site is a Jekyll static site hosted on GitHub Pages at https://github.com/dice-floor/dicefloor.

## Stack
- Jekyll static site
- GitHub Pages hosting
- Fonts: Playfair Display (headings) + Source Sans 3 (body)
- Logo fonts: Cormorant Garamond (italic, "Dice") + DM Sans ("Floor") — do not change these
- Logo colors: "Dice" = #c0392b (red), "Floor" = #1a1a1a (dark)

## Project Structure
- `_posts/` — articles (markdown)
- `_includes/` — header, footer, head partials
- `_layouts/` — page templates
- `_pages/` — standalone pages (registered as a Jekyll collection in `_config.yml`)
- `assets/css/main.css` — all styles, single file
- `index.html` — homepage

## Writing Rules
- **American English spelling only** — recognize, organize, labor, center, etc. Never British.
- Article images currently use `https://picsum.photos/seed/cruise1/800/500` as placeholder — the user will replace images manually later.
- Front matter format for posts:
  ```yaml
  ---
  layout: article
  title: "Title Here"
  date: YYYY-MM-DD
  image: https://picsum.photos/seed/cruise1/800/500
  excerpt: "One sentence summary."
  ---
  ```

## Key Design Decisions
- Row 1 hero on the homepage is a placeholder article. It will become a permanent job agency banner once cruise line/casino partnerships are confirmed. Build it as a standard article slot for now.
- Do not add links to external websites without discussing first.
- Do not add new nav menu items without discussing first — the menu is intentionally minimal.

## Article Categories
The 6 confirmed categories (all articles use exactly one): **Cruise Lines**, **Casinos**, **Job Hunting**, **Career Skills**, **Travel**, **Ports**.

Category audit completed — all post front matter is correct.

Category pages live at `/categories/{slug}/`. To add a new category: create a folder + `index.html` with `layout: category` and `category_name` front matter, then add it to the list in `articles/index.html`.

## What's In Progress
- More articles being written and added to `_posts/`
- Cruise lines page (ocean + river cruise lines) — data being compiled manually
- Ferry operators page — sorted by country, data being compiled manually
- Land casino directory — will be a manual, vetted list (not scraped data)
- Audio/podcast section — will use NotebookLM exports

## Software Products (separate from the website)
Six tools planned under the Dice Floor brand:
1. **Casino Dealer Evaluation** — Flutter desktop/iPad app, in development
2. **Cash Desk Accounting** — waiting for documents before starting
3. **Poker Tournament Director** — planned
4. **Online Poker Player Behaviour Analyser** — design complete (`docs/plans/2026-03-20-poker-behaviour-analyser-design.md`)
5. **Marine Crew Paperwork Verification** — design complete (`docs/plans/2026-03-20-marine-crew-verification-design.md`)
6. **Dealer Rota / Swings** — planned; supervisor live tool, replaces Excel, integrates with Dealer Evaluation

## Scraper Scripts (in `/Users/danielgermanov/Library/Mobile Documents/com~apple~CloudDocs/projects/dice-floor/`)
- `casino_scraper.py` — Wikipedia casino scraper
- `casinocity_scraper.py` — CasinoCity.com scraper (IP blocked, use with care)
- `build_casino_db.py` — parses sitemap.xml into `casinos_sitemap.json`
- `fetch_casino_websites.py` — fetches casino website URLs from sitemap casino pages
- `generate_casino_page.py` — generates Jekyll casino directory page from JSON

## Git
- Remote: https://github.com/dice-floor/dicefloor.git
- Branch: main
- Always commit and push after changes unless told otherwise

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dice-floor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dice-floor)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
