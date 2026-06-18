## claude-skill-patent-writing-chinese

> Generate Chinese patent specifications (中国专利说明书) as well-formatted Word documents (.docx) following the standard structure (技术领域 / 背景技术 / 发明内容 / 附图说明 / 具体实施方式). Provides reusable Python helpers for OMML inline equations, FangSong + Times New Roman fonts, black-headed Heading 1/2 styles, equation-aware tables, embedded patent-style block diagrams (parallelograms for data, dashed module groups, orthogonal arrow routing, pure black-and-white). Use this skill when the user asks to write a Chinese patent, draft 专利说明书, convert a research paper into patent format, or wants help producing a CNIPA-style patent application document.


# Patent Writing - Chinese (中国专利说明书生成)

This skill helps you produce professional Chinese patent specifications as Word `.docx` files. It encapsulates years of trial-and-error around the messy parts: OMML inline math, Chinese-friendly fonts, Word's blue heading default, parallelogram data shapes, and clean orthogonal flowchart routing.

## When to use this skill

Trigger on requests like:
- "帮我写一份专利说明书"
- "把这篇论文改写成专利"
- "Write a Chinese patent for ..."
- "Convert this methodology into 中国专利"
- "Generate a 专利申请书 / 发明专利"
- Any request to produce a Chinese patent docx with figures and equations

## What this skill provides

```
patent-writing-chinese/
├── SKILL.md                       (this file)
├── scripts/
│   ├── patent_helpers.py          PatentDoc class + OMML builders + table/figure helpers
│   ├── figure_helpers.py          B&W matplotlib primitives (rect, para, diamond, ...)
│   └── patent_template.py         Skeleton script you can copy and customize
└── examples/
    └── example_patent.py          Tiny working example
```

The two helper modules are pure infrastructure — they contain no patent-specific content. You import them and call their functions to assemble any patent you want.

## Workflow

### Step 1: Understand the user's request

Ask (or infer) the following before writing code:
- **Title** of the invention (中文)
- **Technical field** (one paragraph for 技术领域)
- **Background**: what existing methods exist, what's wrong with them
- **Core innovations**: 2–4 bullet points the patent will claim
- **Detailed steps (S10/S20/...)**: the actual algorithm/method to describe
- **Equations needed**: list each formula
- **Figures needed**: usually 1 overview + 2–4 module-level diagrams
- **Tables**: if any, what columns/rows

If the user provides a paper or document, read it first and extract these elements yourself.

### Step 2: Set up the build script

Copy `scripts/patent_helpers.py` and `scripts/figure_helpers.py` into the user's working directory (or have the build script add them to `sys.path`). Then create a `build_patent.py` that imports them.

Use `scripts/patent_template.py` as a starting skeleton — it contains the standard 5 sections and `H()`, `MP()`, `EQ()`, `FIG()`, `TABLE()` calls you can fill in.

### Step 3: Define equations early

Use the OMML builders (`mr`, `m_sub`, `m_sup`, `m_subsup`, `m_frac`, `m_delim`, `m_nary`, `m_norm`, `m_abs`, `m_bar`) to build display equations as XML strings. Wrap with `wrap()` for display blocks (used by `EQ()`) or with `imath()` for inline equations (used by `MP()` and `TABLE()`).

Always define a small library of inline symbol constants up front, e.g.:
```python
S_xt    = imath(m_sub(mr("x"), mr("t")))
S_phis  = imath(m_sub(mr("\u03c6"), mr("s")))
S_LBC   = imath(m_sub(mr("L"), mr("BC")))
```
Then reuse them across paragraphs and tables.

### Step 4: Write paragraphs with `MP()`, never plain `P()` for math

**CRITICAL**: never write `'风格标签 s_t'` or `'L_BC'` as plain text in `P()`. The literal `_` will appear as an underscore character in Word, NOT as a subscript.

Instead, split the text and embed the inline equation:
```python
doc.MP('其中，', S_xt, '为自车特征向量，', S_phis, '为编码器的可学习参数。')
```

