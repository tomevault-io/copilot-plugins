## agentero

> Agentero 是一个基于 Tauri 2 + React 19 的本地优先科研工作台。Vault 中：人的笔记与 source 以 Markdown/文件为准；论文集合与结构化 metadata 以 `.agentero/catalog.sqlite` 为准（可导出 `PAPERS.md` / BibTeX，非默认落盘）。离开应用后笔记与源文件仍可被外部工具读取。

# AGENTS.md

## 项目概览

Agentero 是一个基于 Tauri 2 + React 19 的本地优先科研工作台。Vault 中：人的笔记与 source 以 Markdown/文件为准；论文集合与结构化 metadata 以 `.agentero/catalog.sqlite` 为准（可导出 `PAPERS.md` / BibTeX，非默认落盘）。离开应用后笔记与源文件仍可被外部工具读取。

## 当前应用形态

- 前端：`src/`（React、TypeScript、Tailwind CSS 4、shadcn/ui、AI Elements）。
- Host：`src-tauri/`（Rust、Tauri commands、本地文件系统、Wiki 索引、ACP Client）。
- CLI：`cli/`（package `agentero-cli`，bin **`agentero`**）— headless Vault/Catalog；path 依赖 `agentero_lib`；**无 BYOA / 无 paper-reader**（见 `docs/development/cli.md`）。
- 工作台布局：
  - 左侧：Vault 文件树（顶部虚拟 **Library** + 其下 **Recycle Bin**、魔棒、新建文件/文件夹；右键 **Finder 显示 / 终端打开 / 删除**）+ 选中论文时 **Paper Info**；
  - 中间：无 Vault 时欢迎页；有 Vault 时默认 **全库论文表格**；单击非 paper 文件夹在**同一 Library tab** 就地按路径筛选（不新建 tab）；或标签页打开的 **PDF / HTML / 图片** / Markdown 笔记；关光文档 tab 后回到全库；
  - 右侧 Notes：**仅**打开具体论文且 PDF/HTML 时显示该篇 `NOTES.md`（WYSIWYG，无独立预览栏）；
  - 可选右侧栏：`Agent` 或 `Backlinks`（与左栏均为 **常驻 collapsible**，`preserve-pixel-size`）。
  - **全局错误 Toast**（右上角 Sonner）：操作失败经 `notifyError`（`src/lib/notify.ts`）弹出；表单就地校验除外。
  - **Agent 禅模式**（`⌥⌘Z` / 标题栏 Layout「面板」菜单）：仅全屏 Agent 对话，复用 AI Elements `AgentPanel`（`variant="zen"`），不 remount 丢会话；左侧栏 Quest 式弱对比（新建 + 单行历史）；主区顶栏仅 Agent 切换（无 1/2/3 标签）；对话区全宽滚动 + AI Elements；标题栏返回图标退出。精读 / PDF 划词等后台运行不进对话历史。
  - **文档标签页**（浏览器式多 tab）位于**标题栏**（与 Layout / 侧栏图标同行）：可同时打开多个 paper / PDF / HTML / Markdown / Library（全库或文件夹作用域），切换、关闭、拖拽重排；每个 tab **常驻挂载**，切换保留 PDF 滚动/缩放与编辑器状态。快捷键：`⌘W`（仅剩全库时关窗，否则关 tab；关空后自动全库）、切换 `⌥⌘←/→`。**分屏（split）** 仍规划见 roadmap V0.6。
