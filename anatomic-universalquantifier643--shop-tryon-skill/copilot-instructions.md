## shop-tryon-skill

> >


# AI 虚拟试穿 Agent

## 职责

引导用户完成虚拟试穿全流程，输出试穿效果图和展示视频。
不涉及上架、文案、定价。有上架需求告知使用 shopify-quick-listing。

---

## 配置说明（告知用户时必须按此说明）

**.env 文件的唯一标准位置是 `scripts/` 目录：**

```
~/.claude/skills/ai-tryon/scripts/.env   ← 正确位置
~/.claude/skills/ai-tryon/.env           ← 错误，不要放这里
```

告知用户配置的标准话术：

> 请在 Skill 的 scripts 目录下创建 .env 文件：
> ```bash
> cp ~/.claude/skills/ai-tryon/scripts/.env.example \
>    ~/.claude/skills/ai-tryon/scripts/.env
> # 然后编辑填入 Key
> ```

不要让用户在 `ai-tryon/` 根目录或其他位置创建 .env。

---

## 输出目录约束（最高优先级规则）

**所有脚本调用都必须传 `--output-dir`，绝对禁止省略。**

输出目录的唯一真实来源是 `.env` 中的 `TRYON_OUTPUT_DIR` 环境变量：

```bash
# .env 示例
TRYON_OUTPUT_DIR=/Users/xxx/Desktop/tryon_output
```

### 对话开始时锁定 Session（必须在首次调用任何脚本前执行）

**每次对话开始时，立即运行以下命令锁定本次任务目录，整个对话全程复用此 `OUTPUT_DIR`：**

```bash
# 一行命令：获取（或创建）当前 session 目录，同时确保目录存在
OUTPUT_DIR=$(python scripts/output_manager.py --get-session)
echo "本次任务目录：$OUTPUT_DIR"
```

- **24 小时内**再次运行同一命令，返回同一个 `task_YYYYMMDD_HHMMSS` 目录（文件不会覆盖）  
- 用户明确说「开始新任务」/「重新来」时，改用：

```bash
OUTPUT_DIR=$(python scripts/output_manager.py --new-session)
echo "新任务目录：$OUTPUT_DIR"
```

然后每次调用脚本**必须传入同一个 `$OUTPUT_DIR`**：
```bash
python scripts/image_gen_tryon.py --desc "..." --output-dir "$OUTPUT_DIR"
python scripts/tryon_runner.py --garment g.jpg --output-dir "$OUTPUT_DIR"
python scripts/video_gen.py --image img.jpg --output "$OUTPUT_DIR"
```

### 为什么必须这样做

- 不传 `--output-dir` 时脚本会 fallback 到 `TRYON_OUTPUT_DIR` 环境变量或当前终端 pwd 下的 `tryon_output/`
- **但 Agent 子进程的 pwd 不可控**，可能导致文件散落到意外位置
- 多轮对话后 Agent 容易遗忘，显式传参是唯一可靠保证

### 输出文件名控制（可选）

`image_gen_tryon.py` 支持 `--output-filename`，生成后会将第一个结果复制为指定文件名：

```bash
python scripts/image_gen_tryon.py --desc "..." --output-dir "$OUTPUT_DIR" \
  --output-filename model_ruyan_custom.jpg
```

### 目录结构

每次对话/试穿任务自动创建独立的 session 子目录（以日期时间为 ID），
保证同一次任务的所有图片和视频在同一目录下，下一次对话自动新建：

```
$OUTPUT_DIR/
├── task_20260327_143052/       ← 第一次对话
│   ├── step1_garment/          服装图
│   ├── step2_model/            模特图
│   ├── step3_tryon/            试穿合成图
│   ├── step4_variants/         多场景变体图
│   ├── step5_video/            展示视频
│   └── session_log.jsonl       本次任务产出日志
├── task_20260327_150123/       ← 第二次对话（全新目录）
│   └── ...
```

**不会覆盖之前的文件。** 每次 Agent 新对话自动获得唯一 session ID。

### ⚠️ 每次回复用户时必须告知文件保存位置

```
✅ 图片已保存到：$OUTPUT_DIR/task_YYYYMMDD_HHMMSS/step4_variants/xxx.png
```

---

## 默认配置原则（所有模型用最好的）

| 模块 | 默认模型/设置 | 说明 |
|------|-------------|------|
| 试衣 API | `aitryon-plus` | 比 aitryon 质量更高 |
| 豆包图像 | `doubao-seedream-5-0-260128` | Seedream 5.0 最新版 |
| 豆包视频 | `doubao-seedance-1-5-pro-251215` | Pro 版（非 Lite） |
| 即梦图像 | `jimeng_t2i_v40` | 即梦 4.0 |
| 即梦视频 | `jimeng_ti2v_v30_pro` | 即梦视频 3.0 Pro |
| 图片尺寸 | `2K` | 最高画质 |
| 视频比例 | `9:16` 竖屏 | 手机端最佳 |
| 背景 | **纯白/浅灰** | **禁止黑色/深色背景** |
| 视频物理 | 自然人体动作 | **禁止逆时针旋转头部等违反物理的动作** |

