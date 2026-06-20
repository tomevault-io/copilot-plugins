## ig-lead-simple

> >


# IG Lead Magnet Simple (v1)

> **Public release notice**: this skill was extracted from a personal toolkit. All account IDs, MCP URLs, GCS bucket names, vault paths, and product names have been replaced with `<PLACEHOLDER>` tokens. See [README.md](README.md) for setup; see [config.example.md](config.example.md) for the full placeholder list.

The fastest IG lead-magnet loop: comment keyword → public reply → DM hands a Google Drive link directly. **No email capture, no nurture sequence**. You trade the CRM list for the lowest-friction validation speed.

## Usage

```
/100m-leads:ig-simple <product> [language]
```

| Arg | Values | Default |
|---|---|---|
| `product` | One of the products listed in your local config (see `config.example.md`). | required |
| `language` | `zh` / `zh-Hant` (**Traditional Chinese**) or `en` (English) | `zh` |

> 🔴 **Language hard rule**: `zh` = **Traditional Chinese (zh-Hant)**, **simplified Chinese is forbidden**. Every prompt, PDF body, caption, DM, and carousel text overlay must explicitly say `Traditional Chinese` / `繁體中文` so the LLM / gpt-image-2 doesn't render simplified.

**Examples** (the actual handles come from your `config.example.md`):

- `/100m-leads:ig-simple <your-product-A> zh` → Traditional-Chinese run → IG `<your_zh_handle>` + TikTok `<your_zh_tiktok_name>`
- `/100m-leads:ig-simple <your-product-B> en` → English run → IG `<your_en_handle>` + TikTok `<your_en_tiktok_name>`

The skill will automatically:
- Read the product's PRD / `lead-magnet.md` / `design-guideline.md` / `campaign/*.md` / image lists
- Apply the product-specific design system (color preset comes from `design-guideline.md`, not the YAML default)
- Route to the language-matched IG + TikTok account (**dual-platform synchronized publish**)
- Run all 5 phases; pause for final confirmation right before Phase 4 publishes

> The point of skipping interactive questions: stop re-asking "which product / which palette / which account" every run. Decisions are 90% automatic; you only pick the lead-magnet topic and hit publish.

```
        ┌─────────────────────────────────────────┐
        │            parallel execution           │
        │                                         │
        │  ┌──────────────┐  ┌──────────────┐     │
        │  │ Phase 1      │  │ Phase 2      │     │
        │  │ Lead Magnet  │  │ IG Carousel  │     │
        │  │ PDF + Drive  │  │ (gpt-image-2)│     │
        │  └──────────────┘  └──────────────┘     │
        │                                         │
        │  ┌──────────────┐                       │
        │  │ Phase 3      │                       │
        │  │ Caption + DM │                       │
        │  └──────────────┘                       │
        └────────────────────┬────────────────────┘
                             ▼
                  ┌──────────────┐    ┌───────────┐
                  │ Phase 4      │    │ Phase 5   │
                  │ Boring publ. │ →  │ Metrics   │
                  │ +autoReply+DM│    │           │
                  └──────────────┘    └───────────┘
```

> **When to use this vs v2 (`ig-leadmagnet-machine`)**: you want to validate a hook / lead-magnet's market fit fast, you don't want to build an email nurture, the product is too early to justify a list. If you need a nurture sequence, switch to v2.

## References

| File | Purpose |
|------|---------|
| [`references/google-drive-upload.md`](references/google-drive-upload.md) | Drive API flow, folder structure, helper script, permissions |
| [`references/boring-mcp.md`](references/boring-mcp.md) | Boring publish + auto-reply + DM Quick Reply |
| [`references/copywriting-skills.md`](references/copywriting-skills.md) | Caption / Carousel / DM structure (Sugarman framework) |
| [`references/media-creation.md`](references/media-creation.md) | VPick gpt-image-2 prompt patterns, 7-page immersive carousel design |
| [`references/carousel-design-system.yaml`](references/carousel-design-system.yaml) | **Authoritative design source** — 7-page narrative, 4 palette presets, type / grid / cross-page |

---

## Phase 0: Pre-flight + product confirmation

### 🔴 Step 0.1: MCP connection health check

```bash
claude mcp list | grep -E "boring|vpick|obsidian"
```

**All three must be `✓ Connected`**:

| MCP | URL | Purpose |
|-----|-----|---------|
| `boring` | `<your-boring-host>/mcp/t/<token>` | IG publish + auto-reply + DM |
| `vpick` | `<your-vpick-host>/mcp/t/<token>` | Carousel images (gpt-image-2 @ 1K @ 4:5) |
| `obsidian-mcp` | `<your-obsidian-mcp-host>/mcp` | Read/write Obsidian campaign records (cross-machine) |

If any shows `✗ Failed to connect`:

```bash
claude mcp remove <name>
claude mcp add --transport http <name> <url>
# /exit and restart the session — `claude mcp add` does not load deferred tools live
```

> First time on a new machine: run all three `claude mcp add` commands, then `/exit` and restart.

### 🔴 Step 0.1.5: Sandbox permission pre-check (**required**)

Auto-mode harness blocks "Create Public Surface" by default (publishing to IG, uploading public Drive files, calling public APIs). Phase 1 + Phase 4 hit these. **Before running the skill, confirm `.claude/settings.local.json` has these 7 allow rules**; surface what's missing for the user to copy-paste, then proceed to Phase 1.

**Required allow rules** (under `permissions.allow`):

```json
"Bash(curl *www.googleapis.com/drive/v3/files*)",
"Bash(curl *upload/drive/v3/files*)",
"Bash(curl *tinyurl.com/api-create.php*)",
"Bash(gcloud auth print-access-token *)",
"mcp__boring__boring_publish_post",
"mcp__boring__boring_create_auto_reply_rule",
"mcp__boring__boring_update_auto_reply_rule"
```

