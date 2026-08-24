## kyle-ai-prompt-library

> kyle.meets.ai's prompt & skill library: a mobile-first "link in bio" site for a

# CLAUDE.md — working guide for this repo

kyle.meets.ai's prompt & skill library: a mobile-first "link in bio" site for a
non-technical TikTok audience. This file is the process of record. Follow it.

## What this is

- **Static HTML, zero build, zero dependencies.** The files *are* the site. No
  framework, no bundler, no CI. Deploy is **manual**: the owner re-uploads the
  folder to Cloudflare Pages / Netlify (drag-and-drop). Merging a PR does **not**
  deploy — always remind the owner of the manual re-upload step.
- **Structure:** `index.html` is the hub. Every prompt and every skill is its own
  self-contained `.html` file at the repo root, with inline `<style>` and a small
  inline copy-button `<script>`. Each detail page links back with
  `<a href="index.html" class="back-link">← all prompts</a>`.
- **Audience is non-technical.** Optimize for "tap · copy · paste." Never make them
  use GitHub, a terminal, or anything technical to get value.

## Branding tokens (reuse, never reinvent)

- Accent orange `#ff6b2c`, background `#fafaf7`, text `#111`, cards white with
  `1px solid #eee`, radius `14px`, system font stack, `:active { transform: scale(0.98) }`.
- Tool-tag colors: design `#fff0e6/#d1590f`, agent `#f0ecfb/#6d4bd1`,
  chatgpt `#e7f6ee/#1d8a5b`, connector `#e8f1fc/#2668b8`, claude `#eef0f2/#566072`,
  skill `#fdeef2/#c0356b` (rose).

## Hub mechanics (`index.html`)

- Prompts/skills are hardcoded `<a class="prompt-card">` blocks inside `.prompts-list`.
- **Filter pills** are a centered pyramid: `All` alone on top (followed by a
  `<span class="pill-break">`), then topic pills wrap beneath. Order: Skills, Career,
  Design, Content & Social, Health & Life, Money, Agents & Automation.
- Each card carries `data-topic` (which pill) and `data-tool` (which colored tag),
  plus optional `data-new` for a NEW badge. First child of each card is
  `<div class="card-meta">` holding the tool-tag span (+ NEW badge).
- **Pill counts are computed in JS from the cards** — never hand-edit them. The JS
  also hides a pill's count when it is 0, and shows the `.no-results` message when a
  filter matches nothing. Progressive enhancement: with JS off, all cards show.
- Valid `data-topic`: `skills`, `career`, `design`, `content`, `health`, `money`, `agents`.
- Valid `data-tool`: `claude`, `design`, `agent`, `connector`, `chatgpt`, `skill`.

## Adding a PROMPT

Duplicate an existing prompt page (e.g. `resume-optimizer.html`), rename, update the
`<title>`, headline, description, how-to steps, and the text in
`<div class="prompt-box" id="prompt-text">`. Then add a card to `index.html` with the
right `data-topic` / `data-tool` and a `card-meta` row. (Also documented in `README.md`.)

## Adding a SKILL (the important one — do it exactly this way)

Decided with the owner and proven with the Humanizer skill (`humanizer.md`,
`humanizer-skill.html`). Skills are **hosted on this site**, not linked out.

1. **Host the file yourself.** Save the skill's markdown into the repo verbatim
   (e.g. `humanizer.md`). **Preserve the original author's credit and license notice
   inside the file** — for MIT/permissive licenses that notice is what makes
   redistribution allowed. Never strip authorship or imply Kyle wrote it.
2. **Build the detail page** by copying `humanizer-skill.html`. It includes:
   - headline + short human description (avoid em dashes — a humanizer page banning
     them that uses them looks bad; good practice everywhere anyway);
   - a **Source credit line**: the `Skill` tag, "By <author> · <license> ·" and a
     `View source on GitHub →` link to the author's repo;
   - a plain-language **"How to use it"** guide: copy → open Claude → paste the skill
     → paste/attach your text → ask it to run. Plus a note: to keep it permanently,
     upload the `.md` under Claude's Skills (Capabilities) settings;
   - the skill text in a `.prompt-box` copy box, generated escaped from the `.md` so
     the on-page text always matches the file;
   - a `.button-row` with **Copy skill** (black) + **Download** (outline, `href` →
     the `.md` with a `download` attribute).
3. **Add the card** to `index.html`: `data-topic="skills"`, `data-tool="skill"`
   (rose `Skill` tag), `data-new`, placed at the top of the list. The Skills pill
   count updates itself.
4. **Credit rule:** host the file + link back to the author's GitHub as the source.
   GitHub is the credit link, never the front door. Many skill repos are multi-file
   (packaging/scripts), but the self-contained `SKILL.md` is all a user needs —
   hosting that one file with Copy + Download is the complete deliverable.

## Git / PR workflow

- Develop on branch **`claude/prompt-library-organization-7pc5j0`**.
- **One change = one PR into `main`.** Don't stack. The owner reviews and merges.
- **After a PR merges, restart the branch from the updated main before the next
  change:** `git fetch origin main && git checkout -B claude/prompt-library-organization-7pc5j0 origin/main`.
  A `git push --force-with-lease` is fine then (the branch holds only merged history).
  This keeps every PR a clean, single-purpose diff.
- Only open a PR when the owner asks; the established rhythm is: build → show a
  preview → open a PR they can merge.
- Commit messages: clear and descriptive; do not put model identifiers in them.

## Verify before shipping

- Render with the pre-installed headless Chromium at ~480px width:
  `/opt/pw-browsers/chromium-1194/chrome-linux/chrome --headless --no-sandbox --disable-gpu --screenshot=out.png "file://$PWD/index.html"`.
  Check counts, filtering (inject a `load` handler that clicks a pill to test a
  filtered view), tool tags, and NEW badges. Render new detail pages too.
- **Publish an interactive preview as an Artifact** so the owner can tap it. Make it
  self-contained: inline `profile.jpg` as a data URI for the hub; for a skill page,
  inline the `.md` into the Download link as a `data:text/markdown;base64,...` URI so
  Download works in the preview.
- Confirm `git diff --name-only` touches only what you intended; never edit unrelated
  prompt pages.
- Remind the owner to re-upload to Cloudflare/Netlify to go live.

---
> Source: [kylemeetsai/kyle-ai-prompt-library](https://github.com/kylemeetsai/kyle-ai-prompt-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