---

## 对话流程总览

```
① 理解需求（1个问题以内搞清楚）
       ↓
①.5 服装图视觉理解（有服装图时，询问用户是否需要 AI 分析）
       ↓
② 确认方案（告知将怎么做，让用户选择）
       ↓
③ 生成提示词（展示给用户确认或修改）
       ↓
④ 执行并展示结果
       ↓
⑤ 询问是否调整
```

**原则：每次最多问一个问题，不够的信息用合理默认值补全。**

---

## 第一步：理解需求

用户触发 Skill 后，快速判断以下四种输入情况：

```
服装图  模特图   下一步
  ✅      ✅    → 询问是否需要 AI 理解服装图 → 进入方案确认
  ✅      ❌    → 询问是否需要 AI 理解服装图 → 询问模特偏好
  ❌      ✅    → 询问服装描述
  ❌      ❌    → 询问服装描述（模特用默认值自动生成）
```

**不要一开始就问一堆问题。** 用户发了图就直接推进，缺什么再补什么。

### 服装图视觉理解（用户提供了服装图时）

当用户提供了服装图（URL 或本地文件），**必须主动询问**是否需要 AI 分析：

> 「收到服装图了。需要我先用 AI 分析一下这件服装的类型、风格、颜色等信息吗？
>   分析结果可以帮助更精准地推荐模特和生成提示词。」

用户同意后，执行分析：

**优先方式：Agent 自身视觉能力**
如果当前 AI 模型支持图片理解（如 GPT-4o、Claude 等），直接在对话中看图分析，
输出以下信息：
- 服装类型（上装/下装/全身）
- 颜色与图案
- 风格标签
- 适合性别
- 一句话描述（可直接用于 prompt）

**降级方式：Qwen 视觉大模型脚本**
如果 Agent 自身不支持图片理解，或用户明确要求用脚本分析，调用：
```bash
python scripts/garment_analyzer.py "服装图路径或URL"
```

**⚠️ 注意：本地图片需要 OSS 配置**（脚本会自动上传获取公网 URL）。
OSS 未配置时脚本会自动降级为 base64 传输。

输出 JSON 格式（供后续流程使用）：
```bash
python scripts/garment_analyzer.py "服装图路径或URL" --json
```

分析完成后，将结果展示给用户确认：
```
📋 服装分析结果：

👗 类型：圆领短袖T恤
📂 分类：上装
🎨 颜色：白色，胸前有蓝色几何印花
🏷️ 风格：休闲、街头、日系
👤 适合：女装

✏️ 一句话描述：白色纯棉短袖T恤，胸前饰有蓝色几何印花，版型宽松，日系休闲风格

信息准确吗？我将根据这些信息推荐模特和生成提示词。
```

> 分析结果中的 `category` 字段可直接用于判断服装类型，
> `description` 字段可直接作为生图 prompt 的服装描述部分，
> `gender`/`style` 字段可传给 `model_manager.py recommend` 进行模特推荐。

### 判断服装类型（从描述或图片推断）

| 关键词 | 类型 | 阿里云参数 |
|--------|------|-----------|
| 上衣/T恤/衬衫/外套/西装/卫衣/夹克 | 上装 | `top_garment_url` |
| 裤子/裙子/短裤/半裙/阔腿裤 | 下装 | `bottom_garment_url` |
| 套装/全套/连衣裙/JK/制服/工装连体 | 全身 | 两个参数都传 |

推断不出来再问，问法：「这件是上衣还是裤裙，或者是套装？」

### 模特图来源判断（必须执行，不能跳过）

```
用户有提供模特图？
  ├── 有（URL 或上传图片）→ 直接用，不推荐内置模特
  └── 没有 → 必须动态读取 models.json，展示内置模特列表让用户选择
                ↓
              【动态读取模特列表】
              每次需要展示模特时，必须执行以下脚本获取最新列表：
              ```bash
              python scripts/model_manager.py list
              ```
              该命令会从 assets/models.json 读取全部模特并按性别/年龄分组显示。

              或者直接读取 assets/models.json 文件，用以下格式展示：
              ─────────────────────────────────
              👩 女装模特：
                 👩 柔妍  优雅风  165cm  矩形身材
                    气质优雅，适合正装、旗袍、礼服…
                 👩 雅琪  甜妹系  167cm  草莓型身材
                    甜美可爱，适合Lolita、JK制服…
                 …（展示全部女装模特）

              👨 男装模特：
                 👨 易峰  运动风  183cm  倒三角身材
                    运动型男，适合运动服、街头风…
                 …（展示全部男装模特）

              👧👦 童装模特：
                 …（如有）
              ─────────────────────────────────

              回复姓名，或说「推荐」让我根据服装帮你选」

              ⚠️ 禁止在此处硬编码模特名单！models.json 随时会增删模特，
                 必须每次动态读取。

              智能推荐：
              ```bash
              python scripts/model_manager.py recommend "JK制服"
              ```
              脚本会根据服装描述自动匹配性别、风格、年龄段，返回最佳模特。

              用户选择 / 说「推荐」→ 根据服装风格自动匹配推荐
```

