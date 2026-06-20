## amz-product-img-gen-skill

> Use when generating Amazon listing images (main/scene/detail/A+) from user specs and reference images, and you must call wan2.7-image with strict Amazon-compliant prompt rules.


# Amazon 商品图生成（Wan2.7 可复用 Skill）

## Overview

将“用户输入 + 参考图”转成可执行的 Amazon 商品图生成方案：先做需求澄清与结构化输入，再按图片类型生成严格约束的提示词，最后通过 `wan2.7-image-skill` 调用生成并输出可直接用于 Amazon Listing 的图片。

**REQUIRED SUB-SKILL:** `wan2.7-image-skill`

## When to Use

- 需要生成 Amazon Listing 的商品主图、场景图、细节图、A+ 图
- 用户提供了 1–3 张参考图，要求保持"同一商品身份一致性"
- 需要把"营销文案"以严格规则渲染到图片里（只允许出现指定文本，且只出现一次）
- 需要对已生成的图片进行编辑修改（背景更换、颜色调整、元素增删等）
- 需要将多个参考图（多角度或多 SKU 变体）自然融合到同一场景中生成图片
- 需要把生成流程做成可复用的标准化输入 + 提示词 + 调用方式

## Outputs

- 每张图：`{编号, type, size, prompt, image_url}`
- **编号格式**：`IMG-001`、`IMG-002`...，每次生成自动递增
- **【强制要求】必须为每张生成的图片分配唯一编号**，包括默认图生产品图、融合生图和编辑图
- **【重要】image_url 必须是完整的预签名 URL，包含所有查询参数（Expires、OSSAccessKeyId、Signature 等），禁止截断或省略任何参数**
- 推荐输出集合：
  - `main` 1–2 张（白底主图）
  - `scene` 1–4 张（生活方式/场景图）
  - `detail` 1–4 张（细节特写）
  - `aplus` 1–4 张（A+ 氛围营销图）
  - `dimension` 1 张（尺寸标注图，仅当用户提供尺寸数据时生成）

## Input Intake（输入逻辑）

### 1) 必填信息（最少可开工）

- 商品名称（用于提示词的语义锚点）
- 参考图 1–3 张（本地文件 / 公网 https URL / 已上传的 oss:// URL）
- 需要生成的图片类型与数量（main/scene/detail/aplus）

### 2) 强烈建议补充（显著提升可控性）

- 类目（category）、材质（material）、尺寸（dimensions）
- 使用场景（useCase）、目标人群（targetAudience）
- 风格档案（style profile）：`minimal_modern`（现代简约） / `japanese_soft`（日系柔和） / `luxury_editorial`（轻奢质感）/ `rounded_bold`（圆润大粗体）/ `yahoma_light`（雅黑light）

当用户未提供 `dimensions` 时，必须在场景图/A+图里明确要求"真实世界尺寸与比例"，并用环境物体做比例参照，避免产品被生成成不符合常识的大小。

### 3) 文案（可选）

对 `scene/detail/aplus` 单张图可填写一条"要渲染到图里"的文案（description）。该文案必须满足：

- 单行文本
- 必须逐字逐符号完全一致（不得改写、翻译、补字、加标点）
- 图片中除该文案外不得出现任何其他文字

### 4) 文案处理检查点（MANDATORY）

**在生成提示词前必须执行以下检查：**

1. **文案识别检查**
   - 检查用户输入中是否包含文案（如"文案：xxx"、"copy: xxx"、"带xxx文案"等）
   - 提取文案内容并验证格式是否符合要求（单行文本，无特殊格式）

2. **模板选择检查**
   - 如果检测到文案：必须选择对应的"有文案"模板（如 scene 有文案模板）
   - 如果未检测到文案：必须选择对应的"无文案"模板

3. **提示词构建检查**
   - 有文案时：必须在提示词开头添加完整的"唯一文本规则"锚点
   - 有文案时：必须在提示词中明确指定文案位置和排版规范
   - 有文案时：必须在提示词末尾添加 TEXT_TO_RENDER_REPEAT 验证行

4. **执行前验证**
   - 检查生成的提示词是否包含 CRITICAL TEXT TO RENDER 块
   - 检查文案内容是否与用户输入完全一致
   - 检查是否有任何可能导致文案渲染失败的模糊表述

**错误处理：**
- 如果检测到文案但未正确构建提示词：停止执行并给出明确错误提示
- 如果文案格式不符合要求：提示用户修正文案格式
- 如果提示词中包含模糊表述（如"No text unless provided"）：替换为明确的文案渲染指令

### 5) 风格映射规则（MANDATORY）

**中文风格输入到英文风格档案的映射：**

| 中文风格输入 | 英文风格档案 | 描述 |
|-------------|-------------|------|
| 现代简约、简约现代、现代风、简约风 | `minimal_modern` | 几何无衬线字体，干净现代电商感 |
| 日系、日系风、日式、和风、日系柔和 | `japanese_soft` | 柔和手写/圆润字体，温柔轻量 |
| 高级、高级感、轻奢、轻奢质感、编辑风、杂志风 | `luxury_editorial` | 高对比现代衬线，轻奢质感 |
| 圆润大粗体、圆润粗体、胖体、可爱粗体 | `rounded_bold` | 超圆润几何无衬线，粗壮饱满，友好亲切 |
| 雅黑light、微软雅黑、细体、纤细字体 | `yahoma_light` | 几何无衬线，纤细轻量，现代简洁 |

**Typography 选择规则：**
- `minimal_modern`：使用 `modern geometric sans-serif headline, slightly condensed, clean letterforms, high readability, medium weight`
- `japanese_soft`：使用 `rounded minimal sans-serif with a soft friendly feel, warm tone, subtle` 或 `soft handwritten pen lettering, gentle thin-to-medium strokes, warm and subtle`
- `luxury_editorial`：使用 `high contrast modern serif, editorial style, refined and elegant, medium weight`
- `rounded_bold`：使用 `ultra-rounded geometric sans-serif headline, extra bold weight, soft corners, friendly and playful, high impact`
- `yahoma_light`：使用 `clean geometric sans-serif, light weight, subtle, microsoft yahei inspired, minimal and modern`

**执行要求：**
- 当用户输入中文风格名称时，必须映射到对应的英文风格档案
- 必须根据映射的风格档案选择相应的 typography
- 确保生成的文案风格与用户选择的风格一致

## Generation Settings（生图设定）

### 分辨率建议

- `main/scene/detail`：建议 `1600*1600`（或 `1K/2K`）
- `aplus`：建议宽幅，如 `1200*600`、`1464*600` 等（按 A+ 模板需要调整）

### 一致性策略

- 参考图用于“主体一致性”：在所有类型中使用同一组参考图作为输入（1–3 张）
- 提示词中强制声明“同一商品身份不改变”：形状、颜色、材质、细节不变

## Preset Scenes（预设场景库）

生成 scene 类型图片时，可从以下 11 个预设场景中选择：

