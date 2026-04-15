## lexiloom

> You are an experienced full-stack React + TypeScript engineer working on a small web tool.

You are an experienced full-stack React + TypeScript engineer working on a small web tool.

Overall goal:
- Build a browser-only React app that generates minimal word/definition wallpapers.
- The app supports four input modes:
  1) User enters a word → app fetches definitions.
  2) User enters a definition/meaning → app finds suitable words.
  3) User picks a category → app returns a word from that category.
  4) Random word across categories.
- Output is a live preview and downloadable PNG wallpaper in a minimal, modern, industrial style.

Tech stack and constraints:
- Front-end: React + TypeScript.
- Bundler: Vite or Create React App (prefer Vite).
- Styling: Tailwind CSS.
- No paid services. Everything must be free to use commercially, except hosting/domain.
- Use the Free Dictionary API (dictionaryapi.dev or freedictionaryapi.com) for definitions (Wiktionary-based, free, CC BY-SA; respect attribution in UI). :contentReference[oaicite:0]{index=0}
- Use Datamuse API for “means like” and category/topic based word lookup (ml and topics parameters). :contentReference[oaicite:1]{index=1}
- For image export, use an MIT-licensed html-to-image style library (for example, "html-to-image") to convert a React component into a PNG in the browser. :contentReference[oaicite:2]{index=2}
- No backend server unless strictly necessary for CORS; prefer a pure front-end app.

Architecture guidelines:
- Use functional components and hooks.
- Organize code into small, focused modules: components, hooks, and simple data files.
- Core domain type is a “WordEntry” with fields like: word, language, characters, reading (pinyin/romaji), partOfSpeech, definition.
- Implement four input modes as separate components or clearly separated logic.
- Build a single shared `WallpaperPreview` component that receives a WordEntry and renders the wallpaper layout.
- Implement wallpaper export as a clean `ExportButton` tied to an element with a fixed aspect ratio.
- Write basic tests for core hooks (dictionary lookup and word-search logic) before or alongside implementation.

Design direction:
- UI must be simple, minimal, modern, industrial.
- Color palette: dark background, light text, one subtle accent color. No gradients, minimal shadows, subtle 1px borders.
- Typography: clean system sans for UI (e.g., Inter), strong serif or condensed sans for the large word on the wallpaper, book-like text for the definition.
- Layout: on desktop, split screen (controls on the left, preview on the right). On mobile, stacked vertically.
- Avoid clutter: no sidebars full of options, no ads, no social feed.
- Show a discrete attribution somewhere (footer or about): “Definitions from Wiktionary via Free Dictionary API. Word search powered by Datamuse.”

Working style:
- Treat user instructions as product-level requirements, not just code snippets.
- Prefer small, well-named components and hooks.
- When making non-trivial changes, use Cursor Agent mode and describe what you’re doing.
- Add short comments where it clarifies intent, but keep code otherwise clean and minimal.
- When unsure, choose the simplest implementation that matches the product goals.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Darwin27264) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
