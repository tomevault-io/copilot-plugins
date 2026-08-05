## xharness

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 常用命令

```bash
npm run build                 # tsc 编译 src/ → dist/（GUI 依赖 dist，改核心后必须重建）
npx tsc --noEmit              # 类型检查
npm test                      # 单测（vitest unit project，API 全 mock，不耗 token）
npx vitest run --project unit test/unit/loop.test.ts   # 跑单个测试文件
npm run test:e2e              # E2E（先 build；需 ANTHROPIC_API_KEY/DEEPSEEK_API_KEY，无 key 整体 skip）
node dist/index.js -p "..."   # CLI 一次性模式（冒烟最快路径）

cd gui && npm start                        # 启动 Electron GUI（dev）
cd gui && node scripts/package-app.mjs     # 打包 release/xharness.app + mac-<arch>.zip
```

杀 GUI 进程用 `pkill -f "MacOS/xharness"`（Electron bundle 已被 postinstall 改名为
xharness.app，匹配 "Electron" 会失手）。GUI 调试/测试走 CDP
（`--remote-debugging-port=9223` + 原生 WebSocket 直连，`gui/scripts/cdp-eval.mjs`
驱动），完整方案与标准冒烟检查见 `docs/cdp-testing.md`。

## 仓库形态

两个交付物，一个引擎：

- **CLI**（`src/` → `dist/`，Node >= 22，ESM）：终端 REPL / `-p` 一次性模式。
- **GUI**（`gui/`，Electron）：**不复制引擎代码**，主进程直接 `import "../dist/..."`。
  改 `src/` 后不 `npm run build`，GUI 看不到变化。

`GOAL.md` 是产品规格（含各条设计决策与变更日期）；`docs/internal/` 是开发过程记录
（GoalBuddy 任务板、评审报告），改代码前有疑义先查 GOAL.md 对应条目。

## 核心架构与硬约束

### 分层铁律（违反即错，评审按此核对）

- `src/api/` 是**唯一**接触供应商 SDK 与原始流事件的层，对外只发归一化领域事件
  （`text_delta | thinking_delta | tool_start | tool_end | error | turn_end`）：
  `client.ts` = 接口 + 共享重试/流聚合 + 按 `config.apiFormat` 分发；
  `anthropic.ts` = Messages 格式（唯一碰 `@anthropic-ai/sdk`）；
  `responses.ts` = OpenAI Response 格式（零依赖 fetch+SSE，翻译成 RawStreamEvent 复用聚合）。
- `src/agent/loop.ts` 只做回合编排：不碰 SDK、不读 `process.env`、不解析原始流、不内联压缩。
- `src/ui/render.ts` 只消费领域事件。
- `src/config.ts` 的 `loadConfig()` 是全项目**唯一** `process.env` 读取点。
- 工具（`src/tools/`）只做副作用与返回，不感知会话状态；异常一律转 `isError:true` 的
  ToolResult，不外抛。registry 禁止按工具名写特殊分支。

### 不变量：tool_use / tool_result 配对

history 中每个 `tool_use` 必须有配对 `tool_result`，否则官方 Anthropic 端点直接 400。
所有取舍（中断、上限 200 触顶、AskUserQuestion 被 SIGINT/EOF 打断）都用 `is_error`
占位块回填来保住配对——修改 loop/中断路径时此约束优先于其他一切语义。
单测里有 `expectAllToolUsesPaired` 辅助断言，改动相关逻辑必须覆盖。

### 工具执行模型

同一响应内多个 `tool_use` **并行执行**（Promise.all），`tool_result` 按原顺序落位；
单个失败不影响其余；批内已启动的工具在中断时各自响应 AbortSignal 收尾（Bash 对进程组发
SIGTERM）。上限 200 = 单回合已执行 tool_use 总数（非 round-trip 数）。

### 端点与模型

