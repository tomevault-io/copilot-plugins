## cc-resume

> Build single-page A4 Chinese-language resumes as HTML/CSS, then export to PDF via headless Edge/Chrome. Covers Sino-Western typography pairing, user-driven theme colors, strict A4 fit (210mm × 297mm), and pitfalls when iterating with the user.


# Single-Page A4 Chinese Resume — HTML → PDF

## When to use

The user wants a polished CV / 简历 they can print or attach to applications. They will iterate visually: the layout, color, fonts, bullet density, and section ordering will all change multiple times before they're satisfied. Optimize for fast, low-friction iteration over a "perfect first draft."

## Workflow at a glance

1. **Read the source** (`*.txt`, `*.md`, `*.docx`, `*.pptx`, `*.pdf`) and extract every fact verbatim. Don't paraphrase or invent.
2. **Extract the embedded photo** from the source if there is one — see [Extracting the portrait](#extracting-the-portrait) below.
3. **Build one HTML file per person.** Inline `<style>` — no external CSS file. Each resume is self-contained.
4. **Render to PDF** with headless Edge (Chrome works too):
   ```powershell
   & $edge --headless=new --disable-gpu --no-pdf-header-footer --no-sandbox `
     --print-to-pdf="<out>.pdf" "file:///<absolute-path-to-html>"
   ```
5. **Verify page count** by parsing the PDF — see "Page-count check" below.
6. **Iterate.** Each user remark is usually one of: change a fact, change a color, change a font, change a bullet structure, or "fit on one page."

## Extracting the portrait

Most source files (`.docx`, `.pptx`, the user's exported PDF resume) embed the candidate's portrait. **Always check for an embedded photo before falling back to a `PHOTO` placeholder** — the user expects the layout to look complete on first render.

`.docx` and `.pptx` are ZIP archives. The portrait lives at:
- `.docx` → `word/media/image1.{png,jpeg}` (and image2, image3, ...)
- `.pptx` → `ppt/media/image1.{png,jpeg}`
- `.pdf` → use `pdfimages -j input.pdf prefix` (poppler-utils) to dump every embedded image

### PowerShell (Windows)

```powershell
# Treat the file as a zip and extract media/
$src  = "E:\path\to\个人简历.pptx"
$temp = "$env:TEMP\portrait-extract"
if (Test-Path $temp) { Remove-Item $temp -Recurse -Force }
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::ExtractToDirectory($src, $temp)

# pptx → ppt/media; docx → word/media
$mediaDir = if (Test-Path "$temp\ppt\media") { "$temp\ppt\media" } else { "$temp\word\media" }
Get-ChildItem $mediaDir | Format-Table Name, Length

# Copy the largest image (usually the portrait) into your project
$largest = Get-ChildItem $mediaDir | Sort-Object Length -Descending | Select-Object -First 1
Copy-Item $largest.FullName "<your-cv-folder>\photo.jpg"
```

### Bash (macOS / Linux)

```bash
src="$HOME/path/to/resume.pptx"
tmp=$(mktemp -d)
unzip -q "$src" -d "$tmp"
mediadir=$(ls -d "$tmp"/{ppt,word}/media 2>/dev/null | head -n1)
ls -laS "$mediadir"
# largest image is usually the portrait
cp "$(ls -S "$mediadir"/* | head -n1)" "<your-cv-folder>/photo.jpg"
```

### Inserting the photo

Once `photo.jpg` (or `.png`) sits next to the resume HTML:

```html
<img class="header-photo" src="photo.jpg" alt="证件照" />
```

The template's `.header-photo` style (`object-fit: contain` / `cover`) will frame it correctly at 80×110 or whatever you've set.

**Self-contained alternative — inline as data URI** (so the HTML is a single distributable file with no asset dependency):

```powershell
$bytes = [System.IO.File]::ReadAllBytes("photo.jpg")
$b64   = [Convert]::ToBase64String($bytes)
"data:image/jpeg;base64,$b64" | Set-Clipboard
# paste into <img src="..." />
```

Trade-off: inline data URI bloats HTML (~250 KB for a typical portrait) but lets the user email a single `.html` without a missing-image broken icon.

### When there's no photo

If the source has no embedded image, **ask the user**: *"源文件里没有证件照，你提供一张吗？或者先用占位符？"* Don't silently render a placeholder — Chinese resumes default to having a portrait, and skipping it without checking is usually wrong.

If the user opts for a placeholder (e.g. they're still picking the photo), use a soft gradient with a ghosted "PHOTO" wordmark — looks intentional rather than broken:

```html
<div class="header-photo">PHOTO</div>
```
```css
.header-photo {
  background: linear-gradient(135deg, #d1d5db 0%, #e5e7eb 100%);
  display: flex; align-items: center; justify-content: center;
  font-family: "Newsreader", serif;
  font-style: italic;
  font-size: 8pt;
  color: #f3f4f6;
}
```

## Project layout

```
cv/
  resume_<personA>.html         ← one HTML per candidate
  resume_<personB>.html
  resume_<personA>.pdf / resume_<personB>.pdf
  photo_<personA>.jpg, photo_<personB>.png   ← passport-style portraits
  logos/                                     ← school / company logos as round masks
    <school1>.png, <school2>.png, ...
  fonts/                                     ← woff2 fonts, embedded via @font-face
    noto-sans-sc-{400,500,600,700}.woff2
    noto-serif-sc-{400,600,700}.woff2
    ibm-plex-sans-{400,500,600,700}.woff2
    newsreader-{400,400-italic,600}.woff2
  raw_<personA>.txt / raw_<personB>.md       ← original unedited content
```

Keep raw source files alongside the HTML — when the user says "go back to the original wording" you'll need them.

## Typography

**Never rely on PingFang SC** — it doesn't exist on Windows, and headless Edge will fall back to a system font silently, breaking weight rendering. Always embed via `@font-face`.

Stack:

| Role | Font | Weights | Notes |
|---|---|---|---|
| Latin sans body | IBM Plex Sans | 400 / 500 / 600 / 700 | Pairs cleanly with CJK; corporate but not generic |
| CJK body | Noto Sans SC | 400 / 500 / 600 / 700 | PingFang's free open-source equivalent |
| Latin serif citation | Newsreader | 400 / 400-italic / 600 | For paper citations only |
| CJK serif (optional) | Noto Serif SC | 400 / 600 / 700 | If user wants 报刊感 / editorial display |

CDN: `https://cdn.jsdelivr.net/npm/@fontsource/<family>@latest/files/<file>.woff2`

Order in `font-family`: **Latin sans first**, CJK fallback second:
```css
font-family: "IBM Plex Sans", "Noto Sans SC", "PingFang SC", "Microsoft YaHei", sans-serif;
```
The browser uses the first font that has the glyph — IBM Plex covers Latin/digits, Noto Sans SC covers CJK.

**Use `pt`, not `px`, for typography** in print-bound HTML. PDF renders pt natively; px is sensitive to DPI rounding.

Body baseline: 10–10.5pt. Section heads: 11.5–12pt. Name: 20–22pt with `letter-spacing: 0.18em`.

### CJK name spacing

Chinese personal names are 2–4 characters, but most templates assume 3. Without normalization, a 2-character name (李华) sits visually narrower than a 3-character name (王小明) at the same point size, and the header looks unbalanced.

Convention:

| Char count | Treatment | Example |
|---|---|---|
| 2 | Insert one full-width space (U+3000) between the two chars so the rendered width matches a 3-char name | `<div class="header-name">李　华</div>` |
| 3 | Render as-is, **no extra spaces between characters**. `letter-spacing` already handles tracking | `<div class="header-name">王小明</div>` |
| 4+ | Render as-is | `<div class="header-name">欧阳修远</div>` |

**Don't add ASCII spaces between every character** (`王 小 明`). Combined with `letter-spacing: 0.18em` it doubles the gap and produces obvious visual sprawl.

## Theme color systems

Pick **one** brand-aligned accent and stick with it. Define it as a CSS constant the entire stylesheet reuses (one primary `--accent`, one darker `--accent-deep`, one tinted `--tag-bg`).

**Always ask the user for the accent color first.** Don't guess from school/employer affiliation — the user may want a different scheme than their institution's official color. Possible prompts:

- "想要哪个色系？比如学校 / 单位的官方色，或者具体 hex / RGB？"
- Offer 3–4 broad directions to pick from (cool blues, deep purples, school reds, warm oranges/terracotta) so the user can pick fast.

Once the user gives you a primary hex, derive:
- `--accent-deep` ≈ 60–70% lightness of primary (use for section titles & strong text)
- `--tag-bg` ≈ a 6–8% tint of primary on white (use for award-tag pills)

Don't use rainbow icon colors — the user will reject "图标五颜六色". Use **one** accent everywhere; add subtle variation with neutral grays for secondary/tertiary text.

## Layout skeleton

```
.page (210mm × min-height: 297mm)
├─ .header  (photo + name + track + meta + contacts)
└─ .content (sections)
   ├─ section: 教育背景 / Education
   ├─ section: 实习经历 / Internships
   ├─ section: 科研经历 / Research
   ├─ section: 竞赛经历 / Competitions  (2-col grid)
   ├─ section: 荣誉奖项 / Honors  (single line)
   └─ section: 个人陈述 / Statement
```

For non-research majors swap in 项目经历 / 实践工作 / 语言能力 / 技能 instead.

### Section title pattern (uniform accent)

```html
<div class="section-title">
  <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="var(--accent)" stroke-width="2">…</svg>
  教育背景
</div>
```
Border-bottom 1.5px solid in primary accent; font-size 11.5–12pt; weight 700; letter-spacing 0.06em.

### Bullet markers

Avoid:
- `list-style: disc` — looks puny in CJK at small sizes
- Black ▪ ▍ Unicode characters — render unevenly across fonts and look like "black squares" on Windows

Prefer (CSS-rendered, accent-colored):
```css
.entry-bullets li::before {
  content: "";
  position: absolute;
  left: 1px;
  top: 0.62em;
  width: 6px;
  height: 6px;
  background: var(--accent);
  clip-path: polygon(0 0, 100% 50%, 0 100%);  /* small triangle */
}
```
Other clean options: `border-radius: 50%` (filled dot), `transform: rotate(45deg)` (diamond). Pick one and use it everywhere.

### Sub-heading vertical bar

For "▍ 子项目名" — replace the Unicode char with a CSS bar:
```css
.sub-heading { position: relative; padding-left: 11px; }
.sub-heading::before {
  content: "";
  position: absolute;
  left: 0; top: 0.18em; bottom: 0.22em;
  width: 3px;
  background: var(--accent);
  border-radius: 1.5px;
}
```

### Logos

Round masks at 18×18px work well in `entry-header`:
```css
.entry-logo {
  width: 18px; height: 18px;
  border-radius: 50%;
  object-fit: cover;
  flex-shrink: 0;
  background: #fff;
}
```
Some users will say "全部 logo 不要" — be ready to remove every `<img>` and tighten left margins to 0 (顶格).

### Awards grid (2-column)

**Use a flex-item-per-award layout — not cross-row column alignment.** A common mistake is laying out 4 columns (marker / name / tag / role) × 2 sides as an 8-column grid. This breaks two ways:

1. **Long badge wraps.** A fixed `56–64px` tag column can't fit 4-char badges like "国家级立项" or "国一等奖" — they wrap to two lines and blow up the row height.
2. **Odd item count looks lopsided.** 5 awards in an 8-column grid leaves the bottom-right cell empty *and* forces all rows to share column widths globally, so a single long badge in row 2 leaves whitespace in rows 1 and 3.

The fix: outer grid is just `1fr 1fr` for the 2-column page split; each award is its own flex container that sizes itself.

```css
.award-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  column-gap: 24px;
  row-gap: 2px;
  font-size: 9.5pt;
  line-height: 1.65;
}
.award-item {
  display: flex; align-items: baseline; gap: 6px;
  min-width: 0;
}
.award-item .aw-marker { color: var(--accent); font-size: 9pt; flex-shrink: 0; }
.award-item .aw-name   { flex: 1; min-width: 0; }
.award-item .aw-role   { color: #6b7280; white-space: nowrap; flex-shrink: 0; }
.award-tag {
  display: inline-block;
  background: var(--tag-bg); color: var(--accent-deep);
  font-size: 8.5pt; font-weight: 600;
  padding: 0 6px; border-radius: 3px;
  white-space: nowrap; flex-shrink: 0;     /* never wrap, never shrink */
}
```

```html
<div class="award-grid">
  <div class="award-item">
    <span class="aw-marker">▸</span>
    <span class="aw-name">XXXX 全国 XX 大赛</span>
    <span class="award-tag">国家级立项</span>
    <span class="aw-role">第一负责人</span>
  </div>
  ...
</div>
```

`white-space: nowrap` on `.award-tag` and `.aw-role` is mandatory — without it, badges and role labels wrap and the entry height jumps. `flex-shrink: 0` keeps them from being compressed when the name is long.

### Timeline list (date | event grid)

Replace ugly `cols-2` wrapping bullets for a 6+ event list:
```css
.timeline { display: grid; grid-template-columns: 60px 1fr; column-gap: 14px; row-gap: 3px; }
.timeline .tl-date { font-size: 9pt; font-weight: 600; color: var(--accent); font-variant-numeric: tabular-nums; }
.timeline .tl-event { font-size: 9.5pt; line-height: 1.55; border-left: 1.5px solid #d8d4e2; padding-left: 12px; }
```

## Strict A4 fit (1 page)

A4 at 96 dpi ≈ 210 × 297 mm ≈ 794 × 1123 px. Total content height including header must stay under 1123 px.

Don't use `max-height: 297mm; overflow: hidden` — it silently clips content; the user will say "PDF 不全." Instead, keep `min-height: 297mm` only and trim until it actually fits.

When over by 1 page break, the levers (largest payoff first):

| Lever | Saves | Notes |
|---|---|---|
| `.section { margin-bottom }` 14 → 8 | ~30 px | 5 sections × 6px |
| `.entry { margin-bottom }` 10 → 7 | ~12 px | |
| `.entry-bullets li { line-height }` 1.75 → 1.6 | ~25 px | 12+ bullets |
| Header padding 14/12 → 8/7 | ~12 px | |
| Header photo 92×128 → 76×104 | ~24 px | |
| Header `.name` 22pt → 20pt | ~6 px | |
| Merge two short sections into one | ~30 px | e.g. 语言 + 技能 |
| Drop 个人陈述 / 荣誉 sections | ~50 px each | **Ask first — user often wants these.** |

When **under** (bottom whitespace), reverse the same levers — increase line-height to 1.75–1.85, increase section margins, increase header padding. Aim for the page to *just* fill, not crammed and not airy.

## PDF generation (Windows)

```powershell
$edge = "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
$dir  = "<your-cv-folder>"

# Always kill stale Edge processes — they hold cache locks.
Get-Process msedge -ErrorAction SilentlyContinue | Stop-Process -Force -ErrorAction SilentlyContinue
Start-Sleep -Seconds 3

$out = "$dir\resume.pdf"
Remove-Item $out -Force -ErrorAction SilentlyContinue
$p = Start-Process -FilePath $edge -ArgumentList @(
  "--headless=new","--disable-gpu","--no-pdf-header-footer","--no-sandbox",
  "--print-to-pdf=$out",
  "file:///$($dir.Replace('\','/'))/resume.html"
) -PassThru -NoNewWindow
$p.WaitForExit(30000) | Out-Null
```

### Page-count check (no extra dependency)

```powershell
function Get-PdfPageCount($path) {
  $b = [System.IO.File]::ReadAllBytes($path)
  $t = [System.Text.Encoding]::ASCII.GetString($b)
  return [regex]::Matches($t, '/Type\s*/Page[^s]').Count
}
```
1 page = good. 2 pages = trim. The regex avoids matching `/Type /Pages` (the page tree root).

### File-lock workaround

Windows PDF viewers (Adobe, WPS Office, Foxit) hold the file open. When `--print-to-pdf` fails with `Failed to write file: ... 另一个程序正在使用此文件`:

1. Write to a versioned filename (`resume_v2.pdf`) first.
2. Try to delete + rename in a `try/catch`; if the original is still locked, leave the v2 in place and tell the user *"please close the PDF viewer; v2 is the latest."*

```powershell
try {
  Remove-Item $original -Force -ErrorAction Stop
  Move-Item $tmp $original
} catch {
  Write-Output "PDF locked by viewer; $tmp is the latest"
}
```

## Iteration patterns

The user's edits cluster into four shapes. Recognize them fast.

### 1. Content correction
*"师从 X 助理教授（不是 Y 教授）"* / *"源域、目标域用英文"* / *"21 中共党员"*

→ One targeted `Edit`. Don't restyle.

### 2. Bullet restructuring
*"拆点"* / *"这些不好看"* / *"工整一点"*

The user oscillates between **atomic** (one bullet per fact) and **merged** (one bullet per technology layer). Don't over-split:
- 1 atomic point per fact = "拆得太碎"
- 3+ semicolon-joined sub-clauses per bullet = unreadable
- **Sweet spot: 2–3 bullets per project, each with parallel verb-led structure** (完成 / 构建 / 搭建 / 微调 / 部署)

### 3. Theme / style swap
*"主题色换成 #XXXXXX"* / *"图标统一"* / *"字体换苹方"*

→ Use `Edit` with `replace_all: true` for color hex codes. Section icons need individual edits (each SVG `stroke=`).

### 4. Page-fit complaints
*"下面空了"* / *"没合并到一面"* / *"内容不全"*

→ Run page-count check. Then apply levers from the A4-fit table above. **Don't remove sections without asking** — the user will tell you which content matters.

## Content fidelity

- **Don't drop content** to make A4 fit — ask the user which sections to compress.
- **Don't infer personal facts** (age, 民族, 党员身份) when the source omits them. Ask first.

You *may* adjust:
- Spacing, alignment, half-width spaces between Chinese and Latin/digits (`AI 项目` not `AI项目`).
- Curly vs. ASCII quotes for typographic consistency.
- Capitalization of proper nouns when clearly a typo (e.g., `mysql` → `MySQL`) — but only after confirming with user.

## Common content pitfalls

- **Numerals**: prefer half-width Latin digits (`790 万` not `790万`) for tabular alignment with `font-variant-numeric: tabular-nums`.
- **Quotes**: Chinese curly quotes "..." for Chinese terms; ASCII quotes only inside English code/identifiers.
- **Half-width punctuation around English**: `XX 工程` (space) is more readable than `XX工程` for mixed text.
- **Patent / paper / book titles in Chinese context use 《》** (book-title marks), not English italics or quotes. Apply to:
  - 中国专利: `《一种基于 XX 的 XX 装置》`
  - 中文期刊文章: `《物理学报》上的《XXX 研究》`
  - 中文著作 / 教材 / 报告: `《习近平谈治国理政》`
  English-language paper titles stay in italics inside `paper-cite` blocks — don't put `《》` around English titles.

## Section ordering by major

| Major / target | Section order | Notable accents |
|---|---|---|
| 工科 / IC / AI 研究方向 | 教育 → 实习 → 科研 → 竞赛 → 荣誉 → 个人陈述 | Paper citations in `paper-cite` block; technical em-tags |
| 翻译 / 国际事务 / 文科 | 教育 → 项目经历 → 实践工作 → 语言能力 → 技能 | Timeline grid for口译/活动 records; certificate chips |
| 产品 / 商科 | 教育 → 实习 → 项目 → 竞赛 → 技能 | Project-name sub-headings; outcome metrics in `<strong>` |
| 创业 / Entrepreneurship | 教育 → 创业项目 → 实习 → 团队管理 → 竞赛 → 个人陈述 | "项目名 · 阶段（种子轮 / Pre-A / A 轮）"; revenue / user-count metrics |
| 公务员 / 选调生 / 体制内 | 教育 → 政治面貌 → 实践锻炼 → 学生工作 → 实习 → 荣誉 → 个人陈述 | Lead with 党员身份; emphasize 思想觉悟 / 党课 / 志愿服务; lighter on commercial internships |
| 金融 / 投行 / 券商 / 量化 | 教育 → 证书 → 实习 → 项目 → 竞赛 → 技能 | CFA / CPA / FRM 证书前置; deal sheet & ticker tags; bilingual is common |

### Preset-specific guidance

**创业（entrepreneurship）**
- Replace 实习 as the lead with **创业项目** — each entry needs: company / role-as-founder / fundraising stage / team size / KPI moved.
- Use the `award-tag` chip to mark stage: `种子轮` / `Pre-A` / `A 轮` / `已退出`.
- Metrics belong in `<strong>`: MAU, GMV, 营收, 融资额. If pre-revenue, lead with team milestones (人员扩张到 X 人, 完成 MVP).

**公务员 / 选调生**
- The header `header-basic` line **must** include 政治面貌 (中共党员 / 预备党员). Don't redact this in a public-facing output.
- Add a dedicated 实践锻炼 section above 实习 — covers 三下乡, 支教, 志愿服务, 社会实践调研. Each entry is a one-line `entry-desc`, no bullets needed.
- 学生工作 deserves its own section (not merged with 实践工作) — list 团委 / 学生会 / 党支部 roles in chronological order.
- Tone: avoid Latin technical jargon; prefer 中文术语. The skill should NOT auto-translate Chinese terms to English here.

**金融 / 投行 / 量化**
- Lead with 证书 section (right after 教育). Use the `award-tag` chip per certificate: `CFA L2` / `FRM Part 1` / `CPA` / `Series 7`.
- Deal experience: in 实习 entries, list specific deals/transactions as bullets (匿名化时用 `某 A 股上市公司` / `某美元基金 LP`).
- Numbers everywhere. Each bullet should ideally end with a metric: 募集金额 / 估值 / IRR / Sharpe / 模型回测收益.
- Bilingual (中英对照) version is often required for foreign-IB applications — see [`examples/template-bilingual.html`](examples/template-bilingual.html).

## What NOT to do

- ✗ Bold redesigns when the user says "在原先的基础上优化" — they want incremental improvements, not a new design language.
- ✗ Adding inferred personal info (age, party affiliation) without confirming.
- ✗ Removing sections to fit A4 without asking.
- ✗ Using `max-height + overflow: hidden` to "force" 1 page — content gets silently clipped.
- ✗ Using `list-style: disc` or `▪`/`▍` Unicode block characters for markers.
- ✗ Using PingFang SC in the font stack on Windows — silently falls back.
- ✗ Polling / repeatedly retrying when a PDF write fails: it's almost always a viewer holding the lock; tell the user.

## Final checklist before declaring done

- [ ] Page count = 1 (verified by regex)
- [ ] No `max-height: 297mm` clip
- [ ] All theme colors consistent (one accent + one darker variant)
- [ ] All section icons same color (or intentionally varied — confirm with user)
- [ ] All facts traceable to the raw source file (no invented content)
- [ ] Embedded fonts loading (no `PingFang SC` without `@font-face`)
- [ ] Bottom of page is filled (not airy) and content not cramped
- [ ] PDF viewer closed before final write to avoid file-lock confusion

---
> Source: [2084413277/cc-resume](https://github.com/2084413277/cc-resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
