## chase-ai-carousels-template

> This file auto-loads every session. You (Claude Code) read this to know how to drive the carousel-building pipeline end-to-end. The user typed `claude` and is expecting you to take a one-sentence intent and produce a shippable carousel.

# Chase AI Carousels — Claude Code Operator Manual

This file auto-loads every session. You (Claude Code) read this to know how to drive the carousel-building pipeline end-to-end. The user typed `claude` and is expecting you to take a one-sentence intent and produce a shippable carousel.

## What this repo is

A template library + production pipeline for Instagram/TikTok carousels. The user is here because they watched a YouTube video and want to build carousels the same way. Your job is to walk them through it — they may have never used this repo before.

## First-time-in-repo behavior

On the user's first message in a fresh repo:

1. **Preflight check** — verify the environment is set up. Run these in parallel:
   - `node --version` → must be ≥ 18
   - `npx playwright --version` → confirms Playwright installed
   - `higgsfield --version` → confirms Higgsfield CLI installed (also accepts `hf` / `higgs` aliases — but `hf` collides with Hugging Face CLI on many systems, prefer `higgsfield`)
   - `higgsfield auth token` → confirms user is authed (prints token if authed, errors if not — `auth status` does NOT exist)
   - (Optional) `ls ~/.claude/skills/higgsfield-generate/SKILL.md` → if present, the official Higgsfield skills bundle is installed (Marketing Studio, Virality Predictor, Soul ID, etc). Not required for cover gen — we call the CLI directly — but nice for adjacent jobs.
2. **If anything fails**, walk them through fix:
   - Node missing → link to <https://nodejs.org/>
   - Playwright missing → tell them to run `npm run setup`
   - Higgsfield CLI missing → see § Higgsfield install (platform-specific) below
   - Higgsfield CLI not authed → `higgsfield auth login` (opens browser, ~5 sec device login)
   - Higgsfield skills bundle missing → optional; only suggest if user asks about Marketing Studio / Virality Predictor / Soul ID. Install: `npx skills add higgsfield-ai/skills`
3. **If all checks pass**, ask what they want to build. Then drive.

### Higgsfield install (platform-specific)

**macOS / Linux:**
```bash
curl -fsSL https://raw.githubusercontent.com/higgsfield-ai/cli/main/install.sh | sh
# OR
brew install higgsfield-ai/tap/higgsfield
# OR
npm install -g @higgsfield/cli
```