| 编号 | 场景名称 | 场景描述 | 场景提示词片段 |
|------|---------|---------|--------------|
| 1 | 桌面 (Desk/Table) | 摆件、数码产品、文具等 | Place the product naturally on a clean desk surface. **Lighting: 45° side window light, softbox intensity 65%, shadow softness medium. Light temperature: 4000K neutral daylight.** Add subtle shadows. The desk can have minimal items like a notebook, pen, or coffee mug to establish scale and context. Keep the product as the clear focal point with the desk surface taking up the lower third of the frame. |
| 2 | 浴室 (Bathroom) | 洗漱用品、浴室收纳等 | Position the product in a modern bathroom setting. **Lighting: Top-down overhead fixture + ambient vanity mirror lights, 3200K warm tone, slight moisture mist post-effect for depth.** Use marble, quartz, or white tile as backdrop. Add context with items like folded towels, a mirror reflection, or small plants. Premium spa-like atmosphere with soft volumetric light. |
| 3 | 卧室 (Bedroom) | 床头装饰、纺织用品等 | Arrange the product naturally in a bedroom scene. **Lighting: Warm 2700K table lamp from side, soft fill from window (curtain diffused), shallow DOF f/2.8.** Place on a nightstand, dresser, or bed. Use soft bedding (linen or cotton), warm lamp lighting, and subtle textures like throw pillows or blankets. Cozy, restful atmosphere. |
| 4 | 厨房 (Kitchen) | 厨具、餐具、收纳等 | Set the product in a stylish kitchen environment. **Lighting: Bright overhead recessed lights (5000K daylight) + under-cabinet LED strips, clean CRI 95+.** Place on granite or marble countertops. Include context items like cutting boards, fresh ingredients, or cooking utensils. Modern culinary atmosphere with shadow-controlled directional lighting. |
| 5 | 车内 (Car Interior) | 车载配件、装饰品等 | Place the product in a premium car interior. **Lighting: Dashboard ambient LED glow (cool 6500K), rim light from sunroof, leather reflection control.** Position on the dashboard, center console, or seat. Capture leather seats, dashboard textures, and ambient dashboard lighting. Show partial interior elements like steering wheel or gear shift for context. Luxury automotive atmosphere. |
| 6 | 户外/自然 (Outdoor/Nature) | 露营装备、运动用品等 | Show the product in a natural outdoor setting. **Lighting: Golden hour sunlight (early morning 6-8am or late afternoon 4-6pm), 3500K warm, backlight rim on product, soft bokeh background with natural elements.** Place on a tree stump, rock, grass, or sand. Include natural elements like leaves, flowers, or water in the background. Fresh, organic atmosphere with environmental storytelling. |
| 7 | 健身房 (Home Gym) | 健身器材、运动装备等 | Position the product in a home gym environment. **Lighting: Bright overhead LED panel lights (6000K), uniform illumination, shallow DOF f/3.5 to separate from background.** Place on gym equipment, floor mat, or weight rack. Include dumbbells, resistance bands, or exercise bikes in soft bokeh background. Motivational fitness atmosphere with clean, energizing light. |
| 8 | 办公桌 (Office Setup) | 办公用品、显示器支架等 | Set the product in a professional office desk setup. **Lighting: Monitor glow (6500K) as main light source, desk lamp (4000K) as secondary fill, ambient office lighting balanced.** Place on desk surface with monitors, keyboard, and desk lamp visible. Use dual-screen or ultrawide monitor in background with soft bokeh. Clean, organized workspace with professional productivity atmosphere. |
| 9 | 客厅 (Living Room) | 家居装饰、沙发配件等 | Arrange the product in a modern living room. **Lighting: Warm floor lamp (3000K) as key light, ambient ceiling light (3500K) fill, window light as rim light, depth controlled by DOF.** Place on a coffee table, sofa arm, or TV console. Include items like decorative books, candles, or plants. Cozy modern interior with tasteful decor and layered lighting. |
| 10 | 旅行/露营 (Travel/Camping) | 旅行配件、户外装备等 | Show the product in a travel or camping scenario. **Lighting: Natural sunlight (golden hour preferred), environmental backlight, 3800K warm tone, adventure-mood rim lighting.** Place near a backpack, tent, or travel gear. Use outdoor natural background with sky, trees, or campfire. Adventure atmosphere with sense of journey and natural environmental storytelling. |
| 11 | 模特手持/穿戴 (Model Hands/Body) | 可手持或穿戴的产品（如手机壳、手环、项链、眼镜等） | Show the product being held in a person's hand or worn on the body. **Lighting: Natural daylight (4500K) with soft fill, slight rim light to highlight product details.** Capture close-up of hands holding the product or product worn on wrist/neck/face. Model should have natural, relaxed posture. Background should be minimal and complementary, with soft bokeh. Street style or casual lifestyle aesthetic, like candid street photography. Focus on product interaction and natural usage context. |

### 用户自定义场景

用户可指定未在预设列表中的场景名称。Agent 应：
1. 询问用户该场景的具体描述（使用环境、背景元素、光线等）
2. 将用户描述作为场景提示词片段拼接到 scene 模板中
3. 确保自定义场景片段包含：放置位置、环境描述、背景元素、光线氛围

### 场景使用方式

生成 scene 图片时：
1. 用户指定场景编号（1-11）或场景名称，或选择"自定义"并提供描述
2. **默认场景**：如果用户未指定场景，使用当前 scene 模板的默认流程（不添加预设场景片段，保持原有 scene 模板不变）
3. Agent 从预设库选取对应场景提示词片段，或使用用户自定义描述
4. 将场景片段拼接到 scene 模板中生成完整提示词
5. 确保场景片段在 prompt 中明确说明产品的放置方式和背景环境

---

## Prompt Rules（所有图片生成规则与提示词）

下述模板写成可直接投喂给 `wan2.7-image` 的自然语言指令。把其中占位符替换为实际信息即可。

### 通用占位符

- `{PRODUCT_NAME}`：商品名称
- `{CATEGORY}`：类目（可空）
- `{MATERIAL}`：材质（可空）
- `{DIMENSIONS}`：尺寸/大小描述（可空）
- `{COPY}`：要渲染到图里的文案（仅 scene/detail/aplus 可选）
- `{TYPOGRAPHY}`：字体风格描述（由 style profile 决定）

### 共同硬规则（所有类型都适用）

- 基于参考图生成，保持“同一商品身份一致性”，不得变成其他商品
- 画面必须是“真实摄影感/电商摄影感”，透视与光线要合理
- 尺寸比例必须符合真实世界：如果 `{DIMENSIONS}` 为空，则根据参考图、类目与使用场景推断合理尺寸；不要夸张放大或缩小
- 写实渲染风格：强调材质真实感（如陶瓷的温润、木质的纹理、金属的质感），符合电商图片精致度要求
- 禁止：水印、logo（除非用户明确提供并要求保留）、二维码、UI 截图、网页布局、边框、拼接分栏、白边留空面板、海报模板边框
- 如需渲染 `{COPY}`：必须严格执行“唯一文本规则”（见下）

### 唯一文本规则（仅当 `{COPY}` 非空时启用）

把以下内容放在提示词最前面作为最高优先级锚点：

