## ignea-labs

> <!-- CLAUDE.md — IGNEA LABS -->

<!-- CLAUDE.md — IGNEA LABS -->
<!-- Claude Code reads this on every task. -->

# Architecture
- Vanilla HTML/CSS/JS static site. No frameworks. Hosted on Vercel.
- Repo: github.com/fede09balto-gif/onda-ai
- i18n: data-i18n attributes, JS translations in i18n.js (~807 keys). Spanish default. localStorage key: ignea_lang
- Future backend: Supabase (placeholder credentials currently)

# Design Tokens (from shared.css)
--bg: #0a0a0c; --accent: #e8352a; --text: rgba(245,244,242,0.95); --border: rgba(255,255,255,0.08);
--ff: 'DM Sans', sans-serif; --fs: 'DM Serif Display', serif; --fm: 'JetBrains Mono', monospace;
Logo: "IGNEA" white + ".LABS" in var(--accent). No teal. Border-radius: 0px everywhere.

# Pages
| Page | Status | Notes |
|------|--------|-------|
| index.html | Working | Hero, stats, process, terminal, CTA |
| diagnostic.html | Working | 4-question intake form (NOT 11-question v3.0) |
| results.html | BROKEN | Expects scoring data diagnostic.js no longer produces |
| thesis.html | Working | "Nuestra Tesis" |
| contact.html | Partial | Formspree ID is placeholder |
| 404.html | Working | |
| ops.html | Partial | CRM/calculator/scraper — Supabase + Claude API placeholders |
| admin.html | REMOVED | Consolidated into ops.html |
| demo.html | Inconsistent | Old logo classes |

# Pre-edit Rules
1. Read existing code FIRST. Build on top — never rewrite from scratch.
2. Every string in BOTH es + en in i18n.js. Never hardcode text.
3. Every page: shared nav, footer, grid-bg.js, lang toggle, responsive.
4. Inputs: dark bg, 1px border, accent on focus, 48px min tap target.
5. Cards for multiple-choice, never native radio/checkbox.
6. Buttons: btn-primary (accent bg) or btn-ghost (border) only.
7. localStorage keys prefixed "ignea_".
8. No console.log in production. No external frameworks.
9. Mobile-first: test 375px, 768px, 1440px.
10. Spanish first, English second. Default: ES.

# Known Bugs
- Lang toggle must restart terminal animation in correct language.
- CTA buttons with → use innerHTML not textContent (data-i18n-btn).
- Range sliders: use 'input' event, not 'change'.
- Formspree: prevent default, show loading, handle errors.

# Commit Format
feat: | fix: | refactor: | chore: | docs:

# Business Context
- AI consultancy targeting SMBs in Nicaragua/Central America.
- Partner: Gatún Labs (Panama, $15-30K). We target $1,500-8,000.
- Low diagnostic scores = HIGH opportunity (frame positively).
- Client-facing pages show savings, NEVER our prices.
- Audience is non-technical, mostly mobile.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fede09balto-gif) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
