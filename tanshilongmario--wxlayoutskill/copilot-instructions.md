## wxlayoutskill

> > 本文件遵循 [AGENTS.md 开放标准](https://agents.md/)，被大多数 AI 编码 Agent 自动读取（Cursor、Codex、Claude Code、GitHub Copilot、Windsurf 等）。

# AGENTS.md

> 本文件遵循 [AGENTS.md 开放标准](https://agents.md/)，被大多数 AI 编码 Agent 自动读取（Cursor、Codex、Claude Code、GitHub Copilot、Windsurf 等）。
> 用于在不支持 Skill 机制的 Agent（如 Copilot、Windsurf）里提供核心排版权威规则。

## 项目性质

本仓库是 **WeChat Layout Skills** 的发布包，包含三个 Cursor/Codex/Claude Code 通用的 Agent Skill + 一个交互式选择页：

- `skills/wxlayout-wechat-paste` — 公众号排版引擎（模板 + 自主编排）
- `skills/wechat-safe-colors` — 微信安全色配色层
- `skills/wxlayout-template-picker` — 模板配色选择器（帮用户选模板/配色，输出选择摘要）
- `template-color-picker.html` — 浏览器交互式模板配色索引页（仓库根目录，直接打开）

依赖关系：`wxlayout-template-picker` → `wxlayout-wechat-paste` → `wechat-safe-colors`。三个 Skill 必须成套使用。

## 排版权威规则（适用于无 Skill 机制的 Agent）

若你的 Agent 不支持 Skill 自动加载（如 GitHub Copilot、Windsurf），将以下规则视为项目不变量：

### 内容 ≠ 排版

- Markdown 只管内容（文字、结构）
- 排版由独立决策承担：模板选型、配色绑定、对齐、强调
- 不要为了排版去扩展 MD 语法

### 自主编排（用户只说「排版这篇 MD」时必执行）

1. 扫描 MD：字数、H2/H3 数量、table、引用、并列结构、阶段词
2. 判定文章类型：个人观点 / 方法框架 / 产品技术 / 故事案例 / 路线图 / 元宣言 / 长文统一
3. 定模板种类数：短<1500 字 2–3 种；中 1500–3500 字 3–4 种；长>3500 字 4–5 种
4. 映射语义→模板（详见 `skills/wxlayout-wechat-paste/orchestration.md`）
5. 定配色主题，全篇色族 ≤4
6. 自检：深底卡≤2 且不连续、不海报化、T02 仅一张
7. 渲染前用 3–5 行「编排方案」告知用户，然后直接渲染，不反问

### 微信安全色七原则（P1–P7）

1. 低饱和、中明度实色作大面积底（禁止 Tailwind 500+ 高饱和色作卡片底）
2. 浅底→字色方案 N（莫兰迪浅→B）；深底→W（莫兰迪深→C）
3. 边框 = 同族加深（scale.400）
4. 渐变保留 + `background-color`/`color` fallback 必留
5. 不绕过 mp-darkmode（白底 + `color-scheme: light` 对公众号无效）
6. 层次靠底色区分，避免同色连成一页
7. 字色与底色脱钩：浅底统一 `#383838` 正文，深底统一 `#F4ECD7` 正文

### 渲染约束

- 全部 inline style，禁止外链 CSS、禁止 class 依赖
- 标签白名单：`section, p, span, strong, em, br, a`
- 圆角全篇统一 12px（r12），禁止 4/6/8/10/18px 混用
- 禁止 `#FFFFFF` 纯白底/白边、`box-shadow` with rgba、双图并排 inline-block
- 内容区 max-width 680px
- `font-family` 用系统栈：`-apple-system, BlinkMacSystemFont, 'PingFang SC', 'Helvetica Neue', 'Microsoft YaHei', sans-serif`

### 反模式（禁止）

- 每个 H2 换一种模板（海报化）
- 短于 1500 字用 5+ 模板种类
- 连续 3 张深底卡
- T02 与 T04 同篇且相邻
- 全篇每节不同色族
- 高饱和蓝/琥珀/薄荷作大面积底

## 数据文件（不要修改内容，只可引用）

| 文件 | 用途 |
|------|------|
| `skills/wechat-safe-colors/data/wechat-safe-palette.json` | 9 族安全色 + scale + 字色 + 高亮 H1/H2 |
| `skills/wxlayout-wechat-paste/data/wechat-layout-templates.json` | 12 张模板结构 + defaultBindings |
| `skills/wxlayout-wechat-paste/data/wechat-layout-orchestration.json` | 自主编排决策规则 |
| `skills/wxlayout-wechat-paste/data/wechat-layout-tokens.json` | 圆角 r12 / 间距 / 字号规范 |

## 各 Agent 安装位置

| Agent | 目录 |
|-------|------|
| Cursor | `~/.cursor/skills/` 或 `<project>/.cursor/skills/` |
| Codex CLI | `$HOME/.agents/skills/` 或 `<project>/.agents/skills/` |
| Claude Code | `~/.claude/skills/` 或 `<project>/.claude/skills/` |
| GitHub Copilot | 无 Skill 机制，靠本 AGENTS.md + `.github/copilot-instructions.md` |
| Windsurf | 无 Skill 机制，靠本 AGENTS.md + `.windsurf/rules/*.md` |

一键安装：`./install.sh --target all`

## 验证要求

新增模板/色族/文章类型 profile 需附**微信公众号编辑器浅色 + 深色双模式粘贴实测截图**，与现有 `validated` 资产同等验证等级。

---
> Source: [TanShilongMario/WXLayoutSkill](https://github.com/TanShilongMario/WXLayoutSkill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