**Logic**:

```
Read .claude/settings.local.json's permissions.allow array
→ List missing rules to user
→ User edits manually and saves (agent CANNOT self-add — self-modification is always blocked)
→ User says "ok" → proceed to Phase 0.2
```

> ⚠️ **Don't try `update-config` skill to auto-add** — same self-modification guard blocks it. **Only the user can edit manually**.

### Step 0.2: Confirm product + language

**Take `<product>` and `[language=zh]` from args**. Only ask interactively if args are missing.

PRDs are read via the Obsidian MCP (path is relative to your vault root):

```
obsidian-mcp:read_file(path="<your-products-folder>/<product>/PRD.md")
```

**Available products** are defined in your local `config.example.md` — your products live under `<your-products-folder>/<product>/PRD.md` in your Obsidian vault.

> If the user names a product not in your list, run `obsidian-mcp:list_files(path="<your-products-folder>")` to confirm; if no PRD found, ask back.

### 🔴 Step 0.2.4: Account routing (auto-select IG + TikTok by language)

Don't ask "which IG account" — it's deterministic from the `language` arg:

| Language | IG handle | IG account_id | TikTok name | TikTok account_id |
|---|---|---|---|---|
| `zh` / `zh-Hant` (**Traditional Chinese**) | `<your_zh_handle>` | `<IG_ZH_ACCOUNT_ID>` | `<your_zh_tiktok_name>` | `<TIKTOK_ZH_ACCOUNT_ID>` |
| `en` (English) | `<your_en_handle>` | `<IG_EN_ACCOUNT_ID>` | `<your_en_tiktok_name>` | `<TIKTOK_EN_ACCOUNT_ID>` |

> The actual account IDs are in your local `config.example.md` (gitignored). Get them via `boring_list_accounts` and paste into your local copy.

> 🔴 `zh` = Traditional Chinese. Every instruction to LLM / gpt-image-2 must explicitly say **"Traditional Chinese (繁體中文)"** — never just "Chinese" (interpreted as simplified) and never "Simplified Chinese".

**Dual-platform synchronized publish**:
- **IG**: Carousel + auto-reply + DM Quick Reply (full loop)
- **TikTok**: same 7 images as photo slideshow broadcast (no auto-reply — Boring only supports fb/ig keyword rules). Caption adds a redirect line: "Want the PDF? Comment '<keyword>' on my IG @<language-matched IG handle>" routing TikTok viewers to IG.

> Write into BRIEF: `account_handle` uses the language-matched IG handle (for caption top-left).

### Step 0.2.5: Read this product's past campaign records