- 论文库：`paper_list` 读 catalog 一次进内存；表头排序；横向/纵向滚动；**tags** 列 + chip 筛选；**文件夹作用域**按 `paper.path` 前缀过滤（不扫盘、无 per-folder RPC）。虚拟路径 `agentero:library` 不写盘。
- 标签：Paper Info 增删与 **Apple 风格 8 色** → Host `paper_set_tags`（catalog `tags_json` 权威；元素为字符串或 `{name,color}`）；Library 染色 chip 与筛选；CLI `paper set-tags` / `list --tag` / `tags`（CLI 仅名称）。
- 魔棒入库：默认下载 PDF 到 **论文文件夹根目录** `{paper}/{id}.pdf`；arXiv 另解压 e-print LaTeX 到 `source/`。入库成功后刷新树并 `openPaper`，左侧文件树**展开并滚到新论文**。paper 行缺 PDF，或既无 TeX 也无 `PAPER.md` 时显示 Download（hover 说明原因）；Library 行可批量补下。
- **Zotero Connector 兼容**（MVP）：设置 → 通用开关（默认关）；Host 在 `127.0.0.1:23119` 收官方浏览器扩展 `saveItems` + **`saveAttachment`**（浏览器上传登录墙 PDF）→ 当前 Vault；组织子文件夹可选；与 Zotero 桌面端口互斥。见 `docs/backend/connector.md`。
- **可读正文**：TeX 与 `PAPER.md` 有其一即可（优先 TeX）。无 TeX 时下载后 liteparse 生成 `PAPER.md`；有 TeX 不强制 `PAPER.md`。
- **精读工作流**：设置 → Agent **`autoPaperReader`**（**默认关**）；开启后魔棒入库 / 单篇 Download 资源就绪且未读时自动 paper-reader。资源齐全且 `is_read === false` 时文件树 **Zap** 可手动。写入 `NOTES.md`，成功后 `is_read = true`。进度在左下角后台任务条（**hover 实色不透明**）。Skill 运行时语法按 Agent：**Claude `/id`**、其它（含 Codex）仅注入 `SKILL.md`。
- **Agent 权限**：设置 → Agent **全局权限模式**（`restricted` 默认 / **`ask` 每次询问** / `auto` 自动批准）；非 per-provider YOLO。`ask` 时弹权限对话框 → `agent_respond_permission`。
- **个人偏好提示词**：设置 → Agent **`agentPersonalPrompt`**（多行，默认空）；非空时经 `runOnce` → Host `build_prompt` 注入 envelope（所有 workflow）；留空不注入，Chat 不展示该块。
- **Agent 面板工作流**：空态建议 chips → `summary` / `qa` / `related_work`（Summarize、Ask library、Draft Related Work 等）；目标为当前聚焦 paper。Composer：**当前论文默认加入**上下文（实心 chip + paper-name/标题，无加号切换；可 X 移除）；`@` 提及（论文文件夹 + 目录 + paper 外 Markdown；空 `@` 优先最近路径与浅层目录树；行右 **›** 进入子目录、‹/`←`/`Esc` 返回；论文标签与文件树 `paperTreeLabelMode` 一致）与从文件树**拖入**路径均为可移除 context chip（路径引用，非 AI Elements Attachments 二进制）；chip 展示虚拟名（paper-name），prompt 仍用 Vault 相对路径；图标见 `context-path-icon`（**paper** → `ScrollText`，其它文件夹 → `Folder`，文件按扩展名）。
- **笔记写后审阅**：Agent 改写目标笔记后 `agent:notes-review` **统一 Diff**，Keep / Revert。
- **命令面板**（`⌘K` / `⌘P`）：论文 quick-open + Vault Markdown 全文搜索（`vault_search`）。
- 文件树：右键 / `⌥⌘R` 在 Finder 中显示（无双击）；右键 / `⌥⌘T` 在终端中打开（文件夹 = 自身 cwd，文件 = 父目录；系统默认终端）；**`⌘←` 折叠选中文件夹**、**`⇧⌘←` 折叠至默认**（只展开 `papers/`，不展开其子目录）；多选（⌘/Shift）+ 拖拽移动；**删除**走回收站（`path_trash` → `.agentero/.trash/`，**无确认 / 无 Undo toast**）；Library 下方虚拟节点 **Recycle Bin**（`agentero:trash`）打开中间栏回收站视图（恢复 / 永久删除 / 清空）。打开 Vault 时默认只展开 `papers/` 及其一级子目录；激活文档变化时树展开祖先并滚到对应行。Paper 行标签默认 **标题 · 作者**（设置 → 通用 `paperTreeLabelMode`，展示用、不改磁盘名）；同目录排序默认 **显示名称 A–Z**（与 `paperTreeLabelMode` 标签一致；`paperTreeSortMode`：标题 / 作者 / 年份 / 添加时间等预设，展示用）。
- 论文库：**Rescan**（`paper_rescan`）从 `papers/` + `metadata.json` 补齐盘上有、catalog 无的条目。
- PDF：Vault **任意路径** `.pdf` → `blob:` 预览；论文单元本地优先 → 自动下载 → 远程回退；**页码导航 / 适应宽·整页 / 大纲 / ⌘F 查找**；真实 scale 渲染 + 平滑划词覆盖层；划词操作菜单（高亮 / 批注 / 提问 / 翻译，见 `docs/development/pdf-ask.md`）。**批注** = 高亮 + 内联评论（`comment`），带 comment 的高亮显示页边批注针，右侧 **批注** 面板列出当前 PDF 的批注卡（跳转 / 编辑 / 删除），不写入 `NOTES.md`。
- **翻译服务**：应用级可插拔 `TranslateService`（免费 MT + BYOA Agent，无付费 API），设置 → 翻译；PDF 划词等为消费方；见 `docs/development/translate.md`。
- 图片：常见格式任意路径 `blob:` 中间栏预览。
- **Markdown 内嵌图片**：粘贴 / 工具栏 → `{mdDir}/assets/` + `![](./assets/…)`；选中显示源码；删节点且无引用时 GC（`src/lib/markdown-image.ts`）。
- **外部/Agent 改动自动重载**：Host `notify` → `vault:file-changed`（`watcher.rs` / `fs-watch.ts`）；打开中的 `.md`/`NOTES.md` 磁盘变化：**无未存改动则重载**；**有未存改动则 toast 提示**（不静默覆盖）；内容相等抑制自写回声；create/remove/rename 去抖刷新文件树。
- **Wiki 索引**：`.md` 变更防抖重建（`scheduleWikiRebuild`，~900ms），Backlinks/Graph 保持新鲜。
- **保存冲突**：写盘前比对上次落盘内容；磁盘已被外部改则中止写入并警告（`diskConflict.saveBlocked`）。
- **运行日志**：Host `tauri-plugin-log` + 前端 `logger` + CLI `env_logger`；关键操作 op start/end（见 `docs/development/logging.md`）。
- 路线图与 backlog：`docs/development/roadmap.md`、`docs/development/todo.md`（改能力时同步勾选）。规划中：**V0.6 分屏（split，标签页已落地）**、**V0.7 引用关系（hover Info / Connected Papers / Agent 引用工作流）**。
- 多窗口：`⌘N` → Host `window_new`；当前 Vault 按窗口 session 隔离，最近列表在 localStorage。
- Backlinks 右侧栏布局：上方 Backlinks，下方 Graph；Graph 不是独立顶层 tab；Graph 为 **双链图**（非文献引用图）。
- Graph 数据必须来自 Markdown 双链或可重建索引，不能来自手工维护的图数据库。

