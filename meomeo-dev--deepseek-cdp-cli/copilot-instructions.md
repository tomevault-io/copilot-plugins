## deepseek-cdp-cli

> Use this skill when you need DeepSeek to research public web information, fact-check claims, collect source links, summarize or compare uploaded files, or analyze images and UI screenshots into structured observations.


# DeepSeek

本手册给人和无状态 LLM 提供 `deepseek` CLI 的基本使用框架。

## 核心认知

- `deepseek` 驱动本机 Chrome / Chromium 里的 DeepSeek 网页，不是裸 HTTP 客户端。
- 首次使用先运行 `deepseek auth login`。登录后，普通命令默认使用 DeepSeek 专用 profile。
- `--clone-chrome-profile` 是 legacy：从普通 Chrome source profile 复制 DeepSeek 登录态；不要把它当成 auth profile。
- `deepseek` 适合开放网页搜索、摘要、候选链接整理、文件上传、会话继续和导出。
- `deepseek` 不负责结构化行情数据库、数值型截面查询或研究状态管理。
- 首选命令是 `deepseek`。`deepseek-cdp` 和 `deepseek-cdp-cli` 只是兼容别名。
- `deepseek version` 会返回当前包版本、GitHub 地址和 License，适合 LLM 做能力/版本自检。
- `deepseek skillbook` 会原样返回当前 `SKILL.md`。
- 当前只适配 macOS（only supported platform）。

## 安装

```sh
npm install -g deepseek-cdp-cli
```

常用入口：

```sh
deepseek --help
deepseek version
deepseek skillbook
```

## 使用节奏

### 1. 登录

```sh
deepseek auth login
```

`auth login` 会打开可见 Chrome。用户在 DeepSeek 网页完成登录后，CLI 会保存专用 profile：

```text
~/.deepseek-cdp-cli/auth/chrome-profile
```

清理专用 profile：

```sh
deepseek auth logout
```

`auth logout` 只删除 `~/.deepseek-cdp-cli/auth`，不会删除用户自己的 Chrome profile。

### 2. 看运行计划

```sh
deepseek plan
```

`plan` 用来确认本次命令的 request family、是否会 auto-isolate、是否会 attach 到已有 CDP 浏览器。
`plan` 不接收 `--chat-mode`、`--file`、`--message` 这类发送参数；
这些参数只放在 `deepseek reply`。

默认规则：

- 有 auth profile：普通 one-shot 命令优先使用专用 profile，留在 managed 路径。
- 没有 auth profile：普通 one-shot 命令默认尝试 attach 到 `http://127.0.0.1:9222`。
- 显式 `--browser-id`、`--browser-mode`、`--cdp-url` 会固定意图；冲突时 fail-closed。
- auth profile 或 legacy clone 请求不会 auto attach 到用户现成浏览器。

### 3. 发第一条消息

```sh
deepseek reply \
  --message "用三句话介绍这个项目" \
  --headless \
  --quiet \
  --format text
```

`--quiet` 只影响 runtime logs，不会移除文本输出里的 `sessionId`。

```text
sessionId: ...
```

拿到 `sessionId` 后继续：

```sh
deepseek reply --session-id <sessionId> --message "继续" --quiet --format text
```

默认 `reply` 使用 Expert + DeepThink，Search 当前关闭：

```text
--chat-mode expert --deep-think on --search off
```

DeepSeek 官网当前因算力不足暂时隐藏 Expert 的联网搜索和附件入口。
因此 `--chat-mode expert --search on` 与
`--chat-mode expert --file ...` 都会提前 fail-closed。需要联网时先用
Instant：

```sh
deepseek reply \
  --message "联网检索一个公开事实并给出来源" \
  --headless \
  --chat-mode instant \
  --deep-think on \
  --search on \
  --quiet \
  --format text
```

### 4. 找回网页里的旧会话

如果网页里有会话，但本地还没有，先同步 catalog：

```sh
deepseek sync-session --headless
deepseek list-sessions
```

边界：

- `deepseek list-sessions` 只列本地已经持久化的会话，不是远端账号历史目录。
- 不带 `--session-id` / `--session-file` 的 `deepseek sync-session` 只同步网页可观测到的 online catalog，并把结果落成本地 catalog snapshot / placeholder。
- 这一步不会逐个进入会话页。
- 这一步不会同步 transcript / branches / messages。
- 这一步不是稳定 public API。

