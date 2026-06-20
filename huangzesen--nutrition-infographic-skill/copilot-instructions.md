## nutrition-infographic-skill

> Generate nutrition/science infographic images through a consultative workflow: first ask the human what image they need (purpose, audience, topic, language, size, style, data/text, output channel), then generate via available backends such as MiniMax CLI, Codex built-in imagegen, OpenAI Images API, or deterministic Pillow fallback.


# Nutrition Infographic Generator

This skill creates nutrition/science infographic images. The **default workflow is consultative**: ask the human what kind of image they need before generating anything, unless they already supplied enough detail or explicitly asked for a demo.

Recommended generation route: use whichever image backend is actually available in the current environment. MiniMax CLI (`mmx image generate`) is a good first-party CLI option when a MiniMax key/quota is available; Codex CLI / Codex's built-in `imagegen` system skill is a good route in Codex-enabled environments; the deterministic Pillow renderer remains the fallback for exact chart geometry, repeatability, offline rendering, or fast drafts.

## Self-contained usage

This repository is self-contained. When cloned, run commands from the repository root (the directory containing `SKILL.md`). Do not assume the skill lives at `.library/custom/nutrition-infographic`; examples below use relative paths such as `scripts/render_nutrition_infographic.py` and `assets/demo-balanced-plate.json`. If you copy this skill into LingTai's `.library/custom/` or `.library_shared/`, the same relative paths still work when your shell is inside the skill directory.

### Installing on a new LingTai machine

On a fresh computer or in a new LingTai project, clone the repo and copy or symlink it into the agent's skill catalog:

```bash
git clone https://github.com/huangzesen/nutrition-infographic-skill.git
mkdir -p .library/custom
cp -R nutrition-infographic-skill .library/custom/nutrition-infographic
# or: ln -s "$PWD/nutrition-infographic-skill" .library/custom/nutrition-infographic
```

Then refresh the agent so the skill catalog is rescanned. After refresh, the agent only needs to read this `SKILL.md` to know the workflow. The chat history from the machine where the skill was authored is not required.

Prerequisites for polished AI-image routes: at least one image backend must be installed/authenticated and have quota, such as MiniMax CLI (`mmx`) or command-line `codex` with the system imagegen skill / built-in `image_gen` capability. If no AI-image backend is available, use the OpenAI API fallback when credentials exist, or the deterministic Pillow fallback below.

## When this applies

Use this skill for:

- balanced-plate diagrams and meal composition visuals
- macronutrient proportion charts
- nutrition education cards, posters, and article illustrations
- diet/health science explainers where numbers and labels must stay legible
- rapid drafts that should look like finished educational images
- social posts, article covers, slides, handouts, and app onboarding visuals

Do **not** use this as-is for medical diagnosis, patient-specific diet prescriptions, disease-treatment promises, photorealistic food photography, or brand-sensitive publication design without explicit human direction and review.

## First move: ask what image the human needs

If the human has not already provided the essentials, ask a short intake question before generating. Do not interrogate them with a long form; ask for the missing pieces that matter.

### Minimal intake

Ask in the human's language:

> 你想要一张什么样的营养学图片？请告诉我：用途/平台、主题、面向谁、语言、尺寸比例、风格、是否有必须出现的数据或文案。没有细节的话我可以先给你一个默认方案。

For English:

> What nutrition image do you need? Please tell me the purpose/platform, topic, audience, language, aspect ratio, style, and any required data or wording. If you do not have details, I can propose a default.

### If they are in a hurry

Ask only the 3 essentials:

1. **Topic** — what nutrition concept, food, meal, nutrient, or behavior?
2. **Use/channel** — WeChat article, poster, slide, social card, app UI, handout?
3. **Audience/language** — general adults, children, athletes, older adults, diabetes education, etc.; Chinese/English/bilingual?

Then choose sensible defaults for the rest and state them before generating.

### Intake fields to capture