```
CRITICAL CANVAS RULES (HIGHEST PRIORITY)
- Output must be true 1:1 full-bleed.
- No black bars, no letterboxing, no borders, no padding.
- Fill the entire canvas with real scene content (top and bottom edges must contain image content, not empty bands).

CRITICAL TEXT TO RENDER (EXACT, HIGHEST PRIORITY)
TEXT_TO_RENDER_BEGIN
{COPY}
TEXT_TO_RENDER_END
Rules:
- Render the text EXACTLY as provided, character-by-character, including capitalization, spacing, punctuation, and line breaks.
- Do NOT translate, paraphrase, shorten, or extend.
- Do NOT add, remove, or change any punctuation (e.g., keep ":" if present).
- Do NOT insert line breaks unless they already exist in {COPY}.
- Render it as ONE single contiguous text block (one placement). Do NOT split into multiple separate text blocks.
- Render it exactly once. Do NOT repeat any subset as a secondary headline or title.
- No other text anywhere in the image.
Typography must follow: {TYPOGRAPHY}.
```

并在提示词末尾重复一次：

```
TEXT_TO_RENDER_REPEAT (FOR VERIFICATION ONLY, DO NOT RENDER): {COPY}
```

#### 反拼接原则（关键）

当模型对“文案”理解停留在“海报拼版”（标题+副标题、多处重复、改写口号）时，通常是因为 prompt 在多个地方反复出现 `{COPY}`，导致模型把它当作“可重排的文案素材”。为避免这一点：

- `{COPY}` 只允许出现在 `CRITICAL TEXT TO RENDER` 代码块里（以及末尾的 FOR VERIFICATION ONLY 行）
- 在其他正文里不要再拼接/重复/引用 `{COPY}` 原文，只能写 “render ONLY the text from the CRITICAL block”
- 文字相关约束必须集中在同一段 `TEXT RULES`，不要分散在多处

#### 排版规范块（把任务从“拼文案”改成“做排版”）

参考“高密度信息图”类 prompt 的写法，给模型一个明确的排版工作说明与版式规范（layout spec），让它把文字当作“可控的排版对象”而不是“可随意改写的营销文案”：

```
=== TYPESETTING TASK (HIGHEST PRIORITY) ===
You are a professional typesetter (layout designer), not a copywriter.
Your job is to place the provided text EXACTLY as-is, with correct spelling.
Do NOT create alternative copy, do NOT shorten, do NOT summarize.

=== TYPESETTING SPEC ===
[TEXT_SOURCE] Use ONLY the text inside CRITICAL TEXT TO RENDER block.
[TEXT_BLOCKS] Exactly 1 text block total (no extra headline/subtitle).
[ANCHOR] One corner only: top-left / top-right / bottom-left / bottom-right.
[SAFE_MARGIN] Keep a clean margin from edges (about 6–10% of image width).
[MAX_WIDTH] Constrain text block width to 40–55% of image width to prevent auto reflow.
[ALIGNMENT] Left aligned (recommended) unless the corner requires right alignment.
[LINE_BREAKS] Follow line breaks exactly as provided; do not insert additional breaks.
[HYPHENATION] Off; do not hyphenate.
[LETTERING] Crisp, vector-like, uniform stroke, consistent kerning; no painterly texture.
[EFFECTS] Minimal: none; optional subtle shadow only if needed for readability.
```

#### 自动判定（用户不需要显式填写）

用户在会话里通常只会给“产品信息 + 参考图 + 需要的类型 + 文案”，不会显式提供版式字段。因此在 `{COPY}` 非空时，必须自动启用以下默认策略：

- 默认 CopyMode：`strict`（文字准确优先）。只有当用户明确说“要艺术体/手写/刷字/海报感”时，才切换为 `artistic`。
- 默认排版锚点：优先选择四角中“最干净的留白区”（negative space）。如无法判断，默认 `top-right`。
- 默认排版参数（当未显式覆盖时）：
  - `SAFE_MARGIN = 8%`
  - `MAX_WIDTH = 45%`
  - `ALIGNMENT = right`（当锚点为 top-right / bottom-right 时），否则 `left`

当文案较长（例如超过 45 个字符或包含多个长单词）时，为了减少错字与自动重排，允许自动做“排版用换行”，但不得改写任何字符：

- 优先断点：在冒号 `:` 后换行（保留冒号）
- 备选断点：在第一个逗号/分号/破折号后换行（保留标点）
- 兜底断点：按语义分组把句子拆成 2 行（不增删词，只插入 1 个换行）

#### 文案稳定性加固（强烈建议）

当目标模型出现拼写错误、断词、乱换字体/样式时，在提示词中追加以下“硬约束”，并在验收不通过时直接重生成（不要接受近似文本）：

- 禁止自动换行：do NOT insert line breaks, do NOT wrap words, do NOT hyphenate
- 禁止模型擅自改字形：do NOT substitute the requested typography with a generic font; keep letter shapes consistent with {TYPOGRAPHY}
- 禁止导致错字的变形：no perspective warp, no curved baseline, no heavy distortion, no 3D extrude
- 禁止重排：do NOT change capitalization, spacing, punctuation; keep ASCII/Latin letters only
- 禁止标题居中：text placement must be one corner only; center/top-center title is forbidden
- 禁止黑边/电影条：no letterboxing, no black bars; full-bleed image only
- 失败策略：if you cannot render the text exactly, render no text at all (leave it empty)

当文案较长（例如包含多个长单词或超过 45 个字符），单行更容易触发错字/自动换行。允许使用“手动换行文案”来稳定排版：在 `{COPY}` 中显式写入换行（`\n`），并要求按换行逐行渲染（仍然必须逐字一致，且不允许额外换行）。

对关键单词可加入拼写锚点（只作为提示补强，不改变实际渲染文本）：

- Spell-check anchor (do NOT add to the rendered text): MAGNETS = M-A-G-N-E-T-S

#### 验收与重试（必须执行）

- 只要出现任意错字、漏字、换行不符合 `{COPY}`、或出现黑边：直接重生成
- 重试优先级：
  1) 把 `{COPY}` 改为手动换行（`\n`）版本
  2) 缩短 `{COPY}`（保持核心卖点）
  3) 改用 `minimal_modern` 的 sans-serif typography（更稳）
  4) 更换角落位置（top-left/top-right/bottom-left/bottom-right）

#### 单次出图：把文字当“排版叠字”而不是“画面纹理”

当用户明确“不采用两段式出图”时，必须在提示词里强调文字是“后期排版叠字效果”，要求边缘锐利、字重一致、字距稳定，避免被模型当作材质纹理导致错字：

- Text must look like digitally typeset overlay, crisp edges, uniform stroke, consistent kerning
- No painterly texture inside letters, no melting letters, no distorted glyphs
- Treat text as printed typography, not as a 3D object or part of the scene geometry
- Do NOT create additional headline/subtitle blocks; exactly one text block only

#### 文案字体策略（艺术体支持）