### 5. 刷新单个会话正文

如果网页 transcript 比本地新，先 targeted sync：

```sh
deepseek sync-session --session-id <sessionId> --headless
```

`deepseek sync-session --session-id <id>` 或 `--session-file <path>` 才是显式单会话 transcript sync。

记住：

- `reply` 不会自动 sync。
- `export-session` 不会自动 sync。
- `list-branches` 不会自动 sync。

### 6. 继续、查看分支、导出

```sh
deepseek reply --session-id <sessionId> --message "继续说" --stream --quiet --format text
deepseek list-branches --session-id <sessionId>
deepseek export-session --session-id <sessionId> --format text --output ./session.txt
deepseek export-session --session-id <sessionId> --format markdown --output ./session.md
deepseek export-session --session-id <sessionId> --format json --output ./session.json
```

## 投研与网页搜索

投研默认命令：

先创建 prompt 文件。命名规范：

```text
deepseek-research-<topic-kebab>-<YYYYMMDD>.prompt.md
```

示例：

```sh
cp deepseek-research-template.prompt.md deepseek-research-feilong-inquiry-20260425.prompt.md
```

再执行：

```sh
deepseek reply \
  --message "$(cat deepseek-research-feilong-inquiry-20260425.prompt.md)" \
  --headless \
  --chat-mode instant \
  --search on \
  --deep-think on \
  --quiet \
  --format text
```

`deepseek-research-template.prompt.md` 模板。原样复制；只修改 `已知上下文` 和 `任务`，其他部分保持原样。

```markdown
深度搜索, 逐步的, 每步搜索一个查询, 逐步串行地从互联网中搜索收集所需的多个维度信息:
1. ...
2. ...
3. ...
4. ...
5. ...
6. ...
7. ...

我们需要同时探索多个方向, 针对研究中的缺口进行补充搜索。

当上下文未提供必要属性所需的数据信息时：
- 若该信息**无法通过上下文推断或合理派生**，则使用显式标记默认占位填充（templated placeholder, UPPER_CASE）提示用户补充
- 若该信息**可通过上下文中的其他信息推断得出**，则可直接使用推断值
- **禁止**自行生成"看似真实"但实际未提供的具体数据（如虚构的ID、时间戳、用户名等）

附加格式要求:

1. 用摘要或关键词概括那个连续段落的核心内容，并放在问题之前, 总-分的写作方法。
2. 在连续段落的开头或结尾加入信号词, 信号词标记方式与回复格式保持一致, 比如markdown时请使用 `**` 加粗标记，并保持格式统一。信号词中英文对照, 术语英文放到括号里。只需要使用标记方式标记信号词即可, 无需出现“信号词”字眼，保持自然行文。
3. 自然地集中出现该段落内容主题独有的专有名词或术语，但不要过度堆砌，保持自然行文。
4. 使用任何能别出“章节边界”、并促使章节内部术语集中的标记方式分割长文档(默认为(markdown+mathjax+补充HTML片段(内联样式))作为回复格式, 否则按照用户指定格式回复), 使用明显的标题或分隔符标记出章节边界，使得每个章节内部的块更加同质。
5. 对于上下文要额外注意, 由于注意力机制, 召回上下文时会召回紧密衔接的内容，上下文可能无法清晰切割, 因此需要注意用户问题的严格边界。
6. 每次回复需要加上唯一ID(格式: 回复ID: <最最重要的那个关键词,驼峰写法>-<YYYYMMDD日期>-<\d{3}自增长>), 方便后序引用(引用格式: `[inner: 回复ID]`)。

数学公式使用 mathjax 表达, 特殊token表达到markdown代码块中。

已知上下文:

- 附件文件名 , 内容一句话说明
- 若有多个则列出...

任务:

你的检索问题
```

适合：

- 公开网页搜索
- 跨站点线索发现
- 公司官网、交易所公告、权威媒体、行业媒体、公开研报摘要补充阅读
- 候选链接清单整理

不适合：

- 结构化行情数据库
- 数值型截面和时间序列主查询
- 最终投资结论

证据边界：