- **Purpose/channel**: WeChat article, service-account reply, Xiaohongshu card, slide, poster, handout, app screen.
- **Audience**: general public, parents, students, athletes, older adults, clinicians, patients, etc.
- **Topic**: balanced meal, protein, fiber, glycemic load, hydration, sodium, pregnancy nutrition, etc.
- **Message goal**: educate, compare, warn, motivate, summarize, explain a process.
- **Language**: Chinese, English, bilingual; simplified/traditional if Chinese matters.
- **Canvas**: square `1024x1024`, vertical `1080x1920`, horizontal slide `1600x900`, print A4, etc.
- **Style**: clean medical, warm editorial, playful cartoon, premium brand, minimalist data viz, etc.
- **Required text/data**: exact title, numbers, units, source, disclaimer.
- **Brand constraints**: colors, logo, QR code, no-logo, typography.
- **Accuracy constraints**: illustrative vs sourced; whether medical/professional review is required.
- **Output path/channel**: where to save and whether to send/attach it.

## Decision tree

1. **Human provided a complete brief** → draft JSON spec or rich prompt and generate.
2. **Human provided only a topic** → ask the minimal intake question; if they say “随便/默认”, proceed with defaults.
3. **MiniMax backend is available but ambiguous** → do not guess the key/region/backend. Ask the human to confirm which `mmx` config/key/region to use, or run `mmx config show`, `mmx auth status`, and `mmx quota show` and summarize the non-secret evidence before generating.
4. **Human wants to compare quality / backend choice is uncertain** → generate both a pure deterministic version and a MiniMax-hybrid version, then show both for selection.
5. **Publication-grade Chinese infographic / exact wording required** → use Pillow/SVG/HTML for the final text/data layer. AI image backends may create a background or illustration, but should not be trusted to render Chinese copy, numbers, units, or disclaimers.
6. **Exact numbers/charts must be correct** → use the Pillow renderer first, or use AI image only for a decorative companion.
7. **Polished editorial visual matters more than exact geometry** → use MiniMax/Codex/OpenAI image generation, then inspect text carefully.
8. **No imagegen available / offline / reproducible draft needed** → use `scripts/render_nutrition_infographic.py`.
9. **Medical personalization requested** → keep it educational, ask for professional constraints, include disclaimer; do not invent clinical advice.

## Default brief if the human says “你来定”

Use this default unless the context suggests otherwise:

- Channel: WeChat article / social card
- Canvas: square, `1024x1024`
- Language: simplified Chinese if the human is using Chinese; otherwise match human language
- Audience: general adults
- Style: clean warm health-education editorial, high contrast, readable labels
- Claims: educational / illustrative, not medical advice
- Footer: `科普示意，不替代医生或注册营养师建议`

## JSON spec schema

Prefer JSON when the content includes structured nutrition data. Supported top-level fields:

```json
{
  "title": "Main title",
  "subtitle": "Short explanatory subtitle",
  "theme": "green",
  "background": "cream",
  "canvas": {"size": "1024x1024", "aspect": "1:1"},
  "audience": "general adults",
  "style": "clean editorial health education",
  "plate": [
    {"label": "Vegetables", "value": 40, "color": "green"}
  ],
  "metrics": [
    {"label": "Protein", "value": "31", "unit": "g", "note": "Satiety", "color": "orange"}
  ],
  "bars": [
    {"label": "Carbs", "value": 45, "target": 55, "unit": "%", "color": "yellow"}
  ],
  "tips": ["Practical tip 1", "Practical tip 2"],
  "required_text": ["Exact phrase that must appear"],
  "footer": "Source / disclaimer"
}
```

The Pillow fallback uses the original subset: `title`, `subtitle`, `theme`, `background`, `plate`, `metrics`, `bars`, `tips`, and `footer`. Extra fields are for prompt-building and should be reflected in GPT image prompts.

## Procedure