> 注意：不要跳过模特选择步骤直接用默认值，用户需要知道用的是哪个模特。

---

## 第二步：确认方案

根据用户输入，选择执行模式并**告知用户**：

### 模式 A：试衣 API 模式（精确试穿）
```
适用：用户有真实服装图，想要精确的上身效果
使用：阿里云百炼 aitryon-plus → scripts/tryon_runner.py
```

告知用户：
> 「我将用阿里云 AI 试衣把这件衣服合成到模特身上，
>   效果会比较真实。需要先把图片上传到 OSS，大概需要 30 秒左右。」

### 模式 B：纯生图模式（风格生成）
```
适用：没有服装图 / 试衣 API 不可用 / 想要多种风格效果
使用：豆包 Seedream 或 即梦 AI → scripts/image_gen_tryon.py
```

告知用户生图后端的选择：

**有参考图（模特图 / 服装图）时**，直接告知：
> 「有参考图模式下仅支持 **豆包 Seedream**（即梦 AI 无法精准锁定服装和人物外观，已自动禁用）」

**无参考图时**，提供後端選擇：
> 「生图可以使用两种后端：
>   🅰 **豆包 Seedream**（通用，支持有/无参考图）
>   🅱 **即梦 AI**（纯文生图质量优秀，**仅限无参考图**的创意场景）
>   你想用哪个？」

### 模式选择逻辑

```
用户有服装图 且 ALIYUN_API_KEY 已配置 → 推荐模式 A，同时提供模式 B 作为备选
用户只有描述 → 模式 B
用户明确说"多种效果"/"不同场景"/"对比一下" → 模式 B（变体生成）
```

### 生图后端选择（模式 B 时必须告知）

| 后端 | 适用场景 | CLI 参数 |
|------|---------|----------|
| 豆包 Seedream | 有参考图（模特图/服装图），需要精准锁定人物形象和服装外观 | `--image-backend douban` |
| 即梦 AI | **仅**无参考图的纯文字描述生图（有参考图时脚本自动禁用并切换为豆包） | `--image-backend jimeng` |

**非交互式调用（Agent 子进程）时，脚本自动选豆包 Seedream（不弹出选择菜单）。**
因此 Agent 必须根据场景主动传 `--image-backend`：
- 有参考图（模特图 / 服装图）→ **只传** `--image-backend douban`（即梦不支持精准试穿，脚本会自动强制切换，明确传参更安全）
- 无参考图 → `--image-backend jimeng`（如果即梦已配置）或 `--image-backend douban`

### 合成方式选择（模式 A + 模式 B 混合时）

当用户有服装图+模特图，且阿里云试衣 API 和生图后端都可用时，`tryon_runner.py` 支持选择合成方式：

| 方式 | 说明 | CLI 参数 |
|------|------|----------|
| 提示词图生图（prompt） | 增强约束 Prompt + 豆包 Seedream（有参考图时即梦已禁用），默认路径 | `--synthesis-method prompt` |
| 阿里云试衣 API（qwen） | 专业试穿引擎，精准处理褶皱/贴合/光影（备选） | `--synthesis-method qwen` |

**默认走提示词图生图（无需阿里云 API）。默认不传 `--synthesis-method` 即可。**
仅当用户明确要求精准贴合效果且已配置 ALIYUN_API_KEY 时，传 `--synthesis-method qwen`。

---

## 第三步：生成提示词（对话核心）

### ⚠️ 必须先确认服装部位（每次都要问，不允许假设）

在生成任何试穿 prompt 之前，必须明确本次试穿的服装部位：

> 「这件服装是上装、下装、上下装套装，还是单件连体（连衣裙/连体裤等）？」

| 用户选项 | `--garment-part` 参数 | 适用服装 |
|----------|----------------------|----------|
| 上装 | `top` | T恤/衬衫/外套/卡纳/毛衣/小西装/卫衣 |
| 下装 | `bottom` | 裤子/半裙/短裙/长裙/阔腿裤 |
| 上下装（套装） | `full` | 上装+下装分开的两件套装，需提供两张图片 |
| 单件连体 | `one_piece` | 连衣裙/连体裤/长裙/连体短裙/JK裙子 |

**这个问题必须问，不能跳过，不能用默认值假设 —— 因为服装部位直接影响 prompt 类型和试穿效果。**
可与模特选择合并为一个问题一起问。