- `deepseek` 输出默认是开放网页候选证据，不直接等于高可信结论。
- 聚合摘要、二手转载、自媒体、论坛内容、未标明原始出处的网页整理都应降权。
- 重要事实尽量回到原始公告、公司原文、交易所或巨潮页面、权威媒体原文、正式研报页面。

常用问题写法：

- “请联网检索并概括截至 YYYY-MM-DD 最近一周最关键的 4 条催化和 4 条风险，优先公司公告、权威媒体、券商研报摘要。”
- “请列出最值得二次核验的 5 个公开网页链接，并标明来源类型，不要直接给投资建议。”
- “请比较 A 公司与同主题可比公司在公开网页中的主要叙事差异。”

## 附件

当前 DeepSeek 因算力不足暂时隐藏 Expert 附件入口，
`--chat-mode expert --file ...` 会提前报错；图片上传先使用 Vision
路径。

```sh
deepseek reply \
  --message "描述这张图片" \
  --headless \
  --chat-mode vision \
  --file ./image.png \
  --quiet \
  --format text
```

识图模式：

```sh
deepseek reply \
  --message "请用一句话描述这张图" \
  --headless \
  --chat-mode vision \
  --file ./image.png \
  --quiet \
  --format text
```

规则：

- `--file` 可以重复。
- `--chat-mode` 和 `--file` 只属于 `deepseek reply`。
- 不要把它们放到 `deepseek plan`。
- Expert + Search 暂时禁用，等待 DeepSeek 算力恢复后再重开。
- Expert + file 暂时禁用，等待 DeepSeek 恢复 Expert 附件入口后再重开。
- 图片识别使用浏览器 `--chat-mode vision --file <image>`；这不是 OpenAI HTTP chat 多模态 content 或 `/v1/files`。
- 当前官网 vision 模式只稳定提供上传文件 + DeepThink；如果命令同时传入 `--search on|off`，CLI 会按 no-op 忽略搜索请求，不点击或等待智能搜索按钮。
- 文件能力取决于最终页面 surface。
- 如果页面没有真实文件输入框，命令会 fail-closed。

## Vision 模式结构化模板

当用户提供图片并希望获得稳定可复用的数据时，不要只写“描述这张图”。优先把“任务指令 + JSON Schema + 未识别处理规则”一起交给 Vision 模式，让模型按 schema 抽取图像数据。

基础命令：

```sh
deepseek reply \
  --chat-mode vision \
  --file ./image.png \
  --message "$(cat vision-ui-elements.prompt.md)" \
  --quiet \
  --format text
```

如果下一步要程序解析 JSON，改用 `--format json --json-shape native`；
`--format text` 适合人读，不保证整个 stdout 是纯 JSON。

通用提示词骨架：

```markdown
你是图像结构化识别器。请只基于图片中可见内容输出 JSON。

任务：
- <说明本次要识别什么，例如 UI 元素、版式、设计系统、故事场景、图像生成提示词>

输出要求：
- 只输出一个 JSON object，不要 Markdown 代码块，不要解释。
- 输出必须符合下面的 JSON Schema。
- 不要编造图片中不可见的信息。
- 无法识别的字段填 `null`；无法确认的列表填 `[]`。
- 每个识别项必须给 `confidence`，范围 0 到 1。
- 每个识别项必须给 `evidence`，用短语说明来自图片中的哪类可见线索。
- 只有能从图片中估计位置时才填写 `bbox`；否则填 `null`。
- `bbox` 使用归一化坐标 `{ "x": 0-1, "y": 0-1, "w": 0-1, "h": 0-1 }`。
- 如果 schema 的枚举无法准确覆盖，使用 `unknown`，并在 `notes` 说明原因。

JSON Schema:
<粘贴 schema>
```

### UI 元素识别 Schema

适合从截图、网页、App、仪表盘里抽取 UI 元素。用有限 `type` 枚举控制漂移，避免模型把组件随意命名。