1. **Clarify the brief.** Ask for missing essentials. If the human says to proceed, write down the assumptions.
2. **Choose route.** Prefer an available polished image backend (MiniMax CLI or Codex built-in imagegen); use Pillow for exact/reproducible charts; use OpenAI API fallback only when explicitly needed and credentials exist.
3. **Draft a spec/prompt.** Preserve exact numbers and wording. Keep claims conservative. Add source/disclaimer text when appropriate.
4. **Generate.** Save the PNG to a stable output path under the project or requested destination.
5. **Inspect.** Check legibility, label overlap, text language, numerical consistency, and whether unsupported medical claims slipped in.
6. **Iterate.** If text or numbers are wrong in GPT image output, simplify the prompt/spec or switch to Pillow for the data layer.
7. **Deliver.** Send or attach the image on the same channel where the request arrived if possible, and mention assumptions used.

## MiniMax CLI imagegen route

Use this when the environment has the official MiniMax CLI (`mmx`) installed, authenticated, and with available image-generation quota. This route is useful when you want the skill to work outside Codex-specific environments.

### Step 1 — check CLI and auth

```bash
command -v mmx >/dev/null || npm install -g mmx-cli
mmx --help
mmx image generate --help
mmx config show --output json
mmx auth status --output json
mmx quota show --output json
```

Prefer the MiniMax CLI's own auth/config (`~/.mmx/config.json`, created by `mmx auth login` or `mmx auth login --method api-key`) rather than blindly reusing a LingTai LLM preset key. A LingTai MiniMax preset may point at an Anthropic-compatible LLM endpoint such as `https://api.minimaxi.com/anthropic`; that key/backend is not necessarily the same credential/region the first-party image API expects.

Only pass `--api-key` explicitly when you know that key belongs to the MiniMax image API and you also know its region. Otherwise let `mmx` use its stored config:

```bash
# Recommended: use ~/.mmx/config.json as-is.
mmx image generate --help

# Optional only when you intentionally override auth:
# mmx --region global --api-key "$MINIMAX_IMAGE_API_KEY" image generate ...
# mmx --region cn     --api-key "$MINIMAX_IMAGE_API_KEY" image generate ...
```

Use `--region cn` for mainland image API keys and `--region global` (or no region flag) for international/token-plan keys. A region/key mismatch usually appears as `invalid api key`; exhausted or missing quota may appear as `usage limit exceeded` or `no active token plan subscription`. If `mmx quota show` lists `image-01` remaining quota, the image backend is likely configured correctly. When multiple keys/configs exist, ask the human which one to use before spending quota.

### Step 2 — generate an image

```bash
mkdir -p generated
PROMPT='一张方形中文营养学科普信息图，主题：早餐怎么搭配更稳。干净温暖的微信健康科普卡片风格，高对比度，可读中文标题。包含三块：高纤维蔬果、优质蛋白、慢碳水。页脚：科普示意，不替代医生或注册营养师建议。'

mmx image generate \
  --prompt "$PROMPT" \
  --width 1024 --height 1024 \
  --out generated/minimax-nutrition.png \
  --non-interactive --output json --timeout 300
```

Use `--aspect-ratio 1:1` instead of explicit width/height when dimensions are not important. Use `--prompt-optimizer` for rough prompts; avoid it when exact wording/numbers matter.

### Step 3 — inspect and fallback if needed

```bash
file generated/minimax-nutrition.png
```

If MiniMax returns `invalid api key`, first check region/auth/backend; do not assume quota is the problem. If it returns quota/subscription errors after auth is verified, do not loop retries; switch to Codex imagegen, OpenAI Images API, or Pillow fallback. MiniMax can produce attractive layouts, but Chinese text and exact numbers may be garbled; for publication, generate the visual background with MiniMax and render the exact Chinese/text/data layer with Pillow/SVG/HTML, or use the deterministic renderer directly.

## Hybrid: MiniMax no-text background + deterministic text overlay