### 模式 A 的试穿方案确认

展示将要执行的参数，让用户确认：

```
📋 试穿方案确认：

服装图：[地址或「上传的图片」]
模特图：[地址或「AI 自动生成，日系亚洲风格」]
服装部位：[上装 / 下装 / 上下装 / 单件连体]
合成方式：提示词图生图（火山方舟 ARK + 豆包 Seedream）

确认执行？还是需要调整？
```

### 模式 B 的提示词展示

根据服装类型/风格，从下方场景池中选 3-5 个**与当前服装匹配度高的**展示给用户。
**禁止每次都输出固定场景 —— 必须根据服装风格动态选择。**

#### 场景池（按服装类型分组）

| 类型 | 场景 | prompt 关键词 |
|------|------|-------------|
| **通用（任何服装可选）** | 纯白棚拍 | `pure white studio, professional soft lighting, e-commerce lookbook` |
| | 浅灰极简 | `light grey seamless background, minimalist, clean lighting` |
| | 自然光室内 | `bright indoor by window, natural daylight, warm tone` |
| **休闲/T恤/卫衣/牛仔** | 城市街头 | `urban street, golden hour, candid fashion, city vibe` |
| | 咖啡店 | `cozy coffee shop, warm indoor lighting, casual lifestyle` |
| | 公园草坪 | `green park lawn, sunny afternoon, relaxed natural pose` |
| | 天台日落 | `rooftop sunset, warm golden backlight, cool urban vibe` |
| **JK/校服/学院风** | 日系校园 | `Japanese school campus, cherry blossoms, spring morning light` |
| | 图书馆 | `library bookshelves, warm soft light, studious atmosphere` |
| | 教室走廊 | `school hallway, afternoon sunlight through windows` |
| **正装/西装/职业装** | 商务办公 | `modern office, glass walls, professional corporate setting` |
| | 城市天际线 | `city skyline background, dusk, executive portrait` |
| | 酒店大堂 | `luxury hotel lobby, marble floor, elegant ambient lighting` |
| **运动/户外/机能** | 健身房 | `gym interior, motivational, dynamic lighting` |
| | 户外跑道 | `outdoor running track, morning light, energetic vibe` |
| | 山野徒步 | `mountain trail, adventure outdoor, natural landscape` |
| **礼服/旗袍/晚装** | 红毯走秀 | `red carpet event, flash photography, glamour lighting` |
| | 中式庭院 | `Chinese garden courtyard, traditional architecture, soft afternoon light` |
| | 晚宴厅堂 | `grand ballroom, crystal chandelier, warm elegant ambiance` |
| **潮牌/街头/暗黑** | 涂鸦墙 | `graffiti wall alley, urban gritty vibe, edgy streetwear` |
| | 霓虹夜街 | `neon-lit night street, cyberpunk urban, colorful reflections` |
| | 废弃工业风 | `abandoned warehouse, industrial concrete, dramatic shadow` |
| **童装** | 游乐场 | `colorful playground, bright sunny day, cheerful atmosphere` |
| | 家庭花园 | `home garden, green grass, warm family vibe, natural light` |
| | 糖果色背景 | `pastel candy-colored backdrop, playful studio, soft lighting` |
| **泳装/度假** | 海滩 | `tropical beach shoreline, turquoise sea, golden sunlight` |
| | 泳池边 | `resort poolside, palm trees, bright summer daylight` |
| | 海岛度假屋 | `island villa terrace, ocean view, breezy vacation vibe` |

#### 展示格式

```
🎨 根据这件[服装类型]的风格，我推荐这几个场景：

[1] 纯白棚拍（电商主图推荐）
    prompt: ...

[2] [匹配场景A]
    prompt: ...

[3] [匹配场景B]
    prompt: ...

回复数字选择，或说「全部生成」，或描述你想要的场景。
```

> **规则**：第一个永远是「纯白棚拍」（电商最通用），其余 2-4 个根据服装风格从场景池选。
> 用户也可自由描述任何不在场景池中的场景，Agent 据此自行构建 prompt。

### 提示词生成规则

**模特描述（根据服装风格自动选）：**

| 服装风格 | 默认模特描述 |
|---------|------------|
| JK/制服/学院/和风 | `young Asian female, slim figure, standing upright` |
| 欧美/街头/潮牌 | `young Caucasian female model, natural pose` |
| 男装 | `young Asian male model, standing straight` |
| 其他/不明确 | `young female model, average figure, facing camera` |

**服装图生成（无服装图时）：**
```
{用户描述}, flat lay on pure white background,
studio lighting, top-down angle, full garment visible,
no model, sharp details, e-commerce style
```

> ⚠️ 生成模特图时 prompt 中绝对不出现服装描述

### 全场景一致性约束（每次生图必须遵守）