线格式按供应商 `apiFormat` 二选一（`anthropic` / `openai-responses`），内部领域模型
保持 Anthropic 形状不变。默认端点是 DeepSeek `https://api.deepseek.com/anthropic`，
默认模型 `deepseek-v4-pro`。thinking 档位（none/low/high/max）在 Anthropic 格式下：
`none` 发 `thinking:{type:"disabled"}`（实测唯一可靠关闭方式），`low/high/max` 透传
`reasoning:{effort}`（DeepSeek 扩展字段，anthropic.ts 中唯一的 `as` 断言处）；
Response 格式（grok/xAI，`https://api.x.ai/v1/responses`）只有 low/high 两档：
none/未设省略 reasoning，max 归并 high，模型不支持 reasoning 参数（400）时自动去
reasoning 重试一次。思考内容只渲染、**不入 history**。`budget_tokens` 不实现
（DeepSeek 忽略）。Response API 的 usage 须把 cached_tokens 从 input_tokens 里拆出
（对齐「input 不含缓存」计费语义）。CLI 环境变量路径恒 anthropic。

### compact（src/agent/compact.ts）

自动（>80% 窗口）与手动 `/compact` 共用 `doCompact`；保留最近 10 条原始消息，切点若落在
tool_use/tool_result 配对中间则**向旧侧扩窗**（宁多保留不拆对）；摘要以 `[历史摘要]`
前缀 user 消息注入；失败保留原历史不重试。自动触发在调用方层（CLI index / GUI engine），
不在 loop 内。

### 技能与斜杠命令

`~/.agents/skills/<name>/SKILL.md`（全局，跨 harness 通用目录）与项目
`.agents/skills/`（覆盖全局同名）；frontmatter 兼容 Claude Code（name/description）。
内置命令（/help /clear /compact /effort /exit）优先级高于同名技能。技能双触发：
用户 `/<name>`、模型经 `Skill` 元工具。

### 插件系统（src/plugins/，规格见 GOAL.md §4.6）

`~/.agents/plugins/<name>/plugin.json`（项目 `.agents/plugins` 覆盖同名）声明
preToolUse hooks（matcher 正则 + `/bin/sh -c` command，环境变量 `${PLUGIN_ROOT}`
`${NODE}`）；协议兼容 codex/Claude Code（stdin JSON，stdout
`hookSpecificOutput.permissionDecision:"deny"` 即拦截）。deny/超时/非零退出/spawn
失败都拒绝（fail-closed）。分层：loop.ts 只有通用 `preToolUse` 回调挂载点，deny 转
`is_error` tool_result 保配对；插件装载与组装在调用方（CLI createSession、GUI
engine 每回合重载，设置改动即时生效）。内置 `plugins/agentguard`（移植自
codex-agentguard）首启种子安装到全局目录，`.seeded.json` 标记，用户删除不复装；
hook 脚本必须零依赖 Node（打包版靠 `ELECTRON_RUN_AS_NODE=1` 跑 `${NODE}`，不得
假设系统有 python）。GUI 设置 → 插件只管理全局目录，IPC 回传的 root 必须过
`assertInGlobalDir`。

## GUI 要点（gui/）

- `engine.js` 按会话（convId）维护 History/Registry/供应商选择；**每回合开始
  `process.chdir(projectDir)`**——工具相对路径跟会话项目走，不跟 Electron 启动目录。
- 持久化全部为 **append-only JSONL**，数据目录 `~/Library/Application Support/xharness/`
  （sessions/、projects.jsonl、settings.jsonl、attachments/）。settings.jsonl 权限 600，
  手填 API Key **明文**存储（有意决策：ad-hoc 签名下 safeStorage/钥匙串每次启动弹框，
  已弃用；不要改回钥匙串）。GUI **仅手动填写** API Key，不读环境变量、无 env/manual
  双模式。IPC：providers 列表仍脱敏（key 置空 + `hasKey`）；设置详情经
  `settings:getProviderKey` 按 id 取回明文并回填输入框；保存以表单提交值为准。
- 安全基线（开源审查后确立，勿回退）：渲染层 markdown 一律过 DOMPurify；CSP 收紧；
  附件走 `xatt://` 受控协议（只按文件名从 attachments 目录取），禁止裸 `file://`；
  projectDir 类 IPC 校验必须为已添加项目。
