## jandt

> You are the same expert agent Porter would get in Cursor IDE for this repo.

# Holdsworth wedding site — agent instructions

You are the same expert agent Porter would get in Cursor IDE for this repo.
Treat every Slack message in `#jandt-edits` as a direct prompt from Porter or Jayden.
Do NOT take shortcuts. Explore the codebase, understand context, then make the correct change.

## Repo

`pwslcc24-hash/JandT` — Holdsworth wedding site (React + Vite + Base44).
Live URL: jayden-and-taylor.base44.app

## How this site actually works (read before editing)

VISITORS see content from Base44 **SiteContent** entity (CMS JSON), NOT from git fallbacks alone.
Git fallbacks in `src/cms/seed/defaultSite.ts` and `src/config/wedding.js` are merged in but
**published SiteContent wins** for any block that already exists.

Therefore for ANY text/copy/page content change you MUST:

1. Edit the correct source in git (see below)
2. Commit + push to `main`
3. Live content syncs automatically — GitHub Actions runs `publish-sync` when `defaultSite.ts` or `wedding.js` changes on `main`. You do not need to run it manually.

For code/component/style/behavior changes: git push is enough; Porter still **Publish on Base44.com** for deploy.

## Site map — where content lives

### Pages & routes

- `/` home — WeddingLanding.jsx
- `/info` Wedding Info (agenda rich HTML)
- `/story` Our Story
- `/photos` photo albums
- `/registry` Venmo / Registry
- `/studio` PIN 3690 — visual editor (Porter only)

### CMS document structure (SiteDocument in `src/cms/types.ts`)

Pages → sections → blocks. Key slugs:

**Home (`pageSlug: home`)**

- hero: hero-names, hero-lastname, hero-date, hero-video (url, mediaType)
- explore: explore-label, explore-cards (JSON nav items)
- banner: banner-eyebrow, banner-suffix

**Content pages** (info, story, registry): sectionKey `content`

- title (text block)
- body (rich_text — HTML for Tiptap/EditableRichText)

**Story**: also sectionKey `media`, block media-stack (JSON array of images/videos)

**Photos**: photo-albums section with photo-albums block (JSON albums)

### Key files

| Change type | Files |
|-------------|-------|
| Page copy / registry / story text | `src/cms/seed/defaultSite.ts` |
| Nav labels / fallback copy | `src/config/wedding.js` |
| Component layout / logic | `src/pages/`, `src/components/wedding/` |
| Styles | `src/index.css`, `src/styles/editor.css` |
| CMS editor behavior | `src/cms/` |
| AI editor ops reference | `src/cms/api/aiEditor.ts`, `src/cms/ai/` |
| Media upload | Base44 UploadFile via `src/cms/api/base44Media.ts` — never embed huge base64 in JSON |
| Live publish API | `src/cms/api/publish.ts`, `scripts/publish-sync.ts` |

## How to work (same as Cursor IDE session)

**Quality bar:** Slack requests must get the same polish Porter gets in Cursor IDE — not minimal text dumps.
Before editing, read existing pages (especially `/info` agenda HTML in `src/cms/seed/agendaHtml.ts` and its CSS in `src/styles/canvas-editor.css`).
Match that level: structured layout, typography, spacing, cards, hover states, elegant wedding tone.

### Content vs code — choose the right layer

| Request type | What to do |
|--------------|------------|
| Copy / links / page content | Rich HTML in `defaultSite.ts` (or dedicated `*Html.ts` seed file) + CSS classes — **never** bare `<p>` + plain `<a>` lists |
| Layout / components / styling | Edit React components + `src/index.css` or `src/styles/canvas-editor.css` |
| Both | Do both — e.g. registry cards need HTML structure **and** CSS |

### Steps

1. **Understand the request** — if vague, infer the most likely page/block from context.
2. **Search the repo** — grep/read relevant files before editing. Never guess file paths.
3. **Design first** — ask: "Would Porter be happy with this in the IDE?" If it looks like a plain text paste, upgrade it.
4. **Content edits** — use `{ html: ... }` for rich pages (see `agendaHtml.ts`, `registryHtml.ts`).
   Put reusable HTML in `src/cms/seed/*Html.ts`. Add matching CSS next to agenda styles in `canvas-editor.css`.
   For simple story-style pages, paragraphs are fine. For links, cards, lists, timelines — use structured HTML + CSS.
5. **Code/style edits** — edit the real component/CSS, not just fallbacks.
6. **Hero video/images** — use URL fields in hero-video block; uploads go through Base44 file URLs not data URLs.
7. **Run `npm run build`** — fix all errors before finishing.
8. **Git**: commit with clear message, **push directly to `main`**. NO pull request. NO new branch.
   **Critical:** If you push to a `cursor/*` branch instead of `main`, visitors will NOT see copy changes until it is merged. Always push to `main`.
9. **Reply in Slack** — always end with this preview note (include the URL exactly):
   - What page and what changed (1–2 sentences)
   - Then: "To preview the updated site, use the **Open web** link in this thread — it should take you here: https://app.base44.com/apps/6a2b01575fdcdc3d21540f60/editor/preview"
   - For copy edits, mention publish-sync runs automatically after push. For code changes, note Porter may need to Publish on Base44.com.

## Do NOT

- Dump raw links as plain paragraphs — use card/link patterns like agenda or registry pages
- Only edit wedding.js or defaultSite.ts and expect visitors to see copy changes without pushing to main
- Open PRs or create cursor/* branches
- Embed large base64 images/videos in site JSON
- Commit .env or secrets
- Remove hero/explore/banner on home unless explicitly asked
- Reply "done" without verifying build passes

## Ignore

Casual chat: thanks, ok, lol, emoji-only, "sounds good"

## Examples

"Add Amazon registry link and Venmo @handle on registry page"
→ Read `registryHtml.ts` + `agendaHtml.ts` for design patterns
→ Build styled card layout in `registryHtml.ts`, CSS in `canvas-editor.css`
→ Wire in defaultSite.ts → build → push main → Slack reply

"Add Amazon registry link" (WRONG approach)
→ ❌ Three plain `<p>` tags with raw `<a>` links — too basic

"Make hero countdown label say days until forever"
→ home hero section or banner blocks in defaultSite.ts
→ push main

"Make hero text bigger on mobile"
→ styles on hero blocks OR src/index.css — read EditableText/PropertiesPanel first
→ build → push (Publish on Base44.com for deploy)

---
> Source: [pwslcc24-hash/JandT](https://github.com/pwslcc24-hash/JandT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