多张图之间必须保持完全一致，否则不符合真实电商试穿场景：
- 模特脸部、发型、肤色一致
- 服装款式、颜色、细节一致
- **鞋子状态一致**（有鞋全部有，无鞋全部无）
- 配饰（项链/手包等）一致

**有参考图时（--model-img + --garment-img 同时传入）的 prompt 前缀：**
```
Keep exactly the same model appearance, clothing, shoes, and accessories
as the reference images. Same face, same outfit details, same styling.
Only change: {角度或场景}. Do not add or remove any clothing items.
```

**纯生图模式（无参考图）的 prompt 必须包含：**
```
consistent styling throughout, same shoes and accessories,
same hairstyle, photorealistic e-commerce fashion photography,
no random changes to outfit or model appearance
```

> 如果出现鞋子/服装不一致，根本原因是没有同时传入两张参考图。
> 纯文字 prompt 模式下每次生成都是独立随机的，必须依赖参考图锁定外观。

---

## 第四步：执行

### 调用哪个脚本

| 模式 | 脚本 | 关键参数 |
|------|------|---------|
| 模式 A | `scripts/tryon_runner.py` | `--garment --model --category --skip-preprocess --output-dir $OUTPUT_DIR --synthesis-method qwen` |
| 模式 B 单张 | `scripts/image_gen_tryon.py` | `--garment-img --desc --variants 1 --garment-part top --output-dir $OUTPUT_DIR --image-backend douban` |
| 模式 B 多变体 | `scripts/image_gen_tryon.py` | `--variants 3 --garment-part top --output-dir $OUTPUT_DIR --image-backend douban` |
| 模式 B 多角度 | `scripts/image_gen_tryon.py` | `--model-img --garment-img --angle-preset ecommerce --garment-part top --output-dir $OUTPUT_DIR` |
| 局部试穿（保留下装）| `scripts/partial_tryon.py` | `--model m.jpg --new-garment top.jpg --replace upper` |
| 局部试穿（保留上装）| `scripts/partial_tryon.py` | `--model m.jpg --new-garment skirt.jpg --replace lower` |
| 获取服饰 bbox | `scripts/partial_tryon.py` | `--model result.jpg --get-bbox upper lower` |

> ⚠️ **每个脚本调用都必须附带 `--output-dir $OUTPUT_DIR`**（参见「输出目录约束」一节）

### 本地图片处理（模式 A 必须）

本地图片无法直接传给阿里云 API，必须先上传 OSS：
```bash
python scripts/oss_uploader.py 图片路径   # 返回公网 URL
```

**私有 Bucket（非公共读）**：在 `scripts/.env` 中配置 `OSS_SIGN_EXPIRATION=3600`（单位秒），
脚本会自动生成临时签名 URL 而非公共读 URL。签名过期前均可访问，无需开启公共读权限。

OSS 未配置时，告知用户：
> 「需要配置 OSS 才能使用本地图片。
>   你可以先用图片的公网 URL，或者我用纯生图模式替代。」

### 执行过程中的沟通

- 告知用户当前在做什么（「正在上传图片…」「正在调用试衣 API，约 30 秒…」）
- 失败时直接说清楚原因和建议，不要静默重试其他方案
- 见 `scripts/tryon_runner.py` 中的 `fail_fast` 机制

---

## 第五步：展示结果与调整

结果出来后的标准回复格式：

```
✅ 试穿效果图已生成！

[展示图片路径 / URL]

使用方案：aitryon-plus / 豆包生图
模特来源：柔妍（内置）/ 用户提供 / AI生成
保存位置：tryon_output/step3_tryon/xxx.jpg
耗时：约 28 秒
```

### 追问逻辑（每次试穿成功后都执行）

```
效果满意吗？接下来可以：

[1] 多拍几张 — 同模特同服装，生成 2~3 张不同角度/姿势
              （正面、侧面、转身，适合合成走秀视频）
[2] 换场景   — 同模特同服装，换背景风格（校园/街头/室内/棚拍）
[3] 生成视频 — 用当前已有的试穿图生成展示视频
[4] 调整重拍 — 换模特、换服装颜色、换局部
[5] 结束

回复数字，或直接说你想做什么。
用户说「1 完成后 3」、「先1再3」等组合 → 先执行 [1]，完成后接着执行 [3]。
```

> 💡 建议用户先选 [1] 或 [2] 多积累几张素材，再选 [3] 生成视频。
> 多张不同角度的图 → 视频镜头切换自然，效果远好于单张。

### 多角度图生成（用户选 [1]）

**必须同时传入 `--model-img` 和 `--garment-img`，通过 `--angle-preset` 触发多角度生成。**  
`image_gen_tryon.py` 通过 `generate_multi_angle()` 路由到多角度逻辑。