- 有文案时，字体风格由 `{TYPOGRAPHY}` 决定，必须严格执行。
- 需要艺术体/更有设计感时，建议选 `japanese_soft` 的 typography 候选（含手写/刷字倾向，氛围更强）。
- 需要高级编辑感时，选 `luxury_editorial` 的 typography 候选（现代衬线/高对比）。
- 需要干净现代电商感时，选 `minimal_modern` 的 typography 候选（几何无衬线）。

### 1) Main（白底主图）提示词模板

```
Based on the reference image, create an Amazon white background main product photo.
The reference image shows {PRODUCT_NAME}{CATEGORY:+, a {CATEGORY}}{MATERIAL:+, made of {MATERIAL}}.
Keep the exact same product identity and appearance from the reference image (same object, same shape, same color, same material, same details). Do NOT change it into any other product.
Tight framing: the product should fill at least 85% of the image area, minimal white margins, subtle natural shadow only.
Pure white background (#FFFFFF), professional studio lighting, high-end e-commerce photography, ultra sharp focus.
No text, no logo, no watermark.
```

### 2) Scene（场景图）提示词模板

#### 无文案（不渲染文字）

```
Based on the reference image showing {PRODUCT_NAME}{CATEGORY:+, a {CATEGORY}}{MATERIAL:+, made of {MATERIAL}}, create an Amazon lifestyle product photo.

SCENE SELECTION:
- If user specifies a scene (e.g., "desk", "bathroom", "1" for desk): Select and apply the corresponding scene fragment from the Preset Scenes chapter.
- If user specifies "custom" or provides a custom description: Use the user's custom scene description as the scene fragment.
- If user does NOT specify any scene: Use the default scene template below WITHOUT adding any preset scene fragment (keep the original scene template as-is).

DEFAULT SCENE TEMPLATE (when no scene specified):
Place this exact product (same identity and appearance from reference) in a real, natural lifestyle scene with coherent lighting and perspective.
Product is the hero element, realistic environment, premium atmosphere.
Real-world scale rule: the product size must be realistic for {CATEGORY} and the usage scene; if {DIMENSIONS} is provided, follow it; otherwise infer from the reference image and common sense. Add natural scale cues (e.g., hand, mug, fridge handle) and keep proportions believable.
Full-bleed, edge-to-edge composition: the scene must fill 100% of the canvas.
No screenshot, no UI, no webpage layout, no poster mockup, no inset image, no padding, no border, no frame, no white margins, no header bar, no top title strip.
No text, no logo, no watermark.
```

#### 有文案（严格渲染文字）

把“唯一文本规则”锚点放在最前面，然后拼接以下正文：

```
Based on the reference image showing {PRODUCT_NAME}{CATEGORY:+, a {CATEGORY}}{MATERIAL:+, made of {MATERIAL}}, create an Amazon lifestyle product photo.

SCENE SELECTION:
- If user specifies a scene: Select and apply the corresponding scene fragment from Preset Scenes.
- If user provides custom scene: Use user's custom scene description.
- If no scene specified: Place product in a natural lifestyle scene with coherent lighting.

Product is hero element, realistic environment, premium atmosphere.
Real-world scale: follow {DIMENSIONS} if provided, add natural scale cues.
Full-bleed composition: scene fills 100% of canvas.
No screenshot, UI, layout, mockup, padding, border, or white margins.

Text rule: render ONLY the text from CRITICAL TEXT TO RENDER block, exactly as given.
One-block rule: render as single contiguous text block, no split or secondary headline.
No other text anywhere in the image.
Text style: digitally typeset overlay (crisp edges, uniform stroke).
Placement: corner negative space (top-left/top-right/bottom-left/bottom-right), avoid center.
No text over product's main body or key details.
Readability: place on clean space, high-contrast color, subtle shadow if needed.
Typography: {TYPOGRAPHY}; bold weight for short text.
Text scale: 14-24% of image height (22-32% for short text).
Style: flat text, no 3D, bevel, outline, heavy shadow, or perspective warp.
```

### 3) Detail（细节特写）提示词模板

#### 无文案（不渲染文字）

```
Based on the reference image showing {PRODUCT_NAME}{MATERIAL:+, made of {MATERIAL}}, create an Amazon product detail close-up.
Extreme macro / close-up shot with tight crop: focus on the most important detail area and show fine texture and craftsmanship.
The detail subject should fill 60–80% of the frame.
Do NOT do wide shot, do NOT show the full product, do NOT use a lifestyle scene composition.
Real-life background (tabletop, home interior, studio set) is allowed but must be strongly blurred with bokeh and shallow depth of field; keep the background minimal and unobtrusive.
Premium lighting, ultra sharp focus on the detail.
No text, no logo, no watermark.
```

#### 有文案（严格渲染文字）

把“唯一文本规则”锚点放在最前面，然后拼接以下正文：

```
Based on the reference image showing {PRODUCT_NAME}{MATERIAL:+, made of {MATERIAL}}, create an Amazon product detail close-up.
Extreme macro / close-up shot with tight crop: focus on the most important detail area and show fine texture and craftsmanship.
The detail subject should fill 60–80% of the frame.
Do NOT do wide shot, do NOT show the full product, do NOT use a lifestyle scene composition.
Real-life background (tabletop, home interior, studio set) is allowed but must be strongly blurred with bokeh and shallow depth of field; keep the background minimal and unobtrusive.
Premium lighting, ultra sharp focus on the detail.
Text rule: render ONLY the text provided in the CRITICAL TEXT TO RENDER block, exactly as given (including punctuation, capitalization, spacing, and line breaks). Do NOT retype or rewrite it elsewhere.
One-block rule: render it as one single contiguous text block only. Do NOT split into multiple separate text blocks. Do NOT add a secondary headline/title anywhere.
Do NOT add any other text anywhere in the image.
Text rendering rule: render text as digitally typeset overlay (crisp edges, uniform stroke, consistent kerning), not as a painterly texture.
Placement priority: top-left / top-right / bottom-left / bottom-right corner negative space.
Center placement is forbidden.
Never place text in the center for detail shots.
Do NOT place text over the key detail area or product silhouette.
No letterboxing, no black bars, no borders.
Readability rule: prioritize placing text on clean negative space and choose high-contrast text color.
If readability is still insufficient: first move the text to another corner; second adjust text color; third (only if needed) add a very subtle soft shadow behind the letters (opacity 8–12%, blur 6–10px).
Only as the LAST resort, allow a tiny semi-transparent backing strip behind the text (NOT glassmorphism): minimal rounded rectangle sized tightly to the text only, opacity 10–18%, no blur, no big box. Never use full-width banners or large panels.
Typography: {TYPOGRAPHY}; use it as a small premium label/callout; prefer medium or bold weight; ensure readability.
Text scale: about 7–11% of image height; if the text is short (<= 12 characters), make it larger (10–14%).
Style constraints: flat text, no 3D extrude, no bevel/emboss, no outline stroke, no heavy drop shadow, no perspective warp.
```

### 4) A+（营销氛围图）提示词模板

#### 无文案（不渲染文字）