## 开发规则

- 优先做小而聚焦的改动，避免无关重构。
- 保持 local-first：不要引入私有存储作为事实来源。
- 未经明确确认，不要覆盖用户手写的 Vault 文件。
- 编辑或生成 Markdown 时保留 Obsidian 兼容的双链文本（`[[...]]`）。
- Agent 集成采用 BYOA：Agentero 只配置如何启动本机 ACP-compatible Agent，不要求用户在 Agentero 内填写模型 API Key。
- UI 保持简约：图标按钮必须有可访问名称和 Tooltip；除非是必要的空状态/错误说明，否则避免常驻解释文案。操作失败用右上角 `notifyError` Toast，不要在侧栏 header 挂常驻错误条。
- 国际化（i18n）：所有面向用户的文案都必须经 `t()` 走 `react-i18next`，禁止硬编码字符串。English（`en`）为源语言，新增文案先登记 `en` 词条再同步 `zh-CN`（`src/i18n/locales/`）。跨命名空间用 `t("ns:key")` 并在 `useTranslation([...])` 声明；React 之外用全局 `i18n.t()`。数字/日期用 `i18n.language` 格式化。详见 `docs/frontend/ui.md` §4.1。
- 修改后需要同步更新相关文档。如果修改了 UI、数据契约、发布流程或 Vault 语义，必须同步更新相关文档。并检查 Roadmap 和 Todo。