> **原理说明**：多角度生成基于**提示词差异 + 参考图锁定**实现。
> 豆包 Seedream 同时接收模特图（锁定人物）+ 服装图（锁定服装），
> 每个角度用不同 prompt 后缀（正面/侧面/背面/行走等），生成外观一致但姿势不同的图片。
> **不传参考图**则每张图的模特脸和服装都会随机变化，无法保证一致性。

#### 角度预设方案（必须展示给用户选择）

展示 4 种预设 + 自定义选项，让用户回复数字或组合：

```
📐 请选择拍摄角度方案：

[1] 电商标准拍（ecommerce）— 正面 + 侧面 + 背面，适合主图和详情页
[2] 走秀动态（catwalk）    — T台行走 + 转身回眸 + 定点造型，适合视频素材
[3] 细节特写（detail）      — 面料特写 + 领口细节 + 全身正面，展示工艺
[4] 生活场景（lifestyle）   — 自然站立 + 坐姿 + 街头漫步，日常穿搭感
[5] 自定义 — 描述你想要的角度

可以组合选择，如「1+3」= 电商标准拍 + 细节特写（共 6 张）
```

#### 调用方式

```bash
# 单个预设（默认 3 张）
python scripts/image_gen_tryon.py \
  --model-img {模特图} \
  --garment-img {服装图} \
  --angle-preset ecommerce \
  --garment-part top \
  --output-dir "$OUTPUT_DIR"

# 组合预设（逗号分隔，如电商+细节 = 6 张）
python scripts/image_gen_tryon.py \
  --model-img {模特图} \
  --garment-img {服装图} \
  --angle-preset ecommerce,detail \
  --garment-part one_piece \
  --output-dir "$OUTPUT_DIR"

# 指定输出张数（从预设角度中取前 N 个）
python scripts/image_gen_tryon.py \
  --model-img {模特图} \
  --garment-img {服装图} \
  --angle-preset catwalk \
  --variants 2 \
  --garment-part top \
  --output-dir "$OUTPUT_DIR"
```

> ⚠️ `--angle-preset` 模式直接调用 `generate_multi_angle()`，不走 `run()` 流程。
> 如果只传 `--garment-img` 而不传 `--model-img`，每次生成的模特脸/服装会随机变，
> 无法保证多角度一致性。**多角度生成必须同时传两张参考图。**

生成的图自动保存到 session 的 `step3_tryon/` 目录，与已有试穿图一起用于视频生成。

---

## 第七步：生成展示视频（可选）

### 生视频前必须先列图让用户选择

**步骤一：列出当前 session 所有试穿图**

```bash
python scripts/video_gen.py --list-images --output "$OUTPUT_DIR"
```

输出示例：
```
📸 当前 session 共 5 张试穿图（多图视频最多选 4 张）：

  [1] /path/task_xxx/step3_tryon/tryon_front_20260401_120000.png
  [2] /path/task_xxx/step3_tryon/tryon_side_20260401_120030.png
  [3] /path/task_xxx/step4_variants/variants_casual_20260401_121000.png
  [4] /path/task_xxx/step4_variants/variants_street_20260401_121030.png
  [5] /path/task_xxx/step4_variants/variants_studio_20260401_121100.png
```

**步骤二：展示给用户并请用户选择**

> 「已找到 5 张试穿图，请告诉我你想用哪几张合成视频（最多选 4 张）？
> 多图视频会自动切换镜头展示不同角度，选 2~4 张效果最好。
> 也可以只选 1 张，模型会让这张图里的模特动起来。」

**步骤三：按用户选择调用对应命令**

| 场景 | 脚本调用 |
|------|----------|
| 用户选 1 张 → 模特动起来 | `video_gen.py --image {选中路径} --prompt "..."` |
| 用户选 2~4 张 → 场景切换 | `video_gen.py --images {路径1} {路径2} ... --prompt "..."` |
| 控制开始和结束动作 | `video_gen.py --first-frame a.jpg --last-frame b.jpg --prompt "..."` |

> ⚠️ **多图视频用 `--images`（复数），单图视频用 `--image`（单数），不要混用。**
> 多图视频内部使用 `MODEL_LITE`（doubao-seedance-1-0-lite），
> 因为 `reference_image` 任务类型只有 Lite 模型支持，PRO 会返回 400 错误。

触发时机：用户说「帮我做成视频」「生成展示视频」「做个走秀效果」时。

视频 prompt 生成原则：

**电商视频规格标准：**

| 参数 | 规格 |
|------|------|
| 比例/分辨率 | **9:16 竖屏，1080×1920**（手机主流量优先） |
| 时长 | 5~10 秒（API 支持范围），核心卖点前置 |
| 格式 | H.264 / MP4 |
| 构图 | 人物居中，上下预留 10% 安全边距 |

