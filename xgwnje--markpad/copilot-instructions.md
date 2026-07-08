## markpad

> 本文档为 Codex 提供本仓库的代码协作指引。README 面向用户；本文件面向 Agent，负责项目边界、文件职责、验证步骤和历史包袱处理。

# AGENTS.md

本文档为 Codex 提供本仓库的代码协作指引。README 面向用户；本文件面向 Agent，负责项目边界、文件职责、验证步骤和历史包袱处理。

## 项目边界

- 产品名已确定为 **MarkPad**。用户可见位置必须使用 MarkPad，包括 `manifest.json`、`index.html` title、README、图标导出页和发布说明。
- 仓库目录仍为 `D:\ObjectCode\BookmarkTab`，内部类名如 `BookmarkStore`、`BookmarkCard` 表示书签领域对象，不作为产品品牌。不要为了品牌统一批量重命名文件、类、事件或存储 key，除非用户明确要求迁移。
- 当前视觉规范优先参考 `D:\ObjectCode\visual-rules-collection\rules\lumen-index-ui-system.md`：温白/深灰背景、黑白骨架、轻边框、低阴影、克制密度。
- 品牌方向见 `docs/markpad-brand-visual-guide.md`；触控与图标工坊计划见 `docs/touch-icon-mvp-plan.md`。
- `CLAUDE.md` 只是旧工具兼容入口，不再维护独立规则内容。

## 文档分工

- `README.md`：用户入口，只保留项目定位、功能、安装、快捷键、结构摘要、验证和权限。
- `AGENTS.md`：Agent 入口，维护架构、边界、验证、危险操作和历史负担说明。
- `docs/`：专题设计和计划。计划完成后要更新状态，避免 README 和 AGENTS 继续引用过时承诺。
- 不要在 README、AGENTS、docs 之间复制大段同义内容；发现重复时按读者职责归位。

## Verification

项目不维护独立自动评测脚本。代码修改后优先运行与改动范围匹配的轻量检查：

- JavaScript 语法检查：`node --check <file>`
- Chrome 扩展清单检查：确认 `manifest.json` JSON 合法且权限与实际功能匹配
- CSS 结构检查：至少确认改动模块括号匹配
- 文档变更：确认 README、AGENTS 和 `docs/` 中没有过时路径、旧产品名或未实现承诺
- 编码检查：中文 Markdown/JSON/JS/CSS 修改后确认 UTF-8 无 BOM，且没有疑似 mojibake 或替换字符
- Git 空白检查：`git diff --check`

涉及触控交互、弹窗、拖拽、图标工坊、图标库或 Chrome API 行为时，需要在 Chrome 扩展页面刷新后做运行态验证。Playwright 可用于临时 profile 的 MV3 smoke test；不要污染用户真实 Chrome 书签数据。

## 架构

```
BookmarkTab/
├── components/     # UI 组件
├── core/           # 数据层、事件、路由、图标库和外部图标源
├── css/            # main.css 入口 + modules/ 模块
├── docs/           # 品牌、触控和图标工坊计划
├── icons/          # 扩展图标 + export.html
├── wallpapers/     # 历史壁纸资源；当前设置使用 SettingsPanel 内置预设
├── index.html      # 新标签页入口
├── main.js         # 应用装配与全局交互
└── manifest.json   # Chrome 扩展清单 V3
```

### 核心层（./core/）

- **BookmarkStore.js** — 数据层，封装 `chrome.bookmarks` API。负责增删改查、favicon 缓存、自定义图标存储、书签树查询、文件夹子项数量统计。图标缓存优先写入 `chrome.storage.local`，并兼容旧 `localStorage` 数据。
- **IconLibrary.js** — 本地应用图标库。应用自身图标统一由这里输出线性 SVG；真实网站 favicon、用户上传图标和图标工坊候选 SVG 不经过这里替换。
- **IconSourceProvider.js** — 外部 SVG 图标源适配层。混排 iconfont、Iconify 和 SVG API；iconfont API 不返回可用 SVG 时，可后台打开/复用与当前关键词匹配的 iconfont 搜索页并抽取 SVG。不得读取不匹配关键词的旧 iconfont 页面。
- **Router.js** — 导航层，管理文件夹层级与浏览器历史集成。
- **EventBus.js** — 发布/订阅事件系统，解耦组件。