- Mermaid 图渲染（规格 GOAL.md §4.10）：main.js 的 mermaidExt 把 ```` ```mermaid ````
  围栏转 `<pre class="mermaid">` 占位块（仅定稿实例；须在 markedHighlight 之后注册），
  renderer/mermaid.js 懒加载 `vendor/mermaid/` ESM 构建绘 SVG——`securityLevel:"strict"`
  + 全局 `htmlLabels:false`（否则标签在 foreignObject 里被 DOMPurify svg profile 剥掉），
  产出再过 DOMPurify（svg profile + `ADD_TAGS:["style"]`）；`data-theme` 变化按
  `data-mermaid-src` 重渲。
- d2 图形（GOAL.md §4.11）：` ```d2 `/` ```d2lang ` 代码块在段落定稿时经
  `d2:render` IPC 由主进程 `@terrastruct/d2`（wasm，懒加载单例）编译为 SVG，
  渲染层 DOMPurify 后替换 `<pre>`；流式中途不编译，失败保留代码块附报错；
  编译必须在主进程（渲染层 CSP `connect-src 'none'` 禁 fetch wasm）。
- 外观主题（renderer/theme.js + appearance.js）：一套主题只存强调/背景/前景三基色 +
  对比度，其余界面色全部由 `mixColor(bg, fg, t)` 推导——**style.css 禁止再写死颜色**，
  一律走 `:root` CSS 变量；深浅切换经 `data-theme` 与 hljs 双 link 的 disabled 互斥；
  半透明侧栏走窗口 vibrancy（设置页打开时下层 `#app` 必须 `visibility:hidden`，
  否则透明侧栏会透出下层内容重影）；外观持久化为 settings.jsonl 的 `appearance` 事件，
  rewriteSettings 整文件重写时必须一并带上，否则换 key 会把外观清掉。
- Electron 两个易踩的坑：`-webkit-app-region: drag` 区域**不受 z-index 遮挡影响**，
  会吃掉浮层点击（浮层下方不得有 drag 区）；"点击外部关闭菜单"要用 `mousedown` 判定
  （click 阶段若菜单内容已被重建，`contains()` 会误判为外部点击）。
- 打包脚本复制 .app 必须用 `ditto`（cpSync 破坏框架符号链接导致 codesign 失败）；
  打包版内置 ripgrep 于 Resources/bin 并置于 PATH 首位（Finder 启动不继承 shell PATH）。
- 打包脚本里的清单类配置**禁止手写枚举**，一律整体继承来源（如 staging 依赖用
  `{ ...rootPkg.dependencies, ...guiPkg.dependencies }`）——曾因硬编码两个包名，
  后续新增的 marked-highlight/highlight.js/morphdom 漏进包，安装版启动即崩（v0.0.3 热修）。
  新增 GUI 运行时依赖后必须重打包并实测 `open release/xharness.app` 能启动。

## 项目约定

- **YOLO 是产品定位**：无确认、无沙箱、无命令黑名单，不要"顺手"加运行时过滤；
  披露靠 README/SECURITY.md。
- **早期零迁移**：数据结构/路径变更一律当全新项目处理，不写迁移与兼容代码。
- E2E 约定（test/e2e/）：DeepSeek + `deepseek-v4-flash`、mkdtemp 沙箱、提示词禁破坏性
  命令且有 `assertNoDestructiveCommands` 审计、断言产物/退出码优先 + 工具子序列
  （禁全序列指纹）、retry 1 次、无 key skip。
- **GUI 验证闭环（强制）**：每个 issue/feature 完成后必须重启开发环境——
  `npm run build` 后按 `docs/cdp-testing.md` 重启 GUI 并做详细测试（标准冒烟
  三段 + 针对本次改动追加的断言），全部通过才算完成。单实例用固定端口 9223；
  **多 worktree 并发必须用文档 §4 的隔离流程**（`XH_DATA_DIR` + 端口 0 +
  `--data-dir`），禁止共享 9223、禁止裸 `pkill -f "MacOS/xharness"`。
- 依赖最小化：运行时仅 `@anthropic-ai/sdk` + `gray-matter`（GUI 另有 marked/dompurify/
  mermaid、@terrastruct/d2 等）；ripgrep 为硬依赖无 JS 回退；新增依赖需先在 GOAL.md 层面确认。
- git：每完成一组改动即用中文提交信息 commit；`dist/`、`release/` 不入库。
- 单文件不超过 500 行；rg 内容搜索；模型 ID 以 DeepSeek 官方文档为准
  （`deepseek-chat` 等旧名已停用）。

---
> Source: [linearuncle/xharness](https://github.com/linearuncle/xharness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