```
Based on the reference image showing {PRODUCT_NAME}{CATEGORY:+, a {CATEGORY}}{MATERIAL:+, made of {MATERIAL}}, create an Amazon A+ lifestyle marketing poster.

SCENE SELECTION:
- If user specifies a scene: Select and apply the corresponding scene fragment from the Preset Scenes chapter.
- If user specifies "custom" or provides a custom description: Use the user's custom scene description as the scene fragment.
- If user does NOT specify any scene: Use the default A+ scene template below.

DEFAULT A+ SCENE TEMPLATE:
Place this exact product (same identity and appearance from reference) in a premium, aspirational lifestyle context that evokes desire and quality.
Composition: Product is the hero, occupying 50-70% of frame. Use cinematic leading lines, balanced negative space, and editorial-grade framing.
Lighting: High-end editorial lighting — soft volumetric light with subtle rim lighting to separate product from background. Moody, directional light preferred over flat ambient.
Mood: Warm, inviting, aspirational. Convey quality of life improvement. Think high-end home decor magazine or luxury brand campaign.
Depth: Shallow depth of field on background, product in sharp focus. Background should suggest a story (partially visible furniture, architectural details, natural elements).
Full-bleed, edge-to-edge: no empty side panels, no split panels, no letterboxing, no borders.
No text, no logo, no watermark.
```

#### 有文案（严格渲染文字）

把“唯一文本规则”锚点放在最前面，然后拼接以下正文：

```
Based on the reference image showing {PRODUCT_NAME}{CATEGORY:+, a {CATEGORY}}{MATERIAL:+, made of {MATERIAL}}, create an Amazon A+ lifestyle marketing poster with text overlay.

SCENE SELECTION:
- If user specifies a scene: Select and apply the corresponding scene fragment from the Preset Scenes chapter.
- If user specifies "custom" or provides a custom description: Use the user's custom scene description as the scene fragment.
- If user does NOT specify any scene: Use the default A+ scene template below.

DEFAULT A+ SCENE TEMPLATE:
Place this exact product (same identity and appearance from reference) in a premium, aspirational lifestyle context that evokes desire and quality.
Composition: Product is the hero, occupying 50-70% of frame. Use cinematic leading lines, balanced negative space, and editorial-grade framing.
Lighting: High-end editorial lighting — soft volumetric light with subtle rim lighting to separate product from background. Moody, directional light preferred over flat ambient.
Mood: Warm, inviting, aspirational. Convey quality of life improvement. Think high-end home decor magazine or luxury brand campaign.
Depth: Shallow depth of field on background, product in sharp focus. Background should suggest a story (partially visible furniture, architectural details, natural elements).
Full-bleed, edge-to-edge: no empty side panels, no split panels, no letterboxing, no borders.
Text rule: render ONLY the text provided in the CRITICAL TEXT TO RENDER block, exactly as given (including punctuation, capitalization, spacing, and line breaks). Do NOT retype or rewrite it elsewhere.
One-block rule: render it as one single contiguous text block only. Do NOT split into multiple separate text blocks. Do NOT add a secondary headline/title anywhere.
Do NOT add any other text anywhere in the image.
Text rendering rule: render text as digitally typeset overlay (crisp edges, uniform stroke, consistent kerning), not as a painterly texture.
Placement priority: top-left / top-right / bottom-left / bottom-right corner negative space.
Center placement is forbidden (no top-center headline).
Do NOT cover the product with text.
No letterboxing, no black bars, no borders.
Readability rule: prioritize placing text on clean negative space and choose high-contrast text color.
If readability is still insufficient: first move the text to another corner; second adjust text color; third (only if needed) add a very subtle soft shadow behind the letters (opacity 8–12%, blur 6–10px).
Only as the LAST resort, allow a tiny semi-transparent backing strip behind the text (NOT glassmorphism): minimal rounded rectangle sized tightly to the text only, opacity 10–18%, no blur, no big box, never a full-width banner.
Typography: {TYPOGRAPHY}; strong hierarchy with headline emphasis; prefer bold weight if the text is short.
Text scale: about 18–30% of image height; if the text is short (<= 3 words or <= 12 characters), make it very large (26–38%) and bold.
Style constraints: flat text, no 3D extrude, no bevel/emboss, no outline stroke, no heavy drop shadow, no perspective warp.
```

### 5) Dimension（尺寸标注图）提示词模板

仅当用户提供 `{DIMENSIONS}` 时生成。

```
Based on the reference image showing {PRODUCT_NAME}{CATEGORY:+, a {CATEGORY}}{MATERIAL:+, made of {MATERIAL}}, create an Amazon product dimension photo with clear measurements.

SCENE SELECTION:
- If user specifies a scene: Use corresponding scene fragment from Preset Scenes.
- If no scene specified: Use clean, minimal background that complements product.

Composition: Show product in clear setting with space for dimension lines. Use neutral background. Position product at slight angle to show all three dimensions.

Dimension annotation: Use thin black/dark gray arrows for measurements. Label length (left-right), width (front-back), height (bottom-top) with clear text. Position arrows/labels outside product silhouette. Use exact dimensions: {DIMENSIONS}. Format labels as: "17cm", "7.5cm", "7.5cm" (only numbers and units, no Chinese characters).

Product rendering: Maintain realistic material texture and lighting. Avoid technical illustration or line drawing style. No text except dimension labels.

Technical: High contrast for readability. Professional e-commerce presentation. No logo, watermark, or branding. Full-bleed composition.
```

### 6) Fusion（融合生图）提示词模板

仅当用户输入多个参考图编号或显式指定 fusion 模式时使用。

#### 融合模式主体描述层

```
MULTI-PRODUCT FUSION COMPOSITION:
- Product A: {PRODUCT_NAME_A} — from reference image {IMG_REF_1}
- Product B: {PRODUCT_NAME_B} — from reference image {IMG_REF_2}
- [Additional products as needed...]
- Each product must maintain its EXACT identity (shape, color, material, texture, details) from respective reference images.
- Do NOT blend or morph products into each other; keep each product distinct.
```

#### 融合模式构图层

```
FUSION LAYOUT & SCENE:
- Natural, organic arrangement with intentional spatial hierarchy.
- Products overlap slightly in foreground/midground to create depth.
- Slight perspective differences between products (not perfectly parallel).
- One product may be slightly forward, another receded — natural layering.
- Scene context: {SCENE_DESCRIPTION} with coherent lighting and atmosphere.
- Background: {BACKGROUND_DESCRIPTION} — soft bokeh, minimal distraction.
```

#### 融合模式光线层

```
LIGHTING FOR MULTI-PRODUCT:
- Unified lighting direction across all products (45° side light preferred).
- Consistent color temperature (4000K neutral daylight or scene-appropriate).
- Soft shadows that ground each product realistically.
- Rim light on product edges to separate from background and each other.
- No conflicting light sources between products.
```

#### 融合模式构图与技术层

```
COMPOSITION & CAMERA:
- Wide-angle or standard lens perspective (24-50mm equivalent).
- Large depth of field to keep all products in acceptable focus.
- Rule of thirds applied to product arrangement.
- Products occupy 60-75% of frame total.
- Negative space for breathing room.
- Slight upward angle (10-15°) for premium product presentation.
```

#### 融合模式风格与输出层