### 组件层（./components/）

- **BookmarkGrid.js** — 网格容器，渲染当前文件夹的书签卡片。负责 favicon 分批懒加载、卡片多选、拖拽排序、删除执行和刷新协调。
- **BookmarkCard.js** — 单张书签/文件夹卡片。支持拖拽、右键菜单、行内标题编辑、自定义图标、Toast 提示。
- **Breadcrumb.js** — 头部面包屑导航。
- **Toolbar.js** — 常驻顶部工具栏，发射新建书签、新建文件夹和搜索事件。
- **EditDialog.js** — 新建/编辑书签或文件夹弹窗。
- **MoveDialog.js** — 右键菜单“移动到...”目标文件夹弹窗。
- **QuickFind.js** — 全局模糊搜索浮层（`/` 或 `Ctrl+F`）。
- **IconStudio.js** — 图标工坊弹窗。只保留 SVG 搜索、预览和直接应用；不接入模型 API 或生图功能。
- **SettingsPanel.js** — 头部工具面板中的壁纸偏好模块，保留浅色、暗色和自定义图片。

## CSS 维护规则

- 先改 `variables.css` 中的 Lumen 风格 token，再映射到组件模块；避免在组件里散落 magic number。
- 视觉统一只改既有界面的颜色、间距、圆角、阴影、状态和密度；不要新增品牌块、底部栏、说明卡片、装饰图形或额外入口，除非用户明确要求。
- 书签卡片有两种状态：显示文字和隐藏文字。显示文字时域名/计数一行、名称一行，均不换行；隐藏文字时卡片保持正方形。
- 应用自身图标必须使用 `core/IconLibrary.js`。不要重新引入 emoji、字符图标或散落的内联 SVG。
- 真实 favicon、自定义图标和图标工坊候选 SVG 是用户内容，不强制替换为 `IconLibrary` 图标。
- 触控目标保持不小于 44px；弹窗和图标工坊必须保留粗指针友好布局。

关键模块：
- `variables.css` — Lumen Index 风格设计令牌。
- `toolbar.css` / `breadcrumb.css` / `shortcuts.css` — 头部工具、面包屑和工具菜单。
- `card.css` / `grid.css` — 卡片与网格。
- `dialog.css` / `quick-find.css` / `icon-studio.css` — 弹窗、搜索、图标工坊。
- `drag-zones.css` — 左侧移动面板和右侧删除区域。
- `settings.css` / `wallpapers.css` — 壁纸设置与背景层。

## 关键模式

**事件驱动通信**：组件间通过 EventBus 通信，不直接互调业务动作。`Toolbar` 发射 `toolbar:newBookmark`，`EditDialog` 处理；`BookmarkGrid` 监听 `navigate`。

**删除确认**：删除统一通过 `card:requestDelete` 进入 `main.js` 的确认弹窗，再由确认按钮发射 `card:delete` 执行删除。文件夹删除必须显示包含子项数量。

**图标安全**：自定义 SVG 在存储前必须通过 `DOMParser` 清理。移除可执行/嵌入/外链相关元素，过滤 `on*`、`javascript:`、`data:`、外部 `xlink:href` 和 `style url(...)` 等风险。

**图标工坊**：当前只做 SVG 搜索、候选来源标注、预览和直接应用。不新增 API key、模型选择、生图、高清生成或自动批量生成。

**本地偏好**：卡片尺寸、打开方式、卡片文字显隐、壁纸偏好保存在 `localStorage`。不要无迁移方案地改 key。

## 工程卫生

- 无构建步骤，无 npm 运行依赖。纯 ES Modules 直接由扩展加载。
- `.gitignore` 忽略本地 Agent/工具状态目录（如 `.codex/`、`.agents/`、`.openharness/`）和日志，不要提交本机运行态。
- `.gitattributes` 固定文本 LF，并把图片资源标记为 binary。
- 不主动提交；需要提交时先检查 `git status`，不要混入无关文件。

---
> Source: [XGWNJE/MarkPad](https://github.com/XGWNJE/MarkPad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