```json
{
  "type": "object",
  "additionalProperties": false,
  "required": ["imageSummary", "elements", "unreadableAreas"],
  "properties": {
    "imageSummary": { "type": "string" },
    "elements": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["id", "type", "label", "text", "bbox", "state", "confidence", "evidence", "notes"],
        "properties": {
          "id": { "type": "string", "description": "stable id such as el_001" },
          "type": {
            "type": "string",
            "enum": ["text", "button", "input", "textarea", "select", "checkbox", "radio", "toggle", "icon", "image", "avatar", "card", "nav", "tab", "table", "chart", "modal", "toast", "divider", "container", "unknown"]
          },
          "label": { "type": ["string", "null"] },
          "text": { "type": ["string", "null"] },
          "bbox": {
            "type": ["object", "null"],
            "additionalProperties": false,
            "required": ["x", "y", "w", "h"],
            "properties": {
              "x": { "type": "number", "minimum": 0, "maximum": 1 },
              "y": { "type": "number", "minimum": 0, "maximum": 1 },
              "w": { "type": "number", "minimum": 0, "maximum": 1 },
              "h": { "type": "number", "minimum": 0, "maximum": 1 }
            }
          },
          "state": {
            "type": "string",
            "enum": ["default", "hover", "active", "selected", "disabled", "error", "loading", "unknown"]
          },
          "confidence": { "type": "number", "minimum": 0, "maximum": 1 },
          "evidence": { "type": "string" },
          "notes": { "type": ["string", "null"] }
        }
      }
    },
    "unreadableAreas": {
      "type": "array",
      "items": { "type": "string" }
    }
  }
}
```

### 看图讲故事 Schema

适合描述图片内容、人物、动作、氛围和叙事线索。要求把“事实观察”和“合理推断”拆开，避免把故事写成事实。

```json
{
  "type": "object",
  "additionalProperties": false,
  "required": ["literalDescription", "subjects", "setting", "actions", "mood", "storyHypothesis", "uncertainties"],
  "properties": {
    "literalDescription": { "type": "string" },
    "subjects": { "type": "array", "items": { "type": "string" } },
    "setting": { "type": ["string", "null"] },
    "actions": { "type": "array", "items": { "type": "string" } },
    "mood": { "type": ["string", "null"] },
    "storyHypothesis": {
      "type": "object",
      "additionalProperties": false,
      "required": ["summary", "confidence", "evidence"],
      "properties": {
        "summary": { "type": ["string", "null"] },
        "confidence": { "type": "number", "minimum": 0, "maximum": 1 },
        "evidence": { "type": "array", "items": { "type": "string" } }
      }
    },
    "uncertainties": { "type": "array", "items": { "type": "string" } }
  }
}
```

### Layout 版式分析 Schema

适合分析网页、海报、PPT、平面设计的布局问题。输入可以是原图，也可以是上一步 UI 元素 JSON。

```json
{
  "type": "object",
  "additionalProperties": false,
  "required": ["canvas", "regions", "hierarchy", "alignment", "spacing", "layoutIssues"],
  "properties": {
    "canvas": {
      "type": "object",
      "additionalProperties": false,
      "required": ["orientation", "density", "dominantFlow"],
      "properties": {
        "orientation": { "type": "string", "enum": ["portrait", "landscape", "square", "unknown"] },
        "density": { "type": "string", "enum": ["low", "medium", "high", "unknown"] },
        "dominantFlow": { "type": "string", "enum": ["top-to-bottom", "left-to-right", "grid", "radial", "unknown"] }
      }
    },
    "regions": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["id", "role", "bbox", "containsElementIds", "confidence"],
        "properties": {
          "id": { "type": "string" },
          "role": { "type": "string", "enum": ["header", "sidebar", "hero", "content", "form", "toolbar", "chart", "footer", "unknown"] },
          "bbox": { "type": ["object", "null"] },
          "containsElementIds": { "type": "array", "items": { "type": "string" } },
          "confidence": { "type": "number", "minimum": 0, "maximum": 1 }
        }
      }
    },
    "hierarchy": { "type": "array", "items": { "type": "string" } },
    "alignment": { "type": "array", "items": { "type": "string" } },
    "spacing": { "type": "array", "items": { "type": "string" } },
    "layoutIssues": { "type": "array", "items": { "type": "string" } }
  }
}
```

### Stylist / 设计系统 Schema

适合从截图中抽取 UI 风格、颜色、字体、组件视觉语言。注意：字体名通常不可从图片精确确认，必须允许 `null` 和低置信度。