**Required step**. **Try `obsidian-mcp` MCP first**, fallback to local Obsidian vault on read failure (your MCP server's vault may not be the same as your local vault):

```
# First try — obsidian-mcp (cross-machine friendly)
obsidian-mcp:list_files(path="<your-products-folder>/<product>/campaign")
obsidian-mcp:read_file(path="<your-products-folder>/<product>/campaign/<file>.md")

# On failure (ENOENT / empty array) → fallback to local Read
# Local vault path (note vault root often has an extra same-named folder):
Read("<YOUR_LOCAL_VAULT_ROOT>/<your-products-folder>/<product>/campaign/<file>.md")
```

> ⚠️ The Obsidian MCP server may run on a different machine from your local Obsidian app. They are typically **not** symlinked. Local edits won't be visible to MCP until rsync'd. If MCP read fails, just fallback locally — don't panic.

Each `.md` is one past campaign log (Hypothesis → Variables → Publishing Results → Learnings → Results → suggestions for next run).

**Why read them**: absorb prior lessons + adjust this run. Common takeaways:
- Last run's high-reach / high-comment hook → extend it
- Last run's flop hook → don't repeat
- Last run's prompt patterns that timed out / mis-rendered → preempt
- Last run's PDF chapter that resonated most with leads who stayed → strengthen

**If `list_files` returns not-found / empty array**: this is the product's first run; no need to pre-create the folder (`create_file` auto-creates intermediate dirs).

**Output**: a 1-2 sentence summary of "what last time taught me" into the Step 0.4 proposal (so the user sees you read history).

### Step 0.2.6: Read "product images" + "protagonist photos" asset lists (cloud URLs)

**Required step**. The carousel design system needs two visual asset groups, both already on a public-readable GCS bucket. **Same fallback rule as Step 0.2.5 — MCP first, local Read on failure**:

```
# Product UI screenshots (embeddable on Solution / Method pages)
obsidian-mcp:read_file(path="<your-products-folder>/<product>/img-Product.md")
# fallback: Read("<YOUR_LOCAL_VAULT_ROOT>/<your-products-folder>/<product>/img-Product.md")

# Protagonist photos (cover, cross-page hero shots)
obsidian-mcp:read_file(path="<your-protagonist-folder>/protagonist-image.md")
# fallback: Read("<YOUR_LOCAL_VAULT_ROOT>/<your-protagonist-folder>/protagonist-image.md")
# Note: filename may have spaces — list_files to confirm
```

**Md format convention** (hard rule, see `references/media-creation.md` §1): each image entry is two lines — `![[wikilink]]` (Obsidian preview) + `→ https://...` (the cloud URL the carousel actually uses). **Read the md, take only the line starting with `→`** and feed that URL to `vpick:upload_image`.

**Bucket structure** (your own — define in `config.example.md`):
- Protagonist photos → `gs://<YOUR_GCS_BUCKET>/<YOUR_PROTAGONIST_PATH>/`
- Product screenshots → `gs://<YOUR_GCS_BUCKET>/products/<product>/`

**Output**:
- Product image list (purpose + cloud URL, e.g. "Product canvas portrait → https://storage.googleapis.com/.../canvas-portrait.png")
- Protagonist photo list (split landscape / portrait, with cloud URL)

**Selection rule** (see `references/media-creation.md` §1):
- Cover (page 1) + cross-page (5+6) → pick landscape
- Solution / Method page (4) → can embed 1-2 product screenshots
- Across the 7-page run, protagonist photo appears 2-4 times (per BRIEF.protagonist.photos_to_use)

> If an image in the md doesn't yet have `→ URL` → upload it to GCS first (`gsutil -m cp ... gs://<YOUR_GCS_BUCKET>/products/<product>/`), update the md, then proceed to Phase 2.
> If `img-Product.md` doesn't exist → run `list_files` to confirm, then ask the user to add screenshots first.

### 🔴 Step 0.2.7: Read this product's `design-guideline.md` (**required**)

This is the product-specific visual design system, **higher priority than the `carousel-design-system.yaml` default presets**. Phase 2's image generation must apply it.

```
# Same fallback rule as 0.2.5: MCP first, local Read on failure
obsidian-mcp:read_file(path="<your-products-folder>/<product>/design-guideline.md")
# fallback: Read("<YOUR_LOCAL_VAULT_ROOT>/<your-products-folder>/<product>/design-guideline.md")
```

**Extract from design-guideline.md** (write into BRIEF):

| Section | Extract | Used in |
|---|---|---|
| Primary / Secondary / Accent colors | hex codes + roles | YAML PART 2 → `color_scheme: custom` slots |
| Brand mood / 1-line visual summary | e.g. "film studio + AI node canvas" | imageGenerator prompt's `mood` segment |
| Typography system | CJK + Latin font preferences | YAML PART 3 type tweaks |
| Typography rules | title size / leading / contrast | imageGenerator prompt's text overlay segment |
| Do / Don't visuals | e.g. "no toy aesthetic", "needs node-system feel" | imageGenerator prompt's style locking segment |
| Brand keywords | e.g. "Cinematic / Modular / Intelligent" | Caption / DM mood words |

**If `design-guideline.md` doesn't exist** → product hasn't built one. Ask the user to either run another skill to draft one, or fall back to the `carousel-design-system.yaml` default preset.

> ⚠️ **Past lesson**: a previous run completely skipped `design-guideline.md` and used the YAML default preset (`ink_lemon` — black + lemon yellow), but the product's actual design system was **Electric Violet `#7C3AED` + Canvas Navy `#111827` + Cinematic Cyan `#22D3EE`**. Wrong palette. Always read the guideline.

### Step 0.3: Read PRD, extract key info

After reading `<product>/PRD.md`, organize:

| Field | Source |
|------|------|
| **Core pain points** | `## 🔧 Problem` section |
| **Target audience** | inferred from Problem + Why it matters |
| **Value proposition** | `## ✅ Solves` section |
| **Competitors + diff** | `## 📊 Why it matters` section |
| **Product features** | PRD middle (Solution / How it works) |
| **Pricing** | Why it matters table |

### Step 0.3.5: Read Lead Magnet strategy page

**Required step**. Each product has a lead-magnet strategy doc listing 5 personas, 3 lead-magnet candidates per persona, prioritization, CTA design:

```
obsidian-mcp:read_file(path="<your-products-folder>/<product>/lead-magnet.md")
```

**Organize for Step 0.4 proposal**:

| Field | Source |
|---|---|
| **Suitable lead magnet form** | strategy doc opening "TL;DR" (Checklist / Template / Cheat Sheet / Blueprint / Demo) |
| **Product core messages** | "Product core messages" section (4-5 anchors to align lead magnets to) |
| **5 personas** | each persona's "biggest pain" |
| **15 lead-magnet candidates** | per persona × 3 |
| **Top-3 priority** | "Prioritization" section's first tier |
| **CTA design** | "Suggested CTA design" section |

**Selection logic**:
- Default: pick one from **Top-3 priority** (covers core audience, easy to extend into carousel + PDF)
- If user explicitly names a persona / topic → pick from that persona's 3 candidates
- Align with past campaigns (Step 0.2.5): which hook flopped, which drew comments → influences which to pick

**If `lead-magnet.md` doesn't exist** → product hasn't built a strategy. Ask back: run `lead-magnet-creator` skill first, or improvise from PRD pain points this once?

### Step 0.4: Plan proposal (user confirmation)

Combine PRD extract + lead-magnet strategy into a concrete proposal, **wait for user nod before Phase 1**:

```
## IG Lead Magnet plan — [product]

**Lead magnet candidates** (from lead-magnet.md Top-3, user picks one or specifies):

1. "[First-tier title 1]" — form: [Checklist / Template / Cheat Sheet / ...]
   Persona: [...]
   Promise: [one line]

2. "[First-tier title 2]" — form: [...]
   Persona: [...]
   Promise: [...]

3. "[First-tier title 3]" — form: [...]
   Persona: [...]
   Promise: [...]

→ I recommend [N] because [reason from past campaigns / PRD].

**Selected lead magnet PDF**: [pick 1 of 3, or user-custom]
- Audience (Persona): [from lead-magnet.md]
- Problem solved: [PRD Problem #X + lead-magnet.md pain]
- Planned chapters: [expand the chosen candidate's "content" into 3-5 sections]
- CTA: [pick 1-2 from lead-magnet.md "Suggested CTA design"]

**Publishing channels** (auto-routed by language — Step 0.2.4):
- IG: <your_zh_handle / your_en_handle>
- TikTok: <your_zh_tiktok_name / your_en_tiktok_name> (cross-post photo slideshow, caption redirects to IG comments)

**IG carousel plan** (immersive 7-page, applying `design-guideline.md` > `carousel-design-system.yaml`):
- 7 pages (Cover / Hook Reframe / Stakes / Method 1 / Method 2 / Method 3 / CTA)
  - pages 5+6 are a cross-page (one image cut into two halves)
- Palette: **extract from design-guideline.md** (product-specific) → write into YAML PART 2 `custom`
  - Fall back to one of 4 yaml presets only if no design-guideline.md
- Protagonist photos: ×[2-4] from `<your-protagonist-folder>/protagonist-image.md` → upload as VPick reference in Phase 2.0
- Product UI screenshots: [1-2 from `<your-products-folder>/<product>/img-Product.md`] → upload as VPick reference in Phase 2.0
- Cover hook: "[pick from pain / desire / mystery]"
- Trigger keyword: "[short memorable, ≤ 4 zh chars / ≤ 8 en chars]"

**BRIEF summary** (YAML PART 1):
- series_name: [series — reuse from last campaign or new]
- account_handle: @<language-matched IG handle>
- product_name: [...]
- topic: [...]
- color_scheme: `custom` (filled with design-guideline.md hex)
- protagonist.photos_to_use: 2-4
- cta.keyword / deliverable: [keyword / what we're sending]

**DM payload**:
- Copy intro + Google Drive PDF link
- Expected CTA: tap link → reads PDF immediately

Confirm direction OK and I'll run Phase 1/2/3 in parallel.
```

---

## Phase 1: Lead Magnet PDF + Google Drive upload

> 🔴 **Language hard rule**:
> - `language=zh` → **PDF entirely in Traditional Chinese (zh-Hant)**, HTML uses `<html lang="zh-Hant">`, fonts include `Noto Serif TC` + `Noto Sans TC`. **No simplified.**
> - `language=en` → entirely English, HTML `<html lang="en">`, fonts Inter / Playfair Display
> - Every LLM prompt for copy must say `output in Traditional Chinese (zh-Hant)` or `output in English`, never just "Chinese"

### Step 1.1: Design PDF content (using lead-magnet.md selected option)

Invoke `lead-magnet-pdf-creator` skill (from the marketing-agent plugin):

- **Source**: Step 0.3.5 selected option (title / form / promise / outline / CTA)
- **Reinforce with**: PRD pain points, solutions, features
- Built-in Sugarman copywriting engine auto-optimizes
- Outputs a brand-styled PDF

**Content structure** (apply lead-magnet.md's chosen option's "content" as chapters; tweak per form):

| Form | PDF structure |
|------|--------------|
| **Checklist** (most common) | Cover → ch 1-N each one checklist item → final CTA |
| **Template / Prompt Pack** | Cover → templates 1-N (with use case + sample output) → final CTA |
| **Cheat Sheet** | Cover → one comparison table + usage notes → final CTA |
| **Blueprint / Workflow** | Cover → flowchart → each phase breakdown → final CTA |
| **Walkthrough** | Cover → step-by-step (with screenshots / code) → final CTA |

**Generic skeleton** (tweak per form):

```
Cover — title + promise subtitle (from lead-magnet.md option)
↓
[chapters per form, content from lead-magnet.md "content" section]
↓
Final CTA — pick 1-2 from lead-magnet.md "Suggested CTA design"
(CTA 1 signup / CTA 2 try directly / CTA 3 nurture)
```

> Don't always use a fixed "Problem → Failure → Solution → Steps" 5-chapter mold. **Form follows lead-magnet.md** — Checklist is pure Checklist, Cheat Sheet is one comparison table.

### Step 1.2: Upload to Google Drive, get public link

> Full API flow + helper script: [`references/google-drive-upload.md`](references/google-drive-upload.md)

**Architecture**: each lead magnet creates a `<product>-YYYYMMDD` subfolder under your parent folder (`<YOUR_DRIVE_PARENT_FOLDER_ID>`) → set anyone-with-link → drop PDF in → use `https://drive.google.com/file/d/<FILE_ID>/view` for the DM.

**Three core steps** (curl + Drive API; token from `gcloud auth print-access-token --account=<your-gcloud-account>`):

```bash
TOKEN=$(gcloud auth print-access-token --account=<your-gcloud-account>)
PARENT="<YOUR_DRIVE_PARENT_FOLDER_ID>"

# 1. Create dated subfolder
FOLDER_ID=$(curl -s -X POST "https://www.googleapis.com/drive/v3/files" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d "{\"name\":\"<product>-$(date +%Y%m%d)\",\"mimeType\":\"application/vnd.google-apps.folder\",\"parents\":[\"$PARENT\"]}" \
  | python3 -c "import sys,json;print(json.load(sys.stdin)['id'])")

# 2. Set anyone-with-link reader (files inside inherit)
curl -s -X POST "https://www.googleapis.com/drive/v3/files/$FOLDER_ID/permissions" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"role":"reader","type":"anyone"}'

# 3. Upload PDF
FILE_ID=$(curl -s -X POST "https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart" \
  -H "Authorization: Bearer $TOKEN" \
  -F "metadata={\"name\":\"lead-magnet.pdf\",\"parents\":[\"$FOLDER_ID\"]};type=application/json" \
  -F "file=@/tmp/lead-magnet.pdf;type=application/pdf" \
  | python3 -c "import sys,json;print(json.load(sys.stdin)['id'])")

echo "DM URL: https://drive.google.com/file/d/$FILE_ID/view"
```

> ⚠️ **Long URL displays poorly in IG DM** → shorten with bit.ly / TinyURL before Phase 3 (see reference).
>
> ⚠️ **First-time setup**: if token returns `403 ACCESS_TOKEN_SCOPE_INSUFFICIENT`, run `gcloud auth login --enable-gdrive-access` once.

**Output**: `https://drive.google.com/file/d/<FILE_ID>/view` (or shortened) → used in Phase 3.

---

## Phase 2: IG Carousel (VPick gpt-image-2, immersive 7 pages)

> **Authoritative design source**: [`references/carousel-design-system.yaml`](references/carousel-design-system.yaml)
> Phase 2 follows this YAML. Below is just the flow summary — details (PART 3 type / PART 4 grid / PART 5 visuals / PART 7 7-page narrative) live in the YAML.

> 🔴 **Language hard rule** (every imageGenerator prompt):
> - `zh` → `Display Traditional Chinese text (繁體中文 / zh-Hant): "..."` — **never just "Chinese text"** (gpt-image-2 will render simplified)
> - `en` → `Display English text: "..."`
> - Fonts: zh uses `Noto Serif TC Bold` (titles) / `Noto Sans TC Medium` (body) / `Inter Medium` (English labels); en uses `Playfair Display` or `Inter Bold` (titles) / `Inter` (body)

### 🔴 Step 2.0: Upload reference images (**required**)

Before any imageGenerator runs, **upload protagonist + product UI as VPick upload nodes**. imageGenerator prompts then reference them via `@protagonist_ref` `@product_ref` + `connect_nodes`, producing the **real protagonist face and real product UI** instead of AI-imagined versions.

```
# 1. Create / switch into a VPick project for this run
vpick:create_project(name="<product>-<topic>-<date>")

# 2. Upload protagonist photo (URL from Step 0.2.6 → URL)
vpick:upload_image(
  url="<protagonist photo cloud URL from your protagonist-image.md>",
  name="protagonist_ref"
)

# 3. Upload product UI screenshot (most representative 1-2)
vpick:upload_image(
  url="<product screenshot cloud URL from your img-Product.md>",
  name="product_ref"
)
```

**Which pages need references**:

| Slide | Reference | Why |
|---|---|---|
| 1 Cover (image_full_bg) | `@protagonist_ref` | Cover needs the real face — random AI face has no recognition |
| 5 Method 2 cross-page L | `@protagonist_ref` | Sequential shot, must match cover's person |
| 4 Method 1 (with product UI) | `@product_ref` | Embed real UI, not AI imagination |
| 6 Method 3 cross-page R | `@product_ref` (optional) | Topic-dependent |

**imageGenerator prompt pattern**:

```
[scene description...]
Reference image @protagonist_ref for the protagonist's face and styling — keep facial features identical.
[text overlay segment...]
```

When `add_node`'s prompt contains `@protagonist_ref`, VPick auto-`connect_nodes` to the upload node (Element system).

### 🟡 Step 2.0.5: VPick active-project guard

VPick's active project is user-level state — opening another tab in VPick can switch the active project. Verify before each op:

```
# Before add_node / run_image_generator / list_nodes:
vpick:list_projects → find isActive: true → compare against targetProjectId
# If mismatch:
vpick:switch_project(projectId=<targetProjectId>)
```

**Most-common stumbling moment**: mid-batch (4+3 in flight), user opens another VPick project in another tab → subsequent nodes land in the wrong project.

### Step 2.1: Fill BRIEF into YAML PART 1

From Phase 0.4 proposal + PRD extract, fill the YAML `brief` (mental model is fine; don't write a file):

| YAML field | Source |
|------------|--------|
| `series_name` | custom or reuse from last campaign |
| `account_handle` | language-matched IG handle |
| `product_name` | Phase 0.2 confirmed product |
| `topic` | this run's topic (from PRD pain) |
| `protagonist.photo_style` | e.g. "cinematic, low-saturation, study or studio setting" |
| `protagonist.photos_to_use` | 2-4, default 3 |
| `audience.persona / pain_point / desired_outcome` | from PRD |
| `color_scheme` | from design-guideline.md `custom` (preferred) or one of 4 presets |
| `cta.keyword / deliverable` | comment trigger + what's delivered |

### Step 2.2: 7-page narrative breakdown (YAML PART 7)

```
1. Cover         — Cover hook (pain / desire / mystery), full-bleed protagonist landscape
2. Hook Reframe  — "you're not doing X, you're doing Y" reframe, secondary solid bg
3. Stakes        — "{element} is the only {make-or-break / dividing line}", primary solid bg
4. Method 1      — "{action 1}, {action 2}, {action 3}" 3 short verbs, can embed product UI
5. Method 2      — cross-page LEFT (image_full_bg_left), technique + mechanism
6. Method 3      — cross-page RIGHT (image_full_bg_right), metaphor wrap
7. CTA           — desire-language CTA + "comment '<keyword>' for <deliverable>", primary solid bg
```

**Color cadence** (YAML PART 6): page 1 image → 2 secondary → 3 primary → 4 secondary → 5+6 cross-page image → 7 primary.
**Cover headline scoring**: use the `headline-lab` skill, ≥ 20/25 to ship.
**Product image embedding**: page 4 / 5 can embed `img-Product.md` selections (contextual treatment, not naked screenshots).

### Step 2.3: VPick image generation (4+3 parallel batches)

**Use `vpick:run_image_generator` with `gpt-image-2`, fixed 1K + 4:5**:

```
vpick:add_node:
  type: "imageGenerator"
  name: "Slide{N}_{Role}"
  model: "gpt-image-2"
  prompt: "[per media-creation.md §3 4-segment template + YAML PART 7 page template]"
  data: { aspectRatio: "4:5", resolution: "1K" }
```

> **Hard rule**: Carousel images use VPick `gpt-image-2` @ 1K + 4:5. **No fallback to nano-banana**. If layout is bad, rewrite the prompt or hand-fix in a design tool — never switch models.

**Prompt 4-segment** (details in `references/media-creation.md` §3):
1. **Scene**: per BRIEF.protagonist.photo_style + page role (image_full_bg / primary / secondary)
2. **Text overlay**: `Display [language] text: "..."` + position + font (Noto Serif TC Bold / Inter Medium) + key 2-4-word highlight color block (YAML PART 5)
3. **Layout**: top-left series_name + handle, top-right page pill `{N}/7`, bottom gradient mask (required on image_full_bg pages)
4. **Style lock**: YAML palette mood + primary/secondary hex + ban emoji / exclamation / stock photo

**Generation strategy**:
- 7 images split into **4 + 3 parallel batches**
- Cross-page 5+6 in same batch, prompts cross-reference "right edge cuts off" / "left edge continues from prior" + composition negative space
- Each completion auto-uploads, returns URL

### Step 2.4: Order the 7 URLs

Arrange by slide order:

```
slide_urls = [
  "https://...slide1-cover.png",
  "https://...slide2-hook-reframe.png",
  "https://...slide3-stakes.png",
  "https://...slide4-method1.png",
  "https://...slide5-method2-left.png",
  "https://...slide6-method3-right.png",
  "https://...slide7-cta.png",
]
```

> If VPick's URL is rejected by IG, re-host via `boring:boring_upload_from_url` to get a Boring-accepted public URL.

---

## Phase 3: Caption + keyword + DM copy

> 🔴 **Language hard rule**:
> - `zh` → caption / keyword / public reply / DM intro / full DM body **all Traditional Chinese**. Use TW/HK conventions (e.g. 「影片」 not 「视频」, 「軟體」 not 「软件」, 「使用者」 not 「用户」)
> - `en` → all English, no zh/en mixing
> - When invoking copywriting-bible sub-skills, prompt must say: `produce all output in Traditional Chinese (zh-Hant) — Taiwan / HK conventions, NOT Simplified Chinese`

### 🎯 3.0 CTA philosophy (**read before writing copy**)

**Use desire language, not fear language**. Pain can anchor attention, but only **desire** triggers action.

#### Three elements

1. **Familiar scene** — anchor to their daily moment: morning coffee, scrolling IG, end-of-day collapse, weekend "I want to do something but not all day"
2. **Already-desired outcome** — make their wanted result concrete and visible: video shipped, client commenting in group, friends asking how, finishing on time
3. **Easier than they think** — preempt the "this sounds great but the bar must be high" reflex: "one sentence → done", "no new UI to learn", "tools you already use"

#### ❌ Fear version (don't write this)

- "Keep using Runway and you'll keep wasting time"
- "Don't solve this and your videos will never ship"
- "Your competitors are already on AI; you're still hand-cutting timelines"
- "I know [pain] is really frustrating..." ← still fear language

#### ✅ Desire version

> Imagine tomorrow morning, you open your laptop, tell Claude "make me a 5-shot car ad" and walk to the kitchen for coffee.
> By the time you sit back down — 6 storyboard images, 5 video clips, one audio track are arranged on the canvas. Hit publish.
> No new software. No 7-tool tab-switching. No 2 AM cuts.
> What you thought would take half a year of practice? **Today, one sentence kicks it off.**

⤷ This hits all three: ① familiar scene (morning, coffee) ② desired outcome (automated result) ③ easier than thought (one sentence, no learning).

### 3.1 Caption (apply `slippery-slide-writer`)

```
[Hook — first line ≤ 10 zh chars / ≤ 12 en words] (first-sentence-crafter)
↓ Hook can carry pain to anchor attention

[Pain amplification — 1-2 lines ≤ 30%]
short, only does the "yes I feel that too" resonance

[Bridge]
"But..." / "Until I found..." / "Then..."

[Desire scene — 2-3 lines ≥ 70% the real engine]
familiar scene + desired outcome + easier-than-thought

[Value preview — 3-5 bullets]
use "you'll", "you can", "you will" (capability framing)

[CTA — desire language] (emotional-sell-rational-justify)
👇 Comment "<keyword>" and I'll DM you the full guide

#hashtag1 #hashtag2 ... (5-10)
```

> Ratio: pain ≤ 30%, desire + action ≥ 70%. Pain anchors; desire drives.

### 3.2 Keyword choice

Short, memorable, hard to misspell. **Pick something with desire/action energy** rather than vague generic verbs:

| Good | Better (desire-flavored) | Bad |
|---|---|---|
| `YES`, `SEND`, `FREE` | `EDIT`, `BUILD`, `WIN`, `KIT`, `GUIDE` | long phrases, mixed-language |

### 3.3 Auto-reply + DM copy (core diff vs v2)

**Public auto-reply** (other commenters see "yes this really sends" → FOMO):
```
Sent to your DMs ✉️
```
Note: not "thanks for your interest" (sounds desperate). Use a completed-action sentence.

**DM private** (**direct Drive link, all desire language**):

```
[1. Friendly intro — not subservient]
Hey 👋 glad you grabbed this one.

[2. Desire scene — 1-2 lines, NOT "I know you're struggling"]
The thing I love most about it: [familiar scene + desired outcome].
I thought it would take [assumed high bar], but [the easier truth].

[3. Direct delivery — Google Drive URL]
Here's the full guide (free PDF, no signup):
👉 [Google Drive public link]

[4. Value preview — "you can / you'll / you will" framing]
Inside you'll find:
✓ You can [outcome from chapter 1]
✓ You'll learn [capability from chapter 2]
✓ You'll get [specific result] in [specific time] (chapter 3)

[5. Soft CTA — open invitation, not pressure]
Ping me anytime if you want to chat.
Or jump straight in: [product URL] — [one desire line].
```

**Pre-send checklist**:
- [ ] Zero "frustrating", "annoying", "painful", "problem"
- [ ] At least 1 familiar scene line (morning, coffee, end-of-day, weekend...)
- [ ] At least 1 already-desired outcome made concrete
- [ ] At least 1 "easier than you think" line
- [ ] Bullets start with "you can / you'll / you will", not feature names

> **vs v2**: v2's DM is an Intercom landing page (must enter email); this version's DM is the Drive PDF directly — tap and read.

---

## Phase 4: Boring publish + automation

> **🔴 Confirm 3 things upfront with the user**: the harness splits each external write into separate approvals. A single "ok" from the user only lets through the first. Be explicit:
> > "Phase 4 has 3 external writes: ① publish IG carousel ② create IG auto-reply + DM rule ③ cross-post to TikTok. Reply 'send' to authorize all 3 in one go."

### Step 4.1a: Publish carousel to IG (language-matched account)

```
boring:boring_publish_post:
  account_id: "<language-matched IG account_id (Step 0.2.4)>"
  platform: "instagram"
  text: "[Phase 3.1 caption]"
  media_urls: [Phase 2.4 7 URLs]
```

### 🔴 Step 4.1.5: Get post_url (**required — used in 4.2**)

After publish, IG returns `media_id` but **no post_url**. `boring_create_auto_reply_rule` server-side needs `post_url` to bind the rule. **Right after publish, call**:

```
boring:boring_get_publish_history(platform="instagram", limit=3)
# → find the entry matching submission_id, take post_url
# Format: https://www.instagram.com/p/<shortcode>/
```

### 🔴 Step 4.1b: Cross-post to TikTok (same 7 images as photo slideshow)

Don't regenerate. Use the same 7 carousel URLs on TikTok. But rewrite the caption — TikTok has no keyword auto-reply, so **redirect viewers to IG comments** to get the PDF.

```
TIKTOK_CAPTION = """
[1-2 line hook, same as IG]

⬇️ Want the full guide PDF?
Comment "<keyword>" on my IG @<language-matched IG handle>
and I'll DM you the whole thing 📩

[remaining caption...]
"""

boring:boring_publish_post:
  account_id: "<language-matched TikTok account_id (Step 0.2.4)>"
  platform: "tiktok"
  text: TIKTOK_CAPTION
  media_urls: [same 7 URLs as IG]
  draft: false
```

> ⚠️ TikTok doesn't support Boring's keyword auto-reply (only fb / ig). TikTok is broadcast-only — caption funnels traffic to IG.
>
> ⚠️ If TikTok rejects an image URL (hash already used), re-host with `boring:boring_upload_from_url`.

### Step 4.2: Set comment auto-reply + DM

**Conflict check**:
```
boring:boring_list_auto_reply_rules
# Same account + same keyword → boring_delete_auto_reply_rule then recreate
# Different accounts or different post_urls can coexist (rules bind to post_url)
```

**Create rule** (**post_url is required**, account_id matches language):

```
boring:boring_create_auto_reply_rule:
  account_id: "<language-matched IG account_id (Step 0.2.4)>"
  platform: "instagram"
  post_url: "[IG post URL from Step 4.1.5]"  ← required
  keyword: "[Phase 3.2 keyword]"
  reply_message: "[Phase 3.3 public reply]"
  dm_message: "[Phase 3.3 DM intro, with Quick Reply]"
  dm_quick_replies:
    - title: "Send the PDF"
      payload: "send_link"
  dm_quick_reply_responses:
    send_link: "[Phase 3.3 full DM with Drive PDF URL]"
```

> ⚠️ If the MCP tool schema doesn't list `post_url` but the server requires it → schema is stale, **`/exit` and restart the session** to pull a fresh schema.

### Step 4.3: Test the loop

**After publish, manually test**:

1. Use a second IG account, comment the keyword
2. Public auto-reply ✓
3. DM received with intro + Quick Reply button ✓
4. Tap Quick Reply → full DM + Drive link ✓
5. Tap Drive link → PDF opens ✓

If any step fails, fix it now — **don't wait for real comments to discover it's broken**.

### Step 4.4: Write campaign log to Obsidian

**Required step, every run**. This is for the next run to read; skipping = wasted run.

**Use `obsidian-mcp` MCP** (don't use local Write — the skill is designed cross-machine and local paths aren't portable; MCP is the only guaranteed cross-machine path):

```
obsidian-mcp:create_file(
  path="<your-products-folder>/<product>/campaign/<YYYY-MM-DD>-ig-lead-simple.md",
  content="<frontmatter + 11 sections markdown>"
)
```

> Path is relative to vault root (vault root is controlled by your obsidian-mcp server). `create_file` auto-creates intermediate dirs, but **errors if the file exists** — for same-day re-runs, use `update_file` or rename `-v2`, `-v3`.

#### Required frontmatter

```yaml
---
date: YYYY-MM-DD
skill: ig-lead-simple
product: <product>
status: published
platform: instagram
ig_account: <your_zh_handle / your_en_handle>
post_id: <Boring's media_id>
keyword: <trigger keyword>
---
```

#### Required sections

| Section | What to write |
|---------|--------------|
| **Hypothesis** | This run's core bet — which hook, which lead magnet, which keyword |
| **Lead Magnet Choice** | Which option from `lead-magnet.md` (title + form + persona) + why this vs the other Top-3 |
| **Variables** | All controllable variables (hook type / pages / model / resolution / keyword / DM flow / cross-post) |
| **Content Plan** | PDF chapters + Carousel 7-page plan (per yaml) + Caption structure |
| **BRIEF Snapshot** | YAML PART 1 values you used |
| **Image Prompt Base** | Common prompt patterns + design rules (so next run avoids the traps) |
| **Expected Outcome** | Reach, comment rate, DM trigger, conversion baselines |
| **Status** | `published` |
| **Publishing Results** | post_id, submission_id, auto-reply rule_id, publish time, account, Drive PDF URL, 7 carousel URLs |
| **Learnings** | What broke this run, the winning move, what next run should preempt |
| **Results (fill in 72h later)** | 6 empty checkboxes: reach, comments, DMs, Quick Reply taps, signups, paid |
| **Next Action** | D+1 / D+3 / D+7 / D+30 — what to check, what to fill back |
| **Suggestions for next campaign** | Explicit do/don't for future-you (most important section) |

#### Follow-ups

- D+3: run `boring_get_posts_performance` + `boring_list_auto_reply_logs`, fill Results
- D+7: compare with same-product v2 records (if any) for a diff
- D+30: add Stripe signup / paid / retention numbers

---

## Phase 5: Basic metrics

> **No email capture = no funnel mid/lower data**. This version sees only the IG end.

### Core metrics (Boring MCP)

```
boring:boring_get_posts_performance → post analytics
boring:boring_list_auto_reply_logs → keyword trigger log
```

| Metric | Baseline | Optimization |
|--------|----------|--------------|
| Reach | — | Hook + hashtags + posting time |
| Comment rate | ≥ 3% | Cover hook + lead magnet attractiveness |
| Read-through (slide 7/7) | IG insights | Cross-page 5+6 hooked them, CTA strong enough to stop |
| DM delivery rate | ≥ 95% | Auto-reply rule setup |
| Drive PDF click rate | Drive Activity | DM copy persuasiveness |

> **Want to know who downloaded**: change Drive permissions to "must sign in to view" — but adds friction. Simple defaults to no friction; if you want this, upgrade to v2.

### Signals to upgrade to v2

If you hit any of these, switch to `ig-leadmagnet-machine` (v2):

- **Want to know who downloaded** → email capture
- **Want a nurture sequence** → email automation
- **Want to A/B test lead magnets** → landing page splits
- **Want CRM / GA4 integration** → Intercom

---

## Output checklist

| # | Item | Tool |
|---|------|------|
| 0 | Past campaign records review | **obsidian-mcp** `list_files` + `read_file` |
| 1a | Product PRD extract | obsidian-mcp read `<your-products-folder>/<product>/PRD.md` |
| 1b | **Lead magnet selected (Top-3 → 1)** | obsidian-mcp read `<your-products-folder>/<product>/lead-magnet.md` |
| 2 | Lead magnet PDF | `lead-magnet-pdf-creator` skill |
| 3 | Google Drive public link | Drive API + URL shortener |
| 4 | 7 carousel images (4:5 1K, immersive) | **VPick** `run_image_generator` (`gpt-image-2`) + carousel-design-system.yaml |
| 5 | Caption + DM copy | `copywriting-bible/*` skills |
| 6 | IG carousel published | **Boring** `boring_publish_post` |
| 7 | Auto-reply + DM Quick Reply | **Boring** `boring_create_auto_reply_rule` |
| 8 | Loop tested | manual: comment → DM → tap Drive → see PDF |
| 9 | Campaign log written to Obsidian | **obsidian-mcp** `create_file` `<product>/campaign/<date>-ig-lead-simple.md` |

---

## Core principles (Simple ethos)

- **Speed > completeness**: validate hook/PDF first, invest in email later
- **Zero friction**: one tap on Drive link → PDF; no email, no signup
- **PRD-driven content**: every run starts from one product's PRD; PDF + caption serve that product's conversion
- **Run fast, throw fast**: failed hooks get cut, no sentimentality

---

## Troubleshooting

### MCP connection (most common)

| Symptom | Fix |
|---------|-----|
| ToolSearch can't find vpick / boring tools | `claude mcp add` then `/exit` to restart session |
| `claude mcp list` shows ✗ Failed | `claude mcp remove → add → /exit` |
| MCP disappears mid-conversation | Immediate `/exit` and restart, don't retry in same session |

### VPick gpt-image-2 issues

| Symptom | Fix |
|---------|-----|
| Text typos / mis-rendering | Rewrite prompt: `Display [language] text: "..."`, avoid long sentences |
| Some of 7 parallel runs fail | Switch to 4+3 batch, retry failed ones individually |
| Style drift | Always lead prompt with YAML palette mood + primary/secondary hex |
| Cross-page 5+6 misaligned | Same-batch run, prompts cross-reference "right edge cuts" / "left edge continues" + composition; if still off, hand-fix in design tool |
| gpt-image-2 typography looks bad | Text-heavy pages (CTA): rewrite prompt (short lines + explicit position), or hand-fix in design tool. **No fallback to nano-banana** |

### Drive / upload

| Symptom | Fix |
|---------|-----|
| Drive link asks for login | Re-check Step 2 permissions (`type:anyone, role:reader`) |
| URL too long for IG DM | bit.ly / TinyURL |
| No gdrive CLI | Use GCS instead (`gsutil cp`) — `https://storage.googleapis.com/...` is also public |

### Boring auto-reply

| Symptom | Fix |
|---------|-----|
| `boring_create_auto_reply_rule` returns 400 "keyword exists" | `list_auto_reply_rules` confirm → `delete` → recreate |
| DM received but no Quick Reply button | Check `dm_quick_replies` array format, title ≤ 13 chars |
| Comment triggered but no DM | Check IG account is business + DM permission granted |

---
> Source: [snoopyrain/ig-lead-simple](https://github.com/snoopyrain/ig-lead-simple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