This is the recommended way to combine MiniMax with publication-grade Chinese nutrition content.

1. **Generate no-text visual layer.** Prompt MiniMax for background/layout only. Explicitly say: no text, no letters, no numbers, no logo, no watermark, blank cards, empty space for later typography.
2. **Overlay exact content locally.** Use `scripts/compose_hybrid_overlay.py` to render Chinese title, cards, data, and disclaimer using local fonts.
3. **Inspect.** Check both the visual layer and the final overlay. If the AI background contains pseudo-text, regenerate with a stricter no-text prompt or crop/cover it.

Example:

```bash
mmx image generate \
  --prompt 'Square 1024x1024 premium nutrition education poster background only. Warm cream and fresh green palette. A beautiful top-down healthy breakfast plate illustration. Four clean blank rounded white cards. No text, no letters, no numbers, no logo, no watermark, no pseudo-writing, lots of empty space for later typography overlay.' \
  --width 1024 --height 1024 \
  --prompt-optimizer \
  --out generated/minimax-layout-no-text.png \
  --non-interactive --output json --timeout 300

python3 scripts/compose_hybrid_overlay.py \
  --background generated/minimax-layout-no-text.png \
  --out generated/hybrid-minimax-text-overlay.png
```

For custom copy:

```bash
python3 scripts/compose_hybrid_overlay.py --write-demo-spec /tmp/hybrid-spec.json
# edit /tmp/hybrid-spec.json
python3 scripts/compose_hybrid_overlay.py \
  --background generated/minimax-layout-no-text.png \
  --spec /tmp/hybrid-spec.json \
  --out generated/custom-hybrid-card.png
```

## Codex CLI built-in imagegen route

Use this when you are in a Codex-capable environment and want a polished GPT-generated visual. The skill's job is to teach the agent how to call **command-line Codex** so Codex can invoke its own system `imagegen` skill / `image_gen` tool.

### Step 1 — write a brief/spec file

Create a JSON spec or text prompt in a normal project path, for example:

```bash
cat > /tmp/nutrition-spec.json <<'JSON'
{
  "title": "早餐怎么搭配更稳",
  "subtitle": "高纤维 + 优质蛋白 + 慢碳水 · 科普示意",
  "audience": "general adults",
  "language": "Simplified Chinese",
  "canvas": {"size": "1024x1024", "aspect": "1:1"},
  "style": "clean warm WeChat health education card",
  "plate": [
    {"label": "蔬果", "value": 40, "color": "green"},
    {"label": "蛋白质", "value": 30, "color": "orange"},
    {"label": "全谷物", "value": 30, "color": "yellow"}
  ],
  "tips": ["先吃蛋白和蔬果", "主食优先全谷物", "少喝含糖饮料"],
  "footer": "科普示意，不替代医生或注册营养师建议"
}
JSON
```

### Step 2 — run `codex exec`

Set an output path, then ask Codex to use imagegen and copy the generated PNG to that path:

```bash
OUT="$PWD/generated/nutrition-breakfast.png"
mkdir -p "$(dirname "$OUT")"

codex exec --cd "$PWD" "Use the system imagegen skill. Read /tmp/nutrition-spec.json. Generate one polished square nutrition infographic PNG with the built-in image_gen tool. Preserve the provided Chinese text, numbers, units, and disclaimer as much as image generation allows. After generation, locate the newest PNG under \$CODEX_HOME/generated_images/ and copy it to '$OUT'. Print only the final output path. Do not use the OpenAI API fallback script."
```

Important details for agents:

- `codex exec` runs a separate Codex session. Put all necessary instructions in the quoted task; do not rely on hidden context.
- Tell Codex explicitly to use the **system imagegen skill** and the built-in `image_gen` tool.
- Tell Codex explicitly to copy the final PNG from `$CODEX_HOME/generated_images/...` to your desired `OUT` path. Otherwise the image may remain in Codex's generated-images cache.
- Escape `$CODEX_HOME` as `\$CODEX_HOME` inside double-quoted shell strings if you want the child Codex session, not your current shell, to expand it.
- If exact text/numbers are critical, inspect the result. GPT image models can distort text; switch to the Pillow fallback for exact chart labels.