```json
{
  "type": "object",
  "additionalProperties": false,
  "required": ["styleSummary", "colors", "typography", "components", "effects", "designSystemHypothesis"],
  "properties": {
    "styleSummary": { "type": "string" },
    "colors": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["role", "approxHex", "confidence"],
        "properties": {
          "role": { "type": "string", "enum": ["background", "surface", "primary", "secondary", "accent", "text", "mutedText", "border", "success", "warning", "danger", "unknown"] },
          "approxHex": { "type": ["string", "null"] },
          "confidence": { "type": "number", "minimum": 0, "maximum": 1 }
        }
      }
    },
    "typography": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["role", "fontFamilyGuess", "sizeCategory", "weight", "confidence"],
        "properties": {
          "role": { "type": "string", "enum": ["heading", "body", "caption", "label", "code", "unknown"] },
          "fontFamilyGuess": { "type": ["string", "null"] },
          "sizeCategory": { "type": "string", "enum": ["xs", "sm", "md", "lg", "xl", "xxl", "unknown"] },
          "weight": { "type": "string", "enum": ["regular", "medium", "semibold", "bold", "unknown"] },
          "confidence": { "type": "number", "minimum": 0, "maximum": 1 }
        }
      }
    },
    "components": { "type": "array", "items": { "type": "string" } },
    "effects": { "type": "array", "items": { "type": "string" } },
    "designSystemHypothesis": {
      "type": "object",
      "additionalProperties": false,
      "required": ["nameOrStyle", "confidence", "evidence"],
      "properties": {
        "nameOrStyle": { "type": ["string", "null"] },
        "confidence": { "type": "number", "minimum": 0, "maximum": 1 },
        "evidence": { "type": "array", "items": { "type": "string" } }
      }
    }
  }
}
```

### 图像提示词 Schema

适合把图片反向整理成可给生图模型使用的提示词。必须区分可见事实、风格词和不确定推断。

```json
{
  "type": "object",
  "additionalProperties": false,
  "required": ["prompt", "negativePrompt", "styleTags", "composition", "visibleFacts", "uncertainInferences"],
  "properties": {
    "prompt": { "type": "string" },
    "negativePrompt": { "type": ["string", "null"] },
    "styleTags": { "type": "array", "items": { "type": "string" } },
    "composition": { "type": "array", "items": { "type": "string" } },
    "visibleFacts": { "type": "array", "items": { "type": "string" } },
    "uncertainInferences": { "type": "array", "items": { "type": "string" } }
  }
}
```

复杂任务建议拆成多轮，不要一次让模型同时完成所有推断：

1. 第一轮：图片 -> `UI 元素识别 JSON`。
2. 第二轮：图片 + UI 元素 JSON -> `Layout 版式 JSON`。
3. 第三轮：图片 + UI 元素 JSON + Layout JSON -> `Stylist 设计系统 JSON`。
4. 第四轮：元素 + layout + stylist -> 页面 JSON 表达、重绘 SVG、HTML/CSS、设计评审或改版建议。

多轮时，把上一轮 JSON 作为下一轮 prompt 的“已知结构化输入”，并要求模型只修正有新证据支持的字段；没有证据时保留 `null`、`[]` 或低置信度，不要补全成看似完整但不可验证的数据。

## 输出

日常文本：

```sh
deepseek reply --message "..." --quiet --format text
```

机器读取：

```sh
deepseek reply --message "..." --format json --json-shape native
deepseek reply --message "..." --stream --format stream-json --json-shape native
```

规则：

- 不带 `--stream`：`--format text` 或 `--format json`。
- 带 `--stream`：`--format text` 或 `--format stream-json`。
- `--json-shape` 只给 JSON 家族使用。
- `--format text` 可能带 `sessionId` 页脚；不要直接当 JSON 解析。
- `reply --format text` 是本次即时输出；`export-session --format text` 是已持久化会话导出。

## 登录与运行选择

普通使用只需要2条规则：

- 先 `deepseek auth login`。
- 不确定时先 `deepseek plan`。

需要固定运行方式时再使用这些参数：

- `--browser-id <id>`：复用指定 warm runtime。
- `--browser-mode attach|ephemeral|warm`：固定生命周期。
- `--cdp-url <url>`：固定 CDP endpoint。

交互和复用：