**Windows:** `npm install -g @higgsfield/cli` is broken (postinstall tar can't handle Windows paths). Install manually from GitHub releases:
1. Download `hf_<version>_windows_amd64.tar.gz` from <https://github.com/higgsfield-ai/cli/releases/latest>
2. Extract `hf.exe` to a permanent folder (e.g. `C:\Users\<you>\bin\higgsfield\`)
3. Copy `hf.exe` to `higgsfield.exe` in the same folder (so the `higgsfield` command name works — `hf` collides with Hugging Face CLI)
4. Add the folder to user PATH via PowerShell: `[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path", "User") + ";C:\Users\<you>\bin\higgsfield", "User")`
5. Open a NEW terminal, then `higgsfield auth login`

If user already in the repo before (carousels folder exists with previous work), skip the preflight — they're past onboarding.

## When the user asks for a new carousel

Trigger phrases: "make a carousel about X", "build me a carousel for Y", "carousel on Z", "/make-carousel".

### Step 1 — Pick a template

Two templates ship in this repo:

| Template | Use when |
|---|---|
| `01-graph-paper-cream` | Top-N lists, tool roundups, framework comparisons. Cream bg + graph paper grid + terracotta accents. |
| `02-terracotta-zine` | 3-step explainers, process tutorials, concept→example pairs. Full-bleed terracotta texture + zine collage. |

Propose ONE based on user's topic. Give the one-line description. Confirm before proceeding.

If neither fits, ask before forking a new template. New templates are a meaningful undertaking — don't quietly invent one.

### Step 2 — Create the dated folder

```
carousels/YYYY-MM-DD-<slug>/
├── assets/
└── (slides.html copied from template)
```

Use **today's date**, never a past date. Use a short kebab-case slug (e.g. `2026-05-22-claude-code-workflows`).

```bash
mkdir -p carousels/<id>/assets
cp templates/<template>/slides.html carousels/<id>/
```

If template has bundled assets (e.g. `bg-zine.png`), also copy:
```bash
cp templates/<template>/assets/*.png carousels/<id>/assets/
```

### Step 3 — Generate the cover (Higgsfield CLI, direct)

The cover is the **only** AI-gen part. Body slides NEVER use AI gen.

**`gpt_image_2` model params** (locked for our cover use — verified via `higgsfield model get gpt_image_2`):
- `prompt` (string, required) — flatten from our JSON prompt template
- `aspect_ratio` — **3:4** (max Higgsfield supports; we crop to 4:5 after)
- `batch_size` — **4** (always 4 variants minimum, never one-shot)
- `quality` — **high**
- `resolution` — **2k** (cheap + sharp enough; bump to 4k only for hero-launch covers)
- `--image <path>` — optional reference image; CLI auto-uploads local file paths

**Workflow:**

1. **Load the prompt template** from `prompts/cover-<style>.json` matching the template:
   - Template 01 → `prompts/cover-marble-statue.json`
   - Template 02 → `prompts/cover-terracotta-zine.json`
2. **Copy + fill** topic-specific fields. Save the per-carousel copy as `carousels/<id>/cover-prompt.json`. Keep `do_not`, composition, `_usage_notes` blocks locked.
3. **Show the user the filled JSON** for approval before burning credits.
4. **Flatten JSON → text prompt.** Walk non-underscore-prefixed keys in order, emit `KEY: value` lines (uppercase the key, replace `_` with space), join with `. `. Skip `_comment`, `_usage_notes`, etc.
5. **(Optional) Cost dry-run** — free:
   ```bash
   higgsfield generate cost gpt_image_2 --prompt "test" --aspect_ratio 3:4 --batch_size 4 --quality high --resolution 2k
   ```
   2k high batch=4 ≈ 7 credits. 4k high batch=4 ≈ 12 credits.
6. **Run the gen** — single CLI call, `--wait` blocks until all variants ready, `--json` returns the full job array:
   ```bash
   higgsfield generate create gpt_image_2 \
     --prompt "<flattened text prompt>" \
     --image <optional-ref-path> \
     --aspect_ratio 3:4 \
     --batch_size 4 \
     --quality high \
     --resolution 2k \
     --wait \
     --wait-timeout 15m \
     --json \
     > carousels/<id>/gen-output.json
   ```
   Drop `--image` if no reference. Add `--image carousels/<id>/assets/ref.png` to lock to a prior cover's geometry.
7. **Parse `result_url`s from the JSON output:**
   ```bash
   node -e "JSON.parse(require('fs').readFileSync('carousels/<id>/gen-output.json','utf-8')).forEach((j,i)=>console.log(i+1, j.result_url))"
   ```
8. **Download each `result_url` to `cover-iterations/<n>.png`:**
   ```bash
   mkdir -p carousels/<id>/cover-iterations
   curl -sL -o carousels/<id>/cover-iterations/variant-1.png "<url-1>"
   curl -sL -o carousels/<id>/cover-iterations/variant-2.png "<url-2>"
   # ... one per variant
   ```
9. **Present all 4 variants** to user. They pick the winner.
10. **Pre-crop 3:4 → 4:5** (Higgsfield max is 3:4, IG native is 4:5):
    ```bash
    node shared/crop-cover.mjs --carousel <id> --input cover-iterations/<winner>.png
    ```
    Writes to `carousels/<id>/cover.png`. Anchors crop to bottom (preserves headline).

If user pushes for more variants: re-run the gen with the same prompt. Don't change the prompt without their say-so.

### Optional: the `higgsfield-generate` skill

If the user has the official Higgsfield skills bundle installed (`npx skills add higgsfield-ai/skills`), the `higgsfield-generate` skill is available as an alternative invocation path. It wraps the same CLI call. For our cover use case, the direct CLI call above is shorter + more legible, so prefer that.

The skill IS useful for adjacent Higgsfield jobs the user may want later — Marketing Studio (UGC ad videos), Virality Predictor (score finished videos for hook strength + retention), Soul ID (train face-faithful gens of a real person). Don't proactively suggest those for cover gen.

### Step 4 — Author body slides

Edit the `slides[]` array at the bottom of `carousels/<id>/slides.html`. One entry per slide. Format depends on template — see `templates/<template>/recipe.md` for the exact schema.

Key fields (Template 01):
```js
{
  name: 'SkillName',
  headlineHTML: '#1: <span class="accent">SkillName</span><br>does the thing',
  layered: true,
  heroBg: 'assets/hero-skill.png',
  repoCard: { owner, name, desc, lang, langColor, stars },
  body: 'Body line ends with terracotta arrow',
  slideIndex: 1,
  defaults: { cardX: 0, cardY: 0, cardW: 600, tilt: -1.5 },
}
```

Key fields (Template 02):
```js
{
  eyebrow: 'STEP 1 / ARCHITECTURE',
  headlineHTML: 'lowercase massive <span class="accent">headline</span>',
  body: 'Body copy with <strong>2-3 bold accent words</strong> per slide.',
  heroImg: 'assets/step-1-diagram.png',
  defaults: { bodyTop: 420, heroBottom: 200, heroWidth: 600, heroTilt: -1.5 },
}
```

### Step 5 — Source body hero assets (NEVER AI gen)

For each body slide, find a real asset:

**Path A — real product asset (preferred when topic has one)**
- GitHub README banner
- GitHub OG card fallback: `https://opengraph.githubassets.com/1/owner/repo`
- Product site hero screenshots
- Trace viewers / inspector screenshots

**Path B — HTML hero composition (preferred for abstract topics)**
- Author `templates/<template>/heroes/<slug>.html` using real product logos (inline SVG), brand-locked dark-cobalt bg, Inter labels
- Render: `node shared/render-heroes.mjs --template <template> --carousel <id>` → outputs `1600×1000` @ 2x retina to `carousels/<id>/assets/hero-<slug>.png`

**If the user proposes AI-gen for a body slide**, push back:
> "HTML will read better — AI typography drifts and value slides need editable copy. Want me to grab a real screenshot or compose an HTML hero instead?"

### Step 6 — Open in browser for visual tweak

```powershell
# Windows
Invoke-Item carousels/<id>/slides.html
```
```bash
# Mac / Linux
open carousels/<id>/slides.html
```

User will:
- Walk slides with `→` / `←`
- Drag heroes/cards to reposition
- Use sliders (⚙ panel top-right) for fine tilt / width / X / Y / size
- Edit headline / body / repo card text inline via tweak panel
- Tweaks save to **localStorage** automatically (per-browser, persists across reload)

### Step 7 — The sync ritual (CRITICAL — don't skip)

localStorage tweaks are **invisible to Playwright**. The bake will produce what's in the source `slides[]` defaults — NOT what the user sees in browser. So:

1. User clicks **📋 Export** button (bottom of ⚙ tweak panel) — JSON copies to clipboard
2. User pastes JSON into chat
3. **You** parse the JSON and update each slide's `defaults: { ... }` in `slides.html` to match
4. Confirm the sync (e.g. "Synced 5 slides. Default `cardW: 720, tilt: -3.2` baked into slide 'foo'.")

If user runs the bake before syncing, the PNGs won't match what they saw. This is the #1 confusion point — explicitly warn them if they ask to bake without having pasted JSON.

### Step 8 — Screenshot feedback (optional)

If a tweak isn't landing right, ask the user to screenshot the browser state and paste it. Then compare to intent + iterate `defaults` until it matches. You can't see the browser yourself — this is how the visual loop closes.

### Step 9 — Bake the final PNGs

```bash
node shared/export.mjs --template <template> --carousel <id>
```

Output: `carousels/<id>/exports/slide-NN-YYYY-MM-DD.png` — 2160×2700 @ 2x retina, 4:5 IG-native. 5-10s for 9 slides.

### Step 10 — Ship handoff

Report:
- Where the final PNGs landed
- The dimensions (2160×2700, 4:5)
- Suggested IG/TikTok upload (4:5 is native for IG carousel, TikTok crops to 9:16)

Don't try to upload for them — that's their job.

## Production split (LOCKED rule)

| Element | Tool | Why |
|---|---|---|
| Hero illustration / photoreal cover | **AI gen** (Higgsfield gpt_image_2) | HTML can't make marble statues |
| Text-heavy / repeatable body slides | **HTML / CSS** (this repo's templates) | AI mangles typography, drifts on re-renders |
| Real product screenshots | **HTML `<img>`** | AI hallucinates UIs |
| Final delivery | **Playwright** (`shared/export.mjs`) | Deterministic, repeatable, 2x retina |

**If the user proposes AI-gen for a body slide, push back.** This isn't a style preference — AI typography drifts and value slides need editable copy.

## Brand tokens

Default Chase AI palette (swap-able per `docs/brand-tokens.md`):

- Cream `#F5EBE0`
- Ink `#0E0E0E`
- Terracotta `#D97757` (Template 01 accent), `#C46B4F` (Template 02 bg)
- Cobalt sky `#3D7DC8 → #7AB0DE` (gradient, marble-statue covers)
- Headline font: Inter Black (Google Fonts) — substitute for Söhne
- Mono font: JetBrains Mono
- Subtitle font: Didone-style italic serif

If the user wants different brand tokens, edit `templates/<id>/slides.html` (search for `--cream`, `--ink`, `--terracotta`, `--cobalt`) and update the cover JSON prompts in `prompts/`. See `docs/brand-tokens.md` for the full swap guide.

## Common pitfalls (warn the user proactively)

- **Don't reuse a past-dated folder for new work.** Each carousel = today's date.
- **Don't AI-gen body slide heroes.** Real screenshots or HTML composition only.
- **Don't skip the sync ritual.** localStorage ≠ source. Bake uses source `defaults` only.
- **Don't include CHASE AI / pill button / swipe arrow in the cover JSON prompt** — IG and TikTok add their own chrome. Adding chrome in the gen causes double-chrome.
- **Higgsfield max aspect is 3:4** — IG carousel is 4:5. Always pre-crop with `shared/crop-cover.mjs`. Anchor crop to bottom (preserves headline).
- **TOP N / headline must be in the lower half** of the cover, never top edge — IG and TikTok clip the upper 120px.
- **`higgsfield auth status` does NOT exist.** Use `higgsfield auth token` to check auth — prints token if authed, errors otherwise.
- **The CLI binary is named `hf` (not `higgsfield`)** at the file level on most installs. We rename/duplicate to `higgsfield.exe` on Windows because `hf` collides with Hugging Face CLI. On Mac/Linux the install script handles symlinks. Prefer the `higgsfield` command name everywhere — clearer + no collision risk.
- **No `--prompt-file` flag in the CLI.** Flatten the JSON to a text prompt yourself and pass via `--prompt "..."`. (The `higgsfield-generate` skill also doesn't auto-handle this — it takes the same flat text.)
- **`--count` doesn't exist; it's `--batch_size`.** `--aspect-ratio` is `--aspect_ratio` (underscore). Easy to miss.
- **`let i` block scoping breaks Playwright export** — `slides.html` exposes `window.goSlide(idx)` for this reason. Don't refactor that away.
- **Fonts must load before screenshot** — `export.mjs` waits for `document.fonts.ready` + 400ms settle. Don't skip the wait.

## Folder convention

```
carousels/YYYY-MM-DD-<slug>/
├── slides.html              # forked from template, edited per carousel
├── cover.png                # final cropped 4:5 cover
├── cover-iterations/        # Higgsfield variants — keep around for reference
├── assets/                  # hero images, screenshots, etc
└── exports/                 # final PNGs (created by export.mjs)
```

## Where to look for more context

- `README.md` — public-facing overview (don't repeat what's there back to the user)
- `templates/<id>/recipe.md` — per-template how-to + visual DNA + slot reference
- `docs/lessons.md` — accumulated cross-template learnings
- `docs/higgsfield-prompting.md` — deep dive on prompt structure
- `docs/brand-tokens.md` — palette + font swap guide
- `examples/<id>/README.md` — per-example writeup (template choice, production decisions, what worked, what got cut)

## Tone with the user

- They watched a video — assume some familiarity with the concepts (hybrid, tweak loop, sync ritual) but verify each step is happening
- Move through the pipeline without asking permission at every micro-step — but DO show + confirm before burning Higgsfield credits (gen step) and before baking (export step)
- If a step fails, debug it — don't just retry blindly
- After ship, ask if they want to start another or stop

---
> Source: [cth9191/chase-ai-carousels-template](https://github.com/cth9191/chase-ai-carousels-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