```
STYLE & QUALITY:
- Realistic photographic rendering, not illustration or CGI.
- Hyper-real material quality: ceramic warmth, wood grain, metal reflections.
- Clean, professional e-commerce aesthetic.
- No text unless {COPY} is provided (see text rules below).
- No logos, watermarks, or UI elements.
- Full-bleed, edge-to-edge composition.
```

#### 统一文案规则（如有文案）

把"唯一文本规则"锚点放在最前面（与现有规则一致），然后拼接：

```
TEXT OVERLAY (if {COPY} provided):
- Render text exactly once using CRITICAL TEXT TO RENDER block.
- Placement: top-right or bottom-right corner, away from product overlap zones.
- Typography: {TYPOGRAPHY} from style profile.
- Scale: 18-24% of image height, bold weight.
- Ensure text does NOT overlap or cover any product's key detail.
```

## Style Profile（风格档案 → Typography）

目的：当需要渲染 `{COPY}` 时，指定更稳定的“字体气质”以减少模型胡乱出字形的概率。

- `minimal_modern`：现代几何无衬线、瑞士现代风格、克制、可读性优先
- `japanese_soft`：柔和手写/圆润无衬线、温柔、轻量
- `luxury_editorial`：高对比现代衬线、编辑感、克制但高级
- `rounded_bold`：圆润大粗体、可爱友好、粗壮饱满
- `yahoma_light`：雅黑light、纤细轻量、现代简洁

当 `{COPY}` 非空时，先确定你需要的文字风格：

- 需要艺术体/更有设计感：优先 `japanese_soft`
- 需要高级编辑感：优先 `luxury_editorial`
- 需要干净现代电商感：优先 `minimal_modern`
- 需要更突出的视觉效果：优先 `rounded_bold`
- 需要更清楚的文字感或文艺感：优先 `yahoma_light`

艺术体通常更容易出现错字或自动换行，优先用更短的文案，并用“文案稳定性加固”规则做验收与重试。

当用户不提供明确偏好时，默认按 `CopyMode=strict` 处理（文字准确优先）：

- `strict`：优先 `minimal_modern` 的 sans-serif typography + "排版叠字"规则，目标是减少错字/黑边/变形
- `artistic`：允许 `japanese_soft` 的手写/刷字 typography，但需要更多重试；错字即失败，必须重生成

Typography 必须使用英文描述，且建议从对应 style profile 的候选集合中选择一条（每次生成可以随机选一条，用于控制一致的“字体气质”）。Typography 需要同时考虑**设计风格**和**语言适用性**（支持中文、英语、法语、日语等多语言渲染）：

- minimal_modern
  - `modern geometric sans-serif headline, slightly condensed, clean letterforms, high readability, medium weight, supports Chinese/English/Japanese, not cursive, not handwritten`
  - `neo-grotesque sans-serif title (Swiss modern), clean grid typography, medium weight, excellent for Chinese and Latin characters, not cursive, not handwritten`
  - `ultra-minimal condensed sans-serif headline, wide tracking, clean modern, bold weight, versatile for multi-language text, not cursive, not handwritten`
  - `restrained modern serif small-caps headline, editorial minimal, medium weight, elegant for English/French, refined for Chinese, not cursive, not handwritten`
- japanese_soft
  - `soft handwritten pen lettering, gentle thin-to-medium strokes, warm and subtle, supports Japanese kana and Chinese characters, not exaggerated`
  - `rounded minimal sans-serif with a soft friendly feel, warm tone, subtle, excellent for Japanese/hiragana/katakana, Chinese, and Korean Hangul`
  - `delicate brush lettering with minimal texture, soft and clean, ideal for Japanese calligraphy style, Chinese brush characters, not heavy ink`
  - `light modern serif with gentle handwritten vibe, warm and minimal, elegant for Japanese and Chinese mixed text`
- luxury_editorial
  - `high-contrast modern serif headline (editorial), premium and restrained, bold weight, sophisticated for English/French, refined for Chinese, not cursive`
  - `luxury editorial serif small-caps headline, clean hierarchy, minimal effects, bold weight, perfect for luxury brands in Chinese/English/French`
  - `premium condensed serif display headline, elegant proportions, editorial feel, versatile for multi-language luxury text`
  - `modern editorial sans-serif headline with refined proportions, premium minimal, bold weight, elegant for Chinese and Latin mixed text`
- rounded_bold
  - `ultra-rounded geometric sans-serif headline, extra bold weight, soft corners, friendly and playful, high impact, excellent for Chinese/English/Japanese playful text`
  - `extra bold rounded sans-serif title, maximum roundness, chunky letterforms, warm and approachable, ideal for Chinese/English bilingual bold statements`
  - `super round geometric sans-serif, heavy weight, bubble-like appearance, cute and bold, perfect for Japanese/Chinese friendly designs`
- yahoma_light
  - `clean geometric sans-serif, light weight, subtle, microsoft yahei inspired, minimal and modern, excellent legibility for Chinese simplified/traditional, English, Japanese`
  - `thin geometric sans-serif headline, light weight, clean and crisp, highly legible, neutral, versatile for Chinese/English/Japanese/French clean designs`
  - `ultra-light modern sans-serif, delicate strokes, refined proportions, subtle elegance, ideal for elegant Chinese and Latin mixed text`

## Wan2.7 调用（依赖与使用）

### 依赖

- 已安装 `wan2.7-image-skill`（本机默认位置通常为 `~/.trae/skills/wan2.7-image-skill`）
- 环境变量：
  - `DASHSCOPE_API_KEY`（必填）
  - `DASHSCOPE_BASE_URL`（可选，默认 `https://dashscope.aliyuncs.com/api/v1/`）

### 参考图上传（本地文件 → oss://）

**注意：如果用户提供的是公网可访问的 HTTP/HTTPS URL，可以直接使用，无需上传到 OSS。**

当用户提供本地文件时，需要上传到 OSS 生成临时 URL：

```bash
python3 ~/.trae/skills/wan2.7-image-skill/scripts/file_to_oss.py \
  --file /path/to/reference.png \
  --model wan2.7-image
```

**替代方案**：如果没有 OSS 配置，可以使用免费的图片分享服务（如 Imgur、Google Photos 等）生成公网 URL，直接提供给技能使用。

### 生成单张图（推荐：图生图以保证一致性）

```bash
python3 ~/.trae/skills/wan2.7-image-skill/scripts/image-generation-editing.py \
  --user_requirement "<PUT_PROMPT_HERE>" \
  --input_images "oss://..." "oss://..." \
  --n 1 \
  --size "1600*1600"
```

**【重要】输出要求：生成完成后，必须完整输出图片的预签名 URL，包含所有查询参数（Expires、OSSAccessKeyId、Signature 等），禁止截断。**

**【强制】编号分配：每张生成的图片必须分配唯一编号，格式为 `IMG-001`、`IMG-002`...，序号自动递增。输出格式必须为：**

```
IMG-001 | {type} | {size} | prompt: ... | image_url: https://...
```

## 高级功能与脚本支持

### 增强脚本

本技能提供了一系列增强脚本，位于 `scripts/` 目录，用于提升生成效果和用户体验：