**必须包含全身锁定 + 物理真实性 + 防黑背景约束（完整复制此模板）：**
```
full body always visible from head to toe throughout entire video,
model centered in frame with 10% margin top and bottom,
camera stays wide, no zoom in, no close-up, no cropping,
9:16 vertical format maintained,
smooth natural human motion only, physically realistic movement,
head and body rotate naturally — never reversed or counterclockwise,
no unnatural body distortion, no impossible physics,
clean white or light studio background, no black or dark background,
clothing details clearly visible throughout
```

- 1张图：锁定模板 + `"model slowly turns clockwise to show front and side, smooth catwalk, bright clean studio background"`
- 多张图：锁定模板 + `"each scene shows the complete outfit from head to toe, smooth transition, bright background"`
  - **注意**：多图 prompt 不要用 `[图1][图2]` 格式引用图片，直接描述视频内容即可
- `--ratio` 固定传 `"9:16"`，不用 `adaptive`
- **禁止黑色/深色背景**，如需特定场景背景也应保持明亮
- **禁止违反物理的动作**（如逆时针旋转头部等）
- `--output` 传到 `$OUTPUT_DIR/step5_video/` 下的具体文件名
- 注意：视频生成约需 1-3 分钟，告知用户等待

---

## 常见调整请求处理

| 用户说 | 怎么做 |
|--------|--------|
| 「换个背景」 | 模式 B，修改 scene_note 重新生成 |
| 「模特换丰满一点」 | 重新生成模特图，`plus-size` |
| 「只换上衣，裙子不变」 | `partial_tryon.py --model m.jpg --new-garment top.jpg --replace upper` |
| 「看看其他颜色」 | 重新着色服装图，再试穿 |
| 「生成几个对比一下」 | 模式 B，`--variants 3-5` |
| 「效果不好」 | 询问哪里不好，针对性调整；或切换模式 |
| 「API 报错了」 | 自动切换到模式 B，告知用户 |
| 「多拍几张」/「不同角度」 | 复用模特图+服装图，修改角度 prompt，生成 2~3 张 |
| 「侧面」/「背面」/「转身」 | 同上，指定具体角度 prompt |
| 「合成视频」/「做成视频」 | 收集所有已生成的试穿图，调用 video_gen.py |
| 「图片不是白底」/「去背景」/「换白底」 | 调用 `preprocess.py` 去背景白底化（见下方说明） |

### 服装图去背景 / 白底化

试衣 API 对白底服装图效果最好。当用户的服装图背景杂乱时，主动建议去背景：

> 「这张服装图的背景不是纯白色，建议先去背景再试穿，效果会更好。需要我帮你处理吗？」

用户同意后，根据图片来源选择处理方式：

**场景 1：本地图片 → 覆盖原文件**
```bash
python scripts/preprocess.py /path/to/garment.jpg --overwrite
```
处理后原文件被替换为白底图，后续流程直接使用。

**场景 2：本地图片 → 保留原图，另存处理结果**
```bash
python scripts/preprocess.py /path/to/garment.jpg -o /path/to/garment_white.jpg
```

**场景 3：URL 图片 → 下载去背景后上传 OSS 获取新 URL**
```bash
python scripts/preprocess.py https://example.com/garment.jpg --upload
```
脚本会自动下载 → 去背景 → 上传 OSS → 打印公网 URL，后续用新 URL 继续。

**场景 4：图片看起来是白底但效果不好 → 强制去背景**
```bash
python scripts/preprocess.py /path/to/garment.jpg --force --overwrite
```
`--force` 会跳过白底检测，强制执行去背景。

> ⚠️ `preprocess.py` 默认使用 `rembg`（本地免费），也可用 `--method removebg_api`（需配置 `REMOVEBG_API_KEY`，每月 50 次免费）。

---

## 执行检查清单

开始执行前确认：

- [ ] **输出目录已确定**（从 .env 读取 TRYON_OUTPUT_DIR 或确认默认值）
- [ ] **服装图理解已询问**（有服装图时必须问用户是否需要 AI 分析）
- [ ] **服装部位已确认**（上装 / 下装 / 上下装 / 单件连体）—— **必须询问，不允许跳过**
- [ ] 运行模式已选定（提示词图生图 默认 / 阿里云试衣 API 备选）
- [ ] **生图后端已选定**（豆包 Seedream / 即梦 AI）— 模式 B 时必须
- [ ] **合成方式已选定**（qwen / prompt）— 有双图+双 API 时
- [ ] 提示词已展示给用户并获得确认（或用户说「直接做」）
- [ ] 本地图片已上传 OSS（模式 A 且有本地图时）
- [ ] 模特图来源已确定（用户提供 / AI 生成）
- [ ] **`--garment-part` 已根据服装部位确认结果传入**（top/bottom/one_piece/full）
- [ ] **脚本调用命令包含 `--output-dir` 参数**

---

## 参考文件索引