### Step 3 — inspect and deliver

```bash
file "$OUT"
# Optional: use your environment's image viewer or vision tool to inspect legibility.
```

Then send/attach the generated PNG on the same channel where the human asked, if the channel supports media.

## OpenAI API fallback

Use only when an explicit API path is desired and credentials are available:

```bash
python3 scripts/generate_gpt_image.py \
  --spec /tmp/spec.json \
  --out /tmp/nutrition-gpt.png
```

or:

```bash
python3 scripts/generate_gpt_image.py \
  --prompt-file /tmp/nutrition-prompt.txt \
  --out /tmp/nutrition-poster.png \
  --model gpt-image-1 \
  --size 1024x1024 \
  --quality high
```

## Pillow fallback

Use the deterministic renderer when exact chart proportions, repeatable output, no API call, or a fast draft matters:

```bash
# Render demo directly with Pillow
python3 scripts/render_nutrition_infographic.py \
  --out /tmp/demo-nutrition.png

# Render a custom spec with fixed canvas size
python3 scripts/render_nutrition_infographic.py \
  --spec /tmp/spec.json \
  --out /tmp/card.png \
  --width 1400 --height 1000

# Write a starter spec
python3 scripts/render_nutrition_infographic.py \
  --write-demo-spec /tmp/demo-nutrition.json
```

## Prompt template for GPT image

Use this structure when generating through imagegen:

```text
Create a polished nutrition science infographic.

Purpose/channel: <...>
Audience: <...>
Language: <...>
Canvas/aspect: <...>
Style: <...>

Main message: <...>
Title: <exact title>
Subtitle: <exact subtitle>
Required text/numbers: <list exact phrases, values, units>
Visual structure: <plate / metric cards / comparison bars / tips / footer>
Data/source status: <sourced or illustrative>
Disclaimer/footer: <...>

Design constraints:
- Make all labels crisp, high contrast, and readable.
- Preserve all provided numbers, units, and wording exactly.
- Avoid disease-treatment promises, miracle claims, or unsupported medical advice.
- Prefer clean information design over photorealistic food photography unless requested.
```

## Content safety and accuracy checklist

Before sending a nutrition infographic:

- Are values explicitly sourced or clearly marked as illustrative?
- Does the output avoid disease-treatment promises and one-size-fits-all prescriptions?
- Does the footer warn that special groups should consult clinicians/dietitians?
- Are units correct (`g`, `mg`, `kcal`, `%`, `g/kg`, etc.)?
- Is the target audience clear?
- Is the language exactly what the human requested?
- If the human asks for medical personalization, ask for permission to keep it general or request professional constraints; do not invent clinical advice.

## Files

- `scripts/generate_gpt_image.py` — OpenAI API fallback / explicit API path using the OpenAI Python SDK; not the default Codex built-in imagegen route.
- `scripts/render_nutrition_infographic.py` — deterministic PNG renderer built with Pillow.
- `assets/demo-balanced-plate.json` — Chinese demo spec for a balanced-plate infographic.
- `assets/demo-balanced-plate.png` — deterministic demo output.

## Publishing notes

When publishing this skill outside one agent:

- Include `SKILL.md`, `README.md`, `scripts/`, `assets/demo-balanced-plate.json`, and at least one small demo output.
- Do not include transient `__pycache__` folders.
- Generated examples are optional; keep repo size reasonable.
- After copying into a LingTai `.library/custom/` or `.library_shared/`, run the skills validator and refresh the agent.

---
> Source: [huangzesen/nutrition-infographic-skill](https://github.com/huangzesen/nutrition-infographic-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