#### 1. 多语言支持与风格映射
- **`scripts/style_mapper.py`**：支持中文、法语、日语到英文风格档案的自动映射
- **`scripts/enhancer.py`**：自动检测输入语言并应用相应的提示词模板

#### 2. 提示词优化
- **`scripts/prompt_optimizer.py`**：压缩长提示词，缓存优化结果，提升生成速度

#### 3. 提示词优化
- **`scripts/prompt_optimizer.py`**：压缩长提示词，缓存优化结果，提升生成速度

### 使用方法

#### 方式 1：通过对话调用（推荐）

在支持 Skills 的对话/Agent 里，直接输入：

```
Use Skill: amazon-product-image-wan27

# 高级选项（可选）
AdvancedOptions:
  language: zh  # 自动检测或手动指定
  enable_caching: true  # 启用提示词缓存
  batch_edit: true  # 启用批量编辑模式
```

#### 方式 2：直接使用脚本

```bash
# 优化提示词
python3 scripts/prompt_optimizer.py

# 风格映射
python3 scripts/style_mapper.py
```

## Common Mistakes

- 主图出现道具/场景/文字：主图必须白底且无任何文字元素
- **输出缺少图片编号：每张生成的图片必须分配唯一编号，必须在输出中体现 IMG-xxx 格式的编号**
- 文案被模型自动改写：必须使用"唯一文本规则"锚定，且文案只能单行且必须完全一致
- 画面像网页截图/带边框留白：提示词必须明确 full-bleed、禁止 UI/海报模板/边框/白边
- 商品变形/变成其他商品：必须反复强调"同一商品身份一致性"，并始终提供参考图
- 在图片里生成品牌 logo：除非用户明确要求，否则要禁止 logo/watermark/text

## Minimal Example（最小复用示例）

用户输入（示例）：

- 商品：Stainless Steel Insulated Tumbler
- 参考图：1 张
- 生成：main 1 张（无文案）；scene 2 张（分别有文案）
- 风格：minimal_modern

执行步骤：

1. 把参考图上传成 `oss://...`
2. 用 main 模板生成主图 prompt 并调用生成
3. 用 scene 模板生成 2 条 prompt（每条带不同 `{COPY}`），每条都带"唯一文本规则"，分别调用生成

## Complete Example（完整复用示例）

用户输入（示例）：

- 商品名称：Ceramic Matte Glaze Plant Pot
- 类目：Planters
- 材质：Ceramic (matte glaze finish)
- 尺寸：6 inch diameter, 5.5 inch height
- 参考图：2 张（不同角度）
- 需要生成：
  - main 2 张（无文案）
  - scene 2 张（场景 1：桌面 + 文案"A"，场景 2：户外 + 文案"B"）
  - detail 1 张（无文案）
  - aplus 1 张（有文案："Bring Nature Home"）
  - dimension 1 张（尺寸标注图）
    - 子级 fusion 1 张（多SKU变体融合：[IMG-001红] + [IMG-002蓝] + [IMG-003绿]，场景：客厅）
- 风格档案：minimal_modern
- CopyMode：strict

输出示例（实际执行时按此格式）：

```
IMG-001 | main | 1600*1600 | prompt: Based on the reference image showing Ceramic Matte Glaze Plant Pot, a Planters product..., image_url: https://dashscope-7c2c.oss-accelerate.aliyuncs.com/xxx.png?Expires=1776616034&OSSAccessKeyId=xxx&Signature=xxx

IMG-002 | main | 1600*1600 | prompt: Based on the reference image showing Ceramic Matte Glaze Plant Pot..., image_url: https://dashscope-7c2c.oss-accelerate.aliyuncs.com/yyy.png?Expires=1776616034&OSSAccessKeyId=xxx&Signature=xxx

IMG-003 | scene | 1600*1600 | scene: Desk/Table (preset #1) | copy: A | prompt: Based on the reference image showing Ceramic Matte Glaze Plant Pot... [with CRITICAL TEXT TO RENDER block for "A"], image_url: https://dashscope-7c2c.oss-accelerate.aliyuncs.com/zzz.png?Expires=1776616034&OSSAccessKeyId=xxx&Signature=xxx

IMG-004 | scene | 1600*1600 | scene: Outdoor/Nature (preset #6) | copy: B | prompt: Based on the reference image showing Ceramic Matte Glaze Plant Pot... [with CRITICAL TEXT TO RENDER block for "B"], image_url: https://dashscope-7c2c.oss-accelerate.aliyuncs.com/aaa.png?Expires=1776616034&OSSAccessKeyId=xxx&Signature=xxx

IMG-005 | detail | 1600*1600 | prompt: Based on the reference image showing Ceramic Matte Glaze Plant Pot..., image_url: https://dashscope-7c2c.oss-accelerate.aliyuncs.com/bbb.png?Expires=1776616034&OSSAccessKeyId=xxx&Signature=xxx

IMG-006 | aplus | 1464*600 | scene: Living Room (preset #9) | copy: Bring Nature Home | prompt: Based on the reference image showing Ceramic Matte Glaze Plant Pot... [with CRITICAL TEXT TO RENDER block], image_url: https://dashscope-7c2c.oss-accelerate.aliyuncs.com/ccc.png?Expires=1776616034&OSSAccessKeyId=xxx&Signature=xxx

IMG-007 | dimension | 1600*1600 | prompt: Based on the reference image showing Ceramic Matte Glaze Plant Pot... [with dimension annotation rules], image_url: https://dashscope-7c2c.oss-accelerate.aliyuncs.com/ddd.png?Expires=1776616034&OSSAccessKeyId=xxx&Signature=xxx

IMG-F001 | fusion | scene: Living Room (#9) | copy: Bring Nature Home | refs: [IMG-001, IMG-002, IMG-003] | prompt: Multi-product fusion composition... [with unified text overlay], image_url: https://dashscope-7c2c.oss-accelerate.aliyuncs.com/fff.png?Expires=1776616034&OSSAccessKeyId=xxx&Signature=xxx
```

执行步骤：

1. **参考图处理**：
   - 如果用户提供的是公网可访问的 HTTP/HTTPS URL：直接使用
   - 如果用户提供的是本地文件：上传成 `oss://...`（需要 OSS 配置）或使用免费图片分享服务生成公网 URL
2. 用 main 模板生成 2 条 prompt，分别调用生成
3. 用 scene 模板（文案"A" + 预设 #1 桌面）生成 prompt（带"唯一文本规则"，渲染"A"），调用生成
4. 用 scene 模板（文案"B" + 预设 #6 户外）生成 prompt（带"唯一文本规则"，渲染"B"），调用生成
5. 用 detail 模板生成 prompt 并调用生成
6. 用 aplus 模板（有文案 + 预设 #9 客厅）生成 prompt（带"唯一文本规则"），调用生成
7. 用 dimension 模板生成 prompt（基于提供的尺寸数据）并调用生成
8. 用 fusion 模板生成 prompt（多产品描述 + 客厅场景 + 统一文案）并调用生成

## Verification Checklist（交付前自检）

