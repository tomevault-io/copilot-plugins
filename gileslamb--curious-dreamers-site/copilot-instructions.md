## curious-dreamers-site

> Handoff for the Curious Dreamers studio site (`index.html`). Read this first before editing.

# CLAUDE.md

Handoff for the Curious Dreamers studio site (`index.html`). Read this first before editing.

## What this site is

The Curious Dreamers studio site. A single-page, single-file HTML site that introduces the studio (Giles Lamb and Sacha Kyle) and showcases their projects. Each project links out to its own Living Bible site or external page. This is a sibling to `wndrklub.vercel.app` (which has its own repo and `CLAUDE.md`), not the same site.

The page is one long scroll: hero, manifesto, projects grid, about, contact, plus a shared modal that opens when a project card is clicked.

## File layout

This project is **a single `index.html` file**, plus an `img/` folder (images referenced relatively) and audio assets served from Cloudflare R2. There is no build step. No bundler. No framework. Plain HTML, CSS in a `<style>` block, vanilla JS in `<script>` blocks. Edit the file directly, commit, push, Vercel auto-deploys.

This matches Giles's standing preference: no build tools or frameworks on static projects.

## Hosting and deploy

- **Repo**: GitHub (the studio site, separate from `gileslamb/wndrklub`)
- **Deploy**: Vercel, auto-deploy on push to main
- **Audio CDN**: Cloudflare R2 at `https://pub-2b9a4a895c154bf7b7a368d590f2a3af.r2.dev/audio/<folder>/<filename>.mp3`. The `R2` base constant is set near the audio config. Folder names are case-sensitive (`Hushabye`, `WndrKlub`, `teatrino`, `leviathan`, `en`, `distance-to-the-moon`, `dreamland`).
- **Video**: Cloudflare Stream. Hero and project cards use `customer-3aa0vwfgpylhsylu.cloudflarestream.com` IDs. HLS playback uses `hls.js` loaded via CDN at the top of the file.

## Page structure (sections in order)

1. `#hero` — Animated smoke canvas, film tape, intro
2. `#manifesto` — Trumpet button that plays a surprise sound, manifesto copy
3. `#projects` — Project grid. Each card opens the modal
4. `#about` — Studio bio (Curious Dreamers)
5. `#contact` — Contact info
6. Modal (`#modalOverlay`) — Shared modal for all project deep-dives

## Project data model

All project copy lives in two JS objects near the bottom of the file:

- `P` — the main object keyed by project slug. Each entry has `tag`, `title`, `sub`, `body` (HTML string), and `creds` (array of credit chips).
- `SITE_URLS` — outbound link per project to its Living Bible or external site.
- `SITE_LABELS` — button label for the outbound link.
- `PROJECT_TRACKS` (via `makeTracks(folder, files)`) — audio tracks per project, served from R2.

**Project slugs in use**: `hushabye`, `wndrklub`, `teatrino`, `leviathan`, `en` (this is Shãh-rak-uen', kept as `en` for legacy compatibility, do not rename the key), `rebel`, `dttm`, `dreamland`, `sufi`.

To edit a project's modal copy, find its slug in `P` and edit `body`. To change the outbound link, edit `SITE_URLS[slug]`. To change the button text, edit `SITE_LABELS[slug]`. To change the card teaser (in the grid), search the `#projects` section in markup.

## Recent change: Shãh-rak-uen' rename

The project formerly displayed as "Sha Raku En" is now displayed as **Shãh-rak-uen'** (phonetic rendering, with tilde-a and trailing apostrophe). The data key remains `en` everywhere in code. Four places were touched:

1. The card `<h3 class="proj-title">` in the projects grid markup
2. `SITE_LABELS.en` (button label)
3. `P.en.title` and the `<em>Shãh-rak-uen'</em>` reference inside `P.en.body`
4. An internal JS comment ("Start animated thread if it's...")

Also removed from `P.en.body`: the sentence "A twelve-minute looping piece, evolving in real time, drawn entirely from the garden's own material." This was removed because the duration and approach are not yet locked. The credit chip list still reads "12-15 min Generative Loop" — flag this with Giles if it should also be removed for consistency.

The card subtitle still reads "— Reawakening" with an em dash. Giles's standing preference is **no em dashes in copy**, but this one predates the current pass and is a stylistic separator inside a title rather than running prose. Worth raising next time the modal is touched.

## House style rules (apply to all copy edits)

- **No em dashes** anywhere in new copy. Use commas, full stops, or ellipses. The existing site has a few em dashes from earlier passes; replace them when you're already in that area, do not do a sweeping rewrite without asking.
- **No AI prose patterns**: no hollow intensifiers ("truly", "deeply", "incredibly"), no formulaic transitions ("Moreover," "Furthermore,"), no sentences that mean nothing on a second read.
- **British English** throughout. Glasgow-based studio.
- Plain HTML/CSS/JS. No build tools.
- Iterative changes. Confirm with Giles before large structural edits.

## Editing workflow

Two routes:

- **Local**: VS Code with Claude Code, working from the repo. `CLAUDE.md` (this file) lives in the repo root and is read by Claude Code automatically.
- **Conversational**: Giles uploads `index.html` here, asks for an edit, downloads the result, commits it.

When editing here in chat: copy the uploaded file to working dir, use `str_replace` for surgical edits, output the file via `present_files`. Do not rewrite the whole file unless asked.

## Things to know about the studio (context for tone)

Curious Dreamers is Giles Lamb and Sacha Kyle. Glasgow-based. Practice spans film, television, animation, sound, and immersive work. Sacha is Creative Director & Producer, leading visually and directorially (hand-drawn mixed-media collage). Giles is Composer & Sound Artist, leading sound and music. Key institutional relationships: Screen Scotland, Education Scotland, Animation Scotland.

Active projects with their own Living Bible sites:
- **WndrKlub** — primary project, screen-education for ages 3–7, separate repo at `gileslamb/wndrklub`
- **Teatrino** — secondary, surreal comedy for kids, more 3D and commercial
- **Shãh-rak-uen' (Reawakening)** — proposed 360° immersive installation at the Japanese Garden at Cowden
- **The Last Leviathan** — animated film / immersive piece with Mairi Campbell and Alex South
- **Distance to the Moon** — completed multi-award-winning stop motion

## Known gaps and next steps

- Confirm whether the "12-15 min Generative Loop" credit chip on the Shãh-rak-uen' modal should be removed alongside the line that was just deleted from the body
- Em dash in the Shãh-rak-uen' card subtitle ("— Reawakening") could become an en dash, comma, or be removed
- The site is loosely a "living artefact" and should evolve as projects move forward, not be treated as finished

## Useful greps

- All project entries: `grep -n "title:" index.html`
- Outbound links: search for `SITE_URLS`
- Audio config: search for `PROJECT_TRACKS` and `makeTracks`
- Section anchors: `grep -n '<section id=' index.html`

---
> Source: [gileslamb/curious-dreamers-site](https://github.com/gileslamb/curious-dreamers-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