`MP()` accepts an arbitrary mix of text strings and inline OMML XML strings; it routes each segment to either a `w:r` run or an `m:oMath` element automatically.

### Step 5: Build figures

Use `figure_helpers.py` for diagrams. Conventions:
- **Rectangles** (`rect`): processing modules
- **Parallelograms** (`para`): data items / data flow boundaries
- **Diamonds** (`diamond`): decision nodes
- **Dashed rectangles** (`dashed_rect`): module groups (entire pipelines or sub-systems)
- **All B&W**: white fill, black border, no color
- **Orthogonal arrows only**: never diagonal — use `vline`/`hline` for path segments and `arrow_to`/`connect_h`/`connect_v` only at the FINAL destination
- **Arrow heads belong only at destinations**, never in the middle of a multi-segment path

When a path goes through a junction (fork or merge), use plain `vline`/`hline` for the segments and put the arrow head only on the final box edge.

### Step 6: Validate before delivery

After running the build script, unzip the docx and check:
1. `word/document.xml` parses as valid XML
2. `<m:oMathPara>` count is 0 (those would cause Word to error if mixed with text)
3. All `<m:oMath>` elements live in `w:p` paragraphs (some inline alongside `w:r`, some standalone for display)
4. No `w:color` element has `themeColor="..."` blue references (headings must be RGB 000000)

Open the docx and visually verify the figures fit inside their dashed groups, no text overflows boxes, and equations render as italic math.

## Patent document structure (Chinese conventions)

Every patent has these five Heading 1 sections, in order:

1. **技术领域** — 1 paragraph naming the technical field and the specific aspect this invention addresses
2. **背景技术** — 3–5 paragraphs reviewing prior art, identifying its limitations
3. **发明内容** — overview of the invention + numbered steps (步骤一/步骤二/...) + benefits comparison
4. **附图说明** — list of figures, one line each ("图1 本发明总体结构框图")
5. **具体实施方式** — detailed implementation, indexed by S10/S11/S12/S20/... with all equations, tables, and embedded figures

Title is centered, 16pt, bold, NOT a Heading style. Body paragraphs use a 0.74cm first-line indent. Headings should be black (the helper overrides Word's default blue).

## Common pitfalls

1. **`oMathPara` mixed with `w:r`** → Word reports "this file contains content that can't be read". Fix: never put a display math block in the same paragraph as a text run. Use `imath()` (returns `oMath` only) for inline math, `wrap()` (returns `oMathPara`) ONLY when nothing else is in the paragraph.

2. **Plain text math symbols** (`x_t`, `c_t^*`, `L_BC`) → render as literal underscored text. Always use inline OMML via `MP()`.

3. **Blue headings** → Word's default `Heading 1` style has a blue theme color. The helper explicitly sets `RGBColor(0, 0, 0)` and removes `themeColor` attributes. Don't bypass this.

4. **Diagonal arrows in figures** → look messy and cross other elements. Always use orthogonal routing: `vline` then `hline` then `arrow_to` only at the final destination box edge.

5. **Text overflowing rectangles** → use multi-line labels (`'驾驶风格\n聚类与识别'`) and increase box height accordingly. After applying the 2/3 width scaling, very long Chinese labels need 2 lines.

6. **Parallelograms overflowing dashed groups** → the skew adds ~`0.15 × effective_width` to the leftmost/rightmost vertex beyond the bounding box. Account for this when placing parallelograms near a dashed border. Move them inward or reduce width.

7. **Figure caption right under a `FIG()` call** is set automatically — don't add a separate caption paragraph.

## Tips

- Read `scripts/patent_template.py` for the canonical structure of a patent script
- Read `examples/example_patent.py` for a tiny end-to-end example
- All helpers are documented with docstrings — read them before improvising
- For figures, study the `make_box` return dict — every box returns its edge coordinates so arrows can snap precisely
- When in doubt, prefer `vline`/`hline` (plain segments) and a single `arrow_to` at the destination over multiple chained arrows

---
> Source: [StellaQIU27/claude-skill-patent-writing-chinese](https://github.com/StellaQIU27/claude-skill-patent-writing-chinese) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