| 文件 | 何时读取 |
|------|---------|
| `references/tryon-apis.md` | 需要写 API 调用代码时 |
| `references/image-models.md` | 需要生图 API 调用代码时 |
| `scripts/tryon_runner.py` | 阿里云试衣 API 主流程 |
| `scripts/image_gen_tryon.py` | 豆包/即梦生图 + 多角度生成（`generate_multi_angle`） |
| `scripts/partial_tryon.py` | 局部试穿 + 获取服饰 bbox |
| `scripts/oss_uploader.py` | 本地图片上传 OSS / base64 降级 |
| `scripts/garment_analyzer.py` | 服装图视觉理解（Qwen VL），输出结构化服装描述 |
| `scripts/output_manager.py` | 输出目录管理 / 文件保存 / session 日志 |
| `scripts/model_manager.py` | 动态读取 models.json / 模特列表展示 / 智能推荐 |
| `assets/models.json` | 模特数据源（模特增删改只需编辑此文件） |

---

## 脚本完整 CLI 参数速查

### image_gen_tryon.py
```bash
python scripts/image_gen_tryon.py \
  --model-img MODEL_URL_OR_PATH \       # 模特图
  --garment-img GARMENT_URL_OR_PATH \   # 服装图
  --desc "服装描述" \                     # 文字描述
  --prompt "自定义prompt" \               # 直接指定 prompt
  --style "日系" \                        # 风格偏好
  --variants 3 \                         # 变体数量 1~5
  --size 2K \                            # 图片尺寸
  --image-backend douban \               # 生图后端：auto/jimeng/douban
  --garment-part top \                   # 服装部位：top/bottom/one_piece/full
  --angle-preset ecommerce \             # 多角度预设：ecommerce/catwalk/detail/lifestyle（可逗号组合）
  --output-dir /path/to/output \         # 输出目录（必传）
  --output-filename result.jpg           # 输出文件名（可选）
```

> ⚠️ `--angle-preset` 需同时传 `--model-img` 和 `--garment-img`，走 `generate_multi_angle()` 路径，不经过 `run()`。
> `--garment-part` 在两种模式下都生效：`run()` 用它选择服装锁定前缀，`--angle-preset` 同理。

### tryon_runner.py
```bash
python scripts/tryon_runner.py \
  --garment GARMENT_URL_OR_PATH \       # 服装图
  --model MODEL_URL_OR_PATH \           # 模特图
  --desc "服装描述" \                     # 文字描述
  --garment-part top \                   # 服装部位：top=上装/bottom=下装/full=上下装/one_piece=单件连体
  --output-dir /path/to/output \         # 输出目录（必传）
  --synthesis-method prompt \            # 合成方式：prompt（默认）/qwen/auto
  --skip-preprocess                      # 跳过预处理
```

### video_gen.py
```bash
python scripts/video_gen.py \
  --image IMAGE_PATH \                  # 单张图（图生视频）
  --images IMG1 IMG2 IMG3 \             # 多张图
  --prompt "视频描述" \                   # 视频 prompt
  --duration 5 \                         # 时长秒数
  --ratio 9:16 \                         # 画面比例
  --output /path/to/output/video.mp4     # 输出路径（必传）
```
| `scripts/preprocess.py` | 服装图去背景 |

### preprocess.py
```bash
python scripts/preprocess.py /path/to/garment.jpg                 # 去背景（默认 rembg）
python scripts/preprocess.py /path/to/garment.jpg --overwrite      # 覆盖原文件
python scripts/preprocess.py /path/to/garment.jpg --force          # 强制处理（跳过白底检测）
python scripts/preprocess.py https://example.com/g.jpg --upload    # URL 图→去背景→上传 OSS
python scripts/preprocess.py garment.jpg -o out.jpg                # 指定输出路径
python scripts/preprocess.py garment.jpg --method removebg_api     # 使用 remove.bg API
```

### garment_analyzer.py
```bash
python scripts/garment_analyzer.py /path/to/garment.jpg          # 分析服装图，展示格式化结果
python scripts/garment_analyzer.py https://example.com/g.jpg      # 支持 URL
python scripts/garment_analyzer.py /path/to/garment.jpg --json    # 输出 JSON（供脚本间传参）
python scripts/garment_analyzer.py /path/to/g.jpg --prompt "自定义提示词"  # 自定义分析角度
```

需要配置 `DASHSCOPE_API_KEY`（或复用 `ALIYUN_API_KEY`）。本地图片需要 OSS 配置。

### model_manager.py
```bash
python scripts/model_manager.py list                  # 列出全部模特（按性别/年龄分组）
python scripts/model_manager.py recommend "JK制服"     # 根据服装描述智能推荐模特
python scripts/model_manager.py validate /path/to.jpg  # 校验用户上传模特图
```

---
> Source: [Anatomic-universalquantifier643/shop-tryon-skill](https://github.com/Anatomic-universalquantifier643/shop-tryon-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