```sh
deepseek reply --browser-id <browserId> --message "继续" --quiet --format text
deepseek browser list
deepseek browser start --headless --browser-purpose primary
deepseek browser stop --browser-id <browserId>
```

## Legacy Chrome Profile Clone

只在明确要从普通 Chrome profile 复制登录态时使用：

```sh
deepseek plan --clone-chrome-profile
deepseek reply --message "测试" --clone-chrome-profile --headless --format text
```

legacy clone 是 DeepSeek 站点级最小复制 contract，不再存在额外 copy-scope 选项。

会保留：

- 整份 `Local State`
- 自动选中的 source profile 里的 DeepSeek cookie rows
- 自动选中的 source profile 里 `https://chat.deepseek.com/` origin 的 localStorage entries

会排除：

- 非 DeepSeek 的 cookie rows
- 非 DeepSeek 的 localStorage entries
- 整份 `Default/Preferences`
- 整份 `Default/Login Data`
- 整份 `Default/Web Data`
- 整份 `Default/IndexedDB`
- 整份 `Default/Service Worker`
- 整份 `Default/Sessions`

选择 source profile：

- 系统会用 DeepSeek cookie 证据选择唯一 profile。
- 如果多个 profile 都有 DeepSeek cookie，命令会 fail-closed。
- 只有这种消歧场景才传 `--chrome-profile-directory "Profile 4"`。
- `--chrome-user-data-dir` 只表示 source root pinning。

兼容说明：

- 旧的 one-shot 主链路没有接上统一 auto-management seam。
- 现在 legacy clone 仍留在 managed 家族：reuse / launch / auto-isolate / fail-closed。
- legacy clone 不会 auto attach 到用户现成浏览器。

## 思考、回复与文档边界

当用 DeepSeek 生成可交付文档时，必须隔离过程、回复和正文：

- 多部分输出时，分析过程放入 `<thinking>`，对话回复放入 `<answer>`，最终文档正文单独输出或写入目标文件。
- 进入最终文档正文后，删除寒暄、解释性套话、自我修正、“这是为您生成的文档”等对话性内容。
- 从分析转入正文时切换语域：分析可以是内部推演语气，正文必须正式、精炼、可交付。
- 深度推理任务先完成推理，再输出正文；不要边想边写最终文档。
- 外部材料或长文本只吸收事实并输出结论；正文不要写“根据您提供的文本”“我们可以看出”等过程描述。
- 严格 JSON 模式下，用独立 keys 区分 thinking、answer 和 final document，不要混写。
- 长度受限时优先保证最终文档完整，不要为保留过程内容截断正文。
- 只有用户明确要求展示推理过程时，才把可见推理写入最终文本；推翻、重写、自我修正不得泄露到正文。

## 常见问题

### 一上来就连不上浏览器

```sh
deepseek plan
deepseek auth login
```

如果用户明确要连接自己的 CDP 浏览器，再使用 `--browser-mode attach` 和 `--cdp-url`。

### 想继续旧会话，但不知道 `sessionId`

```sh
deepseek sync-session --headless
deepseek list-sessions
```

### 复用浏览器后状态仍不连贯

复用的是 runtime，不是页面临时状态。需要稳定继续时，用 `--session-id`。

## 给无状态 LLM 的默认规则

- 默认使用 `deepseek`。
- 第一次运行优先 `deepseek auth login`，然后 `deepseek plan`。
- 如果用户只想先跑通，优先 auth profile 默认路径，不要推荐 `--clone-chrome-profile`。
- 只有用户明确要从普通 Chrome source profile 复制登录态时，才使用 legacy `--clone-chrome-profile`。
- 如果用户给了 `--browser-id`、`--browser-mode` 或 `--cdp-url`，不要替用户 silent reroute。
- 投研网页搜索默认加
  `--chat-mode instant --search on --deep-think on --quiet --format text`。
- 继续旧会话但没有 `sessionId` 时，先执行 `deepseek sync-session --headless`，再执行 `deepseek list-sessions`。
- 真实继续会话优先使用 `--session-id`。
- 同一研究线优先复用已有 `sessionId`。
- 需要完整手册时，执行 `deepseek skillbook`。

---
> Source: [meomeo-dev/deepseek-cdp-cli](https://github.com/meomeo-dev/deepseek-cdp-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