## 常用命令

```bash
pnpm install
pnpm dev
pnpm tauri dev
pnpm build
pnpm lint
pnpm format
pnpm tauri build

# Headless CLI（仓库根 workspace）
cargo build -p agentero-cli
cargo run -p agentero-cli -- vault which --json
cargo test -p agentero-cli
```

完成实现前运行最小必要验证。UI 改动优先启动应用并检查对应流程；如果 dev 端口被占用或无法做浏览器级验证，需要明确说明。

## 文档地图

- `README.md`：项目简介、快速开始、发布说明、文档入口。
- `docs/index.md`：整体技术框架与文档分层。
- `docs/frontend/index.md`：前端技术选型和 UI 文档入口。
- `docs/frontend/ui.md`：UI 布局、组件、快捷键和设置规范。
- `docs/frontend/components.md`：AI Elements 与组件约定。
- `docs/backend/index.md`：后端技术选型、API 和数据模型入口。
- `docs/backend/api.md`：Tauri command 与 event 契约。
- `docs/backend/wikilinks.md`：双链、反链与图谱设计。
- `docs/backend/data-model.md`：Vault 文件模型。
- `docs/backend/catalog.md`：论文目录库（`.agentero/catalog.sqlite`）与导出。
- `docs/backend/paper-import-pipeline.md`：多入口入库现状与统一 `paper_commit` / `afterPaperImport` 设计。
- `docs/development/index.md`：产品、路线图、开发和发布流程入口。
- `docs/development/roadmap.md`：实现状态与路线图。
- `docs/development/todo.md`：可执行 backlog。
- `docs/development/technical-plan.md`：跨前后端技术方案。
- `docs/development/prd.md`：产品需求和验收标准。
- `docs/development/cli.md`：headless CLI 语义与实现（`cli/`）。
- `docs/development/pdf-ask.md`：PDF 划词提问与批注。
- `docs/development/translate.md`：翻译服务（应用级可插拔 free + Agent；设置 → 翻译）。

当修改 UI、数据契约、发布流程或 Vault 语义时，必须同步更新相关文档。

## 文档站与发布

- 文档站使用 [MkDocs](https://www.mkdocs.org/) 与 Read the Docs 主题。
- 本地预览：`python3 -m venv .venv-docs && . .venv-docs/bin/activate && pip install mkdocs==1.6.1 && mkdocs serve`。
- `.github/workflows/docs.yml` 在文档相关文件变更后构建文档并部署到 `gh-pages` 分支。

## Commit

- 提交信息必须符合 [Conventional Commits](https://www.conventionalcommits.org/) 规范。
- 一次提交只做一件事，避免混合多个 unrelated changes。

## 应用发布流程

推送 `v*` tag 会触发 `.github/workflows/release.yml`（Release）：先 **prepare** 草稿 Release，再并行 **installers**（Tauri）与 **cli**（`agentero` 预编译包），上传到同一草稿。

不要在未补充文档和 secrets 说明的情况下加入签名、公证或自动发布步骤；本地开发构建不能依赖发布凭据。

---
> Source: [poco-ai/Agentero](https://github.com/poco-ai/Agentero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