- **【重要】image_url 必须是完整 URL：必须包含 ?Expires=...&OSSAccessKeyId=...&Signature=... 等全部参数，禁止省略或截断**
- **【强制】每张图片必须分配唯一编号**：默认图使用 IMG-xxx，融合图使用 IMG-Fxxx，编辑图使用 IMG-xxx-EDIT-x
- main：背景纯白（接近 #FFFFFF）、无文字、无道具、商品占画面约 85%+、仅有自然阴影
- scene/detail/aplus：无边框/无网页截图感/无拼接分栏/无多余文字
- copy：图片中仅出现 `{COPY}` 一次，逐字逐符号一致，且为单行
- 一致性：所有图中的商品外观与参考图一致（颜色/材质/结构/关键细节不变）
- dimension：尺寸标注清晰，使用黑色或深灰色细线箭头，标注长度、高度、宽度，数值与用户输入一致
- fusion：多个产品自然融合，无明显拼接痕迹，产品身份各自保持一致；产品有前后层次和自然遮挡关系，无生硬分栏；统一的光线方向和色温，无冲突光源；文案（如有）仅出现一次，位置避开产品遮挡区域；编号使用 IMG-Fxxx 格式

## Red Flags（出现即停止并改 Prompt）

- 输出的图片 URL 被截断（缺少 ?Expires=... 等参数）：这是严重错误，必须完整输出
- “顺便加点 slogan / badge / 促销字样”：违反唯一文本规则
- “主图加点道具更好看”：主图应无道具无场景
- "参考图不重要，随便生成"：会导致商品身份漂移，必须保留参考图输入

---

## Fusion Mode（融合生图模式）

### 触发方式

融合生图模式支持三种触发方式：

#### A. 用户输入编号组合（自动识别）

当用户输入包含多个参考图编号时自动触发：

```
输入示例：
- 融合生图：[IMG-001] + [IMG-002]
- 场景：桌面 (#1)
- 风格：minimal_modern
- 文案：Pure Elegance（可选）
```

#### B. 显式参数

使用 `fusion: true` 或 `type: fusion` 参数：

```
输入示例：
- 类型：fusion
- 参考图：IMG-001, IMG-002
- 场景：预设 #1 桌面
- 风格：minimal_modern
- 文案：Pure Elegance（可选）
```

#### C. 用户输入编号组合（显式指定）

用户明确指定要融合的编号：

```
输入示例：
- 融合生图：[IMG-001] + [IMG-002]
```

#### D. 图片 URL 输入

支持直接输入图片 URL（包括 HTTP/HTTPS URL 和 oss:// URL）：

```
输入示例：
- 融合生图：https://example.com/img1.jpg + https://example.com/img2.jpg
- 融合生图：oss://bucket/path/to/img1.png + oss://bucket/path/to/img2.png
- 场景：桌面 (#1)
- 风格：minimal_modern
- 文案：Pure Elegance（可选）
```

#### E. 混合输入

支持编号和 URL 的混合输入：

```
输入示例：
- 融合生图：[IMG-001] + https://example.com/img2.jpg
- 场景：客厅 (#9)
- 风格：luxury_editorial
```

### 执行流程

1. **检测融合需求**
   - 解析用户输入，识别多个参考图编号、图片URL或fusion关键字
   - 提取融合模式参数（场景、文案、风格）

2. **获取参考图信息**
   - 从会话记忆获取编号对应的图片URL
   - 直接使用用户提供的HTTP/HTTPS URL或oss:// URL
   - 识别是否为同一产品多角度或不同SKU变体

3. **构建融合提示词**
   - 按模板填充多产品描述
   - 应用选定的场景片段（如有）
   - 添加统一文案规则（如有）
   - 应用风格档案的排版参数

4. **调用 wan2.7 生成**
   - 使用图生图模式（多图输入）
   - 指定分辨率（建议 1600*1600 或 2000*2000）

5. **输出结果**
   - 分配编号：IMG-F001, IMG-F002...
   - 完整输出预签名 URL
   - 记录关联的原始参考图信息（编号或URL）

### 编号生成规则

- 格式：`IMG-F{序号}`，如 IMG-F001、IMG-F002
- F 代表 Fusion 融合模式
- 序号在当前会话中独立递增（与普通图片编号分开）

### 会话记忆管理（融合模式扩展）

融合图片记录格式：
```
{
  编号: "IMG-F001",
  类型: "fusion",
  原始参考图: ["IMG-001", "IMG-002", "https://example.com/img3.jpg", "oss://bucket/path/to/img4.png"],
  场景: "Living Room (#9)",
  文案: "Bring Nature Home",
  prompt: "...",
  image_url: "..."
}
```

---

## Image Editing Mode（图片编辑模式）

### 触发条件

用户输入包含"编辑"关键词 + 图片编号时，激活图片编辑模式。

### 输入格式（3 项）

```
1. [编辑模式] — 声明使用编辑模式
2. IMG-003 — 图片编号（格式：IMG-001）
3. 修改意见 — 用户对图片的修改要求（如：背景换成户外、颜色调暗一些、把框选区域的道具去掉）
```

### 执行流程

1. **查找原图**：从会话记忆中查找编号对应的图片信息（URL、类型、场景、prompt）
2. **获取 URL**：如果会话记忆中找不到，请求用户粘贴原图完整预签名 URL
3. **沿用场景**：必须沿用原图的场景模板，不得改变场景类型
4. **生成提示词**：将修改意见融入原始提示词，调用 wan2.7 进行图生图编辑
5. **输出结果**：生成新编号（如 IMG-003-EDIT-1），完整输出预签名 URL

### 编号生成规则

- 格式：`IMG-{序号}`，如 IMG-001、IMG-002
- 编辑版本：`IMG-{原编号}-EDIT-{序号}`，如 IMG-003-EDIT-1
- 序号在当前会话中自动递增

### 会话记忆管理

- 每次生成图片后，Agent 自动记录：`{编号, 类型, 场景, prompt, URL}`
- 编辑时优先从会话记忆获取图片信息
- 如果会话记忆中没有该编号，提示用户提供原图 URL

### 编辑模式提示词模板

基于原图 prompt，将修改意见融入生成新 prompt：

```
Based on the reference image (URL: {ORIGINAL_URL}), make the following modifications:
{EDIT_REQUEST}

Keep all other elements identical to the original image:
- Same product identity and appearance (shape, color, material, details)
- Same scene context and composition
- Same lighting style and quality

Generate only the modified version.
```

### 输出格式（编辑模式）

```
{新编号} | edit | {原编号} | {完整预签名URL（含Expires、OSSAccessKeyId、Signature等参数）}
```

示例：
```
IMG-003-EDIT-1 | edit | IMG-003 | https://dashscope-7c2c.oss-accelerate.aliyuncs.com/xxx.png?Expires=1776616034&OSSAccessKeyId=LTAI5tPxpiCM2hjmWrFXrym1&Signature=bRMGLPJUZK0rjo2FhHl1Ptj21eg%3D
```

**重要**：URL 必须包含所有查询参数，禁止截断。

---
> Source: [colddew-yj/amz-product-img-gen-skill](https://github.com/colddew-yj/amz-product-img-gen-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
