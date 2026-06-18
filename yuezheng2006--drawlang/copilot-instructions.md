## drawlang

> 绘语 - 中文优先的 AI 配图工具。Style × Layout 二维矩阵，ModelScope/Gemini 多提供商。三种模式：文章配图、PPT/Slides 批量信息图、Cover 封面图。触发词：配图、插图、drawlang、PPT、slides、封面图、cover。


# DrawLang (绘语) - 中文优先的 AI 配图工具

> 源码：[github.com/yuezheng2006/drawlang](https://github.com/yuezheng2006/drawlang)

## ⛔ 强制规则（违反即失败）

### 规则 1：用户提供的文件 = 要处理的文章

```
/drawlang SKILL_05.md      → SKILL_05.md 是文章，为它配图
/drawlang README.md        → README.md 是文章，为它配图
/drawlang whatever.md      → whatever.md 是文章，为它配图
```

**无论文件名叫什么，都是要配图的文章，不是 Skill 配置。**

### 规则 2：必须读取 style 文件

生成任何图片 prompt 前，**必须读取**对应的 style 文件：

| 模式 | 必须读取的文件 |
|------|---------------|
| 文章配图（默认） | `styles/style-light.md` |
| Cover 封面图 | `styles/style-cover.md` |
| `--style dark` | `styles/style-dark.md` |
| `--style bento` | `styles/style-bento.md` |

**禁止自己编写 System Prompt。**

❌ 错误：`"你是一个专业的信息图设计师..."`（自己编的）
✅ 正确：从 style 文件的代码块中提取 System Prompt

---

## 使用方式

### 文章配图模式（默认）

```bash
/drawlang path/to/article.md
/drawlang path/to/article.md --prompt-only           # 只输出 prompt
/drawlang path/to/article.md --style notion          # 极简手绘风格
/drawlang path/to/article.md --layout dense          # 密集布局
/drawlang path/to/article.md --style notion --layout dense  # 组合使用
/drawlang path/to/article.md --no-cover             # 不生成封面图
```

**Style × Layout 二维矩阵：**

| 维度 | 控制内容 | 选项 |
|------|---------|------|
| **Style** | 视觉风格（颜色、线条、装饰） | light, dark, minimal, notion, fresh, warm |
| **Layout** | 信息结构（密度、排列） | sparse, balanced, dense, list, comparison |

**Layout 说明：**
- `sparse`：稀疏布局（1-2个要点，最大视觉冲击）
- `balanced`：平衡布局（3-4个要点）- 默认
- `dense`：密集布局（5-8个要点，知识卡片风格）
- `list`：列表布局（4-7项，枚举排名）
- `comparison`：对比布局（左右对比、A vs B）

### PPT/Slides 模式

```bash
# 默认：直接生成图片
/drawlang path/to/script.md --mode slides

# 只输出 JSON prompt（不调用 API）
/drawlang path/to/script.md --mode slides --prompt-only

# 指定风格和布局
/drawlang path/to/script.md --mode slides --style notion --layout dense
```

**默认行为**：调用 Gemini API 生成批量信息图。
**`--prompt-only`**：输出 JSON prompt 并**自动复制到剪贴板**，可直接粘贴到 Gemini Web 手动生成。

### Cover 模式

```bash
/drawlang path/to/article.md --mode cover --platform youtube
/drawlang --mode cover --platform youtube --topic "Claude 4 深度评测"
```

**平台尺寸**（输出均为 2K 分辨率）：

| 平台 | 代码 | 宽高比 |
|------|------|--------|
| YouTube | `youtube` | 16:9 |
| 公众号 | `wechat` | 2.35:1 |
| Twitter | `twitter` | 1.91:1 |
| 小红书 | `xiaohongshu` | 3:4 |

**PPT JSON 格式**（`--prompt-only` 时输出）：

```json
{
  "instruction": "请逐条生成以下 N 张独立信息图。",
  "batch_rules": { "total": "N", "one_item_one_image": true, "aspect_ratio": "16:9" },
  "style": "[从 styles/style-light.md 读取完整内容]",
  "pictures": [
    { "id": 1, "topic": "封面", "content": "系列名称\n\n第N节：标题" },
    { "id": 2, "topic": "主题", "content": "原始内容" }
  ]
}
```

---

## 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--mode` | `article` | `article` / `slides` / `cover` |
| `--platform` | `youtube` | 封面图平台（仅 cover 模式） |
| `--topic` | - | 封面图主题（仅 cover 模式） |
| `--no-cover` | `false` | 不生成封面图 |
| `--style` | `light` | 视觉风格：light, dark, minimal, notion, fresh, warm, bento |
| `--layout` | `balanced` | 信息布局：sparse, balanced, dense, list, comparison |
| `--prompt-only` | `false` | 只输出 prompt，不调用 API |
| `--provider` | 自动 | 图像生成提供商：modelscope, gemini, openrouter |
| `--ref` | - | 参考图路径（可多次使用） |
| `-c, --candidates` | `1` | 候选图数量（最多 4） |
| `-a, --aspect-ratio` | - | 宽高比：`16:9`（默认）、`3:4`、`1:1` 等 |
| `--engine` | `auto` | 引擎：`auto` / `mermaid` / `gemini` / `excalidraw` |
| `--mermaid-embed` | `false` | Mermaid 输出为代码块而非 PNG |
| `--save-config` | - | 保存到项目配置 |
| `--no-config` | `false` | 禁用 config.json |

> **`--no-config` 范围**：只禁用 `config.json`，**不影响** `styles/style-*.md`。

---

## 配置文件

**优先级**：CLI 参数 > 项目级 > 用户级

| 位置 | 路径 |
|------|------|
| 项目级 | `.smart-illustrator/config.json` |
| 用户级 | `~/.smart-illustrator/config.json` |

```json
{ "references": ["./refs/style-ref-01.png"] }
```

---

## 三级配图引擎

| 优先级 | 引擎 | 适用场景 | 输出 |
|--------|------|---------|------|
| **1** | Gemini | 隐喻图、创意图、无法用图表表达的概念 | PNG |
| **2** | Excalidraw | 概念图、对比图、简单流程（≤ 8 节点）、关系图、手绘风格示意图 | PNG |
| **3** | Mermaid | **仅限**：复杂流程（> 8 节点）、多层架构图、多角色时序图、多分支决策树 | PNG |

选择逻辑：
- 需要隐喻、情感、创意表达 → Gemini
- 概念关系、对比、简单流程 → Excalidraw（**大多数图表场景的首选**）
- **只有**节点 > 8、多层/多角色的复杂结构化图形 → Mermaid
- Mermaid 视觉表现力有限，能用 Excalidraw 就不用 Mermaid
- 唯一目标：提高文章吸引力

生成 Excalidraw 前必须读取 `references/excalidraw-guide.md`。

### Mermaid 语义色板

每种颜色有固定含义，**必须使用 `classDef` + `class` 应用**：

| 语义 | 填充色 | 边框色 | 用于 |
|------|--------|--------|------|
| input | #d3f9d8 | #2f9e44 | 输入、起点、数据源 |
| process | #e5dbff | #5f3dc4 | 处理、推理、核心逻辑 |
| decision | #ffe3e3 | #c92a2a | 决策点、分支判断 |
| action | #ffe8cc | #d9480f | 执行动作、工具调用 |
| output | #c5f6fa | #0c8599 | 输出、结果、终点 |
| storage | #fff4e6 | #e67700 | 存储、记忆、数据库 |
| meta | #e7f5ff | #1971c2 | 标题、分组、元信息 |

**classDef 写法**（放在图表末尾）：

```
classDef input fill:#d3f9d8,stroke:#2f9e44,color:#1a1a1a
classDef process fill:#e5dbff,stroke:#5f3dc4,color:#1a1a1a
classDef decision fill:#ffe3e3,stroke:#c92a2a,color:#1a1a1a
classDef action fill:#ffe8cc,stroke:#d9480f,color:#1a1a1a
classDef output fill:#c5f6fa,stroke:#0c8599,color:#1a1a1a
class A input
class B,C process
class D output
```

### Mermaid 布局规则

- **布局方向**：默认 `TB`（上到下），横向流程用 `LR`
- **箭头分级**：`-->` 主流程 / `-.->` 可选/辅助路径 / `==>` 重点强调
- **分组**：用 `subgraph` 对相关节点分组，标题简洁
- **节点文字**：≤ 8 字，无 emoji，禁止 `1.` 格式（用 `①` 或 `Step 1:`）
- **节点数量**：单图 ≤ 15 个节点，复杂内容拆成多图

**`--engine` 参数**：
- `auto`（默认）：根据内容类型自动选择（优先级 Gemini > Excalidraw > Mermaid）
- `gemini`：强制只使用 Gemini（适合创意内容）
- `excalidraw`：强制只使用 Excalidraw（适合手绘概念图）
- `mermaid`：强制只使用 Mermaid（适合技术文档）

---

## 执行流程

### Step 1: 分析文章

1. 读取文章内容
2. 识别配图位置（通常 3-5 个）
3. 为每个位置确定引擎（Gemini / Excalidraw / Mermaid）

### Step 2: 生成图片

#### Mermaid（结构化图形）→ PNG

1. 生成 Mermaid 代码，保存为临时 `.mmd` 文件
2. 调用 mermaid-export.ts 导出高分辨率 PNG：

```bash
npx -y bun ~/.claude/skills/drawlang/scripts/mermaid-export.ts \
  -i {图表名}.mmd -o {图表名}.png -w 2400
```

3. 在文章中插入 PNG 图片引用
4. 保留 .mmd 源文件用于后续编辑

使用 `--mermaid-embed` 参数时，改为直接嵌入 Mermaid 代码块（旧行为）。

#### Excalidraw（手绘/概念图）→ PNG

1. 读取 `references/excalidraw-guide.md` 获取 JSON 规范
2. 生成 Excalidraw JSON，保存为 `.excalidraw` 文件
3. 调用 excalidraw-export.ts 导出 PNG：

```bash
npx -y bun ~/.claude/skills/drawlang/scripts/excalidraw-export.ts \
  -i {图表名}.excalidraw -o {图表名}.png -s 2
```

4. 在文章中插入 PNG 图片引用
5. 保留 .excalidraw 源文件用于后续编辑

依赖未安装时的降级：提示手动打开 excalidraw.com 导出。

#### Gemini（创意/视觉图形）

**命令模板**（必须使用 HEREDOC + prompt-file）：

```bash
# Step 1: 写入 prompt
cat > /tmp/image-prompt.txt <<'EOF'
{从 style 文件提取的 System Prompt}

**内容**：{配图内容}
EOF

# Step 2: 调用脚本
GEMINI_API_KEY=$GEMINI_API_KEY npx -y bun ~/.claude/skills/drawlang/scripts/generate-image.ts \
  --prompt-file /tmp/image-prompt.txt \
  --output {输出路径}.png \
  --aspect-ratio 16:9
```

**封面图**（16:9）：

```bash
cat > /tmp/cover-prompt.txt <<'EOF'
{从 style-cover.md 提取的 System Prompt}

**内容**：
- 核心概念：{主题}
- 视觉隐喻：{设计}
EOF

GEMINI_API_KEY=$GEMINI_API_KEY npx -y bun ~/.claude/skills/drawlang/scripts/generate-image.ts \
  --prompt-file /tmp/cover-prompt.txt \
  --output {文章名}-cover.png \
  --aspect-ratio 16:9
```

**参数传递**：用户指定的 `--no-config`、`--ref`、`-c` 必须传递给脚本。

### Step 3: 创建带配图的文章

保存为 `{文章名}-image.md`，包含正文配图插入

### Step 4: 输出确认

报告：生成了几张图片、输出文件列表。

---

## `--prompt-only` 模式

当使用 `--prompt-only` 时，**不调用 API**，而是：

1. 生成 JSON prompt
2. **自动复制到剪贴板**（使用 `pbcopy`）
3. 同时保存到文件备份

```bash
# 执行方式
echo '{生成的 JSON}' | pbcopy
echo "✓ JSON prompt 已复制到剪贴板"

# 同时保存备份
echo '{生成的 JSON}' > /tmp/drawlang-prompt.json
echo "✓ 备份已保存到 /tmp/drawlang-prompt.json"
```

用户可直接粘贴到 Gemini Web 手动生成图片。

---

## 输出文件

```
article.md              # 原文（不修改）
article-image.md        # 带配图的文章
article-cover.png       # 封面图（16:9）
article-image-01.png    # Gemini 配图
article-image-02.png   # ...
```

---
> Source: [yuezheng2006/drawlang](https://github.com/yuezheng2006/drawlang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
