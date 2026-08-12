## aiditor

> > 这个文件是给 Codex 看的项目状态说明,任何新的 Codex 会话开始前都必须读完。

# aiditor — Codex 工作交接

> 这个文件是给 Codex 看的项目状态说明,任何新的 Codex 会话开始前都必须读完。
> 用户在不同电脑之间切换工作环境,本文件保证上下文不丢失。

---

## 1. 项目是什么

**aiditor** —— 一个纯前端、零依赖、Blender 风格的通用编辑器框架。

当前产品边界分四块:

1. **AIditor Core/UI**:稳定零依赖内核,提供 Dock/Panel/Component、UI 组件库、主题、signal/log/bus/settings/history/workspace contract。
2. **AIditor AI Host**:可选上层模块,提供 agent runtime、provider、tools/context/operations、permissions、ChangeSet、compaction。它依赖 Core/UI,但 Core/UI 不依赖 AI。
3. **AIditor Extension Runtime**:可选上层模块,把 component/tool/context/reference/operation/settings/command/menu/dock panel contribution 安装进已有 registry。它不是第二套组件或 AI 模型。
4. **Demo Project Runtime**:示例宿主应用,用于演示“打开 workspace、加载文件、注册组件、挂到 dock”。它不属于框架层概念,不得写进 `src/` 的通用设计。

AIditor 仍坚持零依赖、零模块系统、单命名空间和 file:// 可运行。AI Host 和 Extension Runtime 是框架提供的可选能力,不是把 Core/UI 变成业务编辑器。

- **零构建**:经典 `<script>` 标签,直接 `file://` 双击 `index.html` 就能跑
- **零依赖**:不用 npm,不用打包工具,不用任何框架
- **单命名空间**:所有东西挂在 `window.aiditor` 下
- **三层架构**:Dock(布局容器)/ Panel(内容单元)/ Component(UI 组件)
- **核心机制**:不可变 N 叉分割树 + 自研 ~70 行响应式 signal + 按 dock id 的 keyed reconciliation

### 核心思想(读完这一段等于看懂了一半)

整个框架就是这一张图:

```
Layout(Blender 风格的 N 叉分割树)
 └─ Dock ×M                      ← 可以被分裂 / 合并 / 调整大小的矩形容器
     ├─ Toolbar(可选,top/bottom/left/right 四选一条)
     │   ├─ Static Items          ← dock 配置写死的(方向、初始按钮等)
     │   └─ Dynamic Items         ← active panel 动态贡献的,切 active 自动装卸
     └─ Panel ×N(同一时刻只有 1 个 active,也可能 0 个)
         └─ Component(真正渲染内容的 UI 组件)
```

**关键结论**(每一条都是一条硬约束,后面章节会细化):

1. **Dock 里装 0 ~ N 个 panel**,有 panel 时**总有一个 active**,只 active 的那个显示。
2. **Dock 的显示 = toolbar + panel content**,toolbar 可无,content 区永远存在(空 dock 时是空 div)。
3. **Toolbar 组件不分种类**:`tab-standard`、"关闭按钮"、"文件切换器"全都是平等的 toolbar 组件。tab component 的"特殊性"仅仅是它订阅了所在 dock 的 `panels` 和 `activeId` signal 从而能渲染成 tab 栏 —— 但框架不给它开任何特权 API,它用的 ctx 和别的 toolbar 组件完全一样。
4. **Toolbar 组件有两种来源**:
   - **Static**:dock 配置里写的,随 dock 生命周期存在
   - **Dynamic**:active panel 在 PanelData 里声明的 `toolbarItems[]`,panel 激活时自动挂到 toolbar,切走自动卸载。panel 跨 dock 移动时,动态 items 自然跟着 panel 走。
5. **Panel 可跨 dock 拖放,也可弹独立窗口**。Dock 可以配 `accept` 白名单限定只接受哪些 component 类型的 panel。跨 dock 拖放和同 dock 切 active 共享同一条 detach/re-attach 代码路径;跨窗口则走 `serialize`/`deserialize` 协议(§ 4.14)。
6. **多 panel 的性能要求是真的"只有 active 存在"**:非 active panel 的 contentEl **直接从 DOM detach**(不是 display:none,不是 content-visibility:hidden),浏览器对它零 layout、零 paint、零事件开销。切回 active 时 re-append,DOM 状态和 JS 对象完全保留。这是 § 4.3 的唯一实现路径。
7. **Panel / Dock 之间通讯走一条统一的解耦总线 `aiditor.bus`**:pub/sub,topic + payload,通过 `ctx.bus` 自动在 panel dispose 时取消订阅。没人直接持有别人的引用。

本文件是**最高优先级的工作交接与硬规则权威**。当前架构细节以 `doc/*.md` 为准,但不得违反本文件里的产品边界、零依赖、零模块系统、设计先行和代码风格红线。`doc/old/**` 只作历史资料。

---

## 2. 硬规则(不可违反)

这些是和用户多次对话后确立的红线,违反会让用户失望。

### 2.1 零应用级快捷键
**框架代码绝对不许内置"应用级/业务级"快捷键**。
我们是通用库,不是某个特定的编辑器应用。Focus Mode、关闭 panel、切换 tab、保存、命令面板、撤销/重做……所有"属于应用决策"的快捷键都**只暴露 API**(例如 `ctx.dock.toggleFocus()`),由调用方决定要不要绑键、绑哪个键。Demo 里可以演示一种绑法,但绝不写进 `src/`。

**但"组件内部语义键"允许,且必须有**——这不是快捷键,是组件自身功能的一部分,删掉组件就废了:
- **输入/编辑组件的编辑键**:textarea 的 Tab 缩进、codeInput 的 Tab、input 的 Enter 提交等。浏览器默认行为不满足组件语义时,组件必须自己 `preventDefault` + 处理
- **Overlay 的 dismiss 键**:modal / drawer / popover / menu 按 ESC 关闭最上层(由 `_overlay.js` 统一管,LIFO 栈)
- **Focus trap 的 Tab 循环**:modal 打开时 Tab 在 modal 内部循环,防止焦点跑到背后不可见元素。这是 WAI-ARIA 对 modal dialog 的硬性要求
- **进行中的交互的取消键**:拖拽 splitter / 拖 panel 过程中按 ESC 取消本次拖拽

判据很简单:**这个键绑了之后,是在替用户决定应用该怎么响应,还是在完成组件自己必须做的事?** 前者禁止(写 API 让调用方绑),后者允许(组件自己绑)。拿不准就当作前者。

### 2.2 零构建、零模块系统
**不许用 ES modules,不许引入打包工具,不许写 `import/export`。**
所有源文件都是 IIFE,挂载到 `window.aiditor`:
```js
;(function (aiditor) {
  'use strict'
  // ...
  aiditor.something = something
})(window.aiditor = window.aiditor || {})
```
HTML 用 `<script src="...">` 按依赖顺序加载。用户必须能双击 `index.html` 直接看到运行效果。

### 2.3 设计先行(Design-First)
**任何非平凡的改动,先写计划,等用户明确说"开始"再动代码。**
顺序:
1. 列数据模型 / 文件清单 / API 表面
2. 列出待决问题并明确请用户拍板
3. 用户回复后修订计划,可能多轮
4. 用户回复"开始" / "go" / "确认开始"才动代码

用户在动代码之前更正过设计方向多次。如果你跳过这一步直接动手,会浪费工作量。当用户说"先不着急改代码"或类似的话,意思就是只设计不写代码。

### 2.4 一个独立功能一个文件
独立的功能单元住在独立的文件里。`src/` 下用子目录把相关关注点分组(`core/`、`tree/`、`components/`、`dock/`、`style/`)。不要把 6 个不相关的概念塞进一个 800 行的文件。但也不要把 30 行的"焦点模式"硬拆出去 —— 见 § 5 的目录方案。

### 2.5 不写防御性代码
框架内部相互调用是受信任的契约,不需要 try/catch、null 检查、参数兜底。**只有在用户 component 调用边界用 `safeCall` 包裹**(因为用户代码可能抛错)。不为不可能发生的情况写代码。

### 2.6 不擅自加功能
- 没让你做的功能不要做("顺手清理一下"、"加点配置项"、"补个 docstring"全都不要)
- 没让你重构的代码不要重构
- 修 bug 时不连带改无关代码
- 不在没改的代码上加注释 / 类型 / 文档
- 不为假想的未来需求做准备

### 2.7 不擅自破坏性操作
不未经允许:`git push --force`、`git reset --hard`、删文件、改 git 配置、`--no-verify`。

---

## 3. 当前代码状态(已实现)

> 本节是当前实现快照。读完这一节就知道代码到什么程度了;不要用旧对话、旧阶段或 `doc/old/**` 推断当前架构。

### 3.1 目录(实际落盘)

```
aiditor/
  index.html                       # demo 入口 — 引用 dist/aiditor-full.{css,js} + demo widgets
  AGENTS.md                        # 本文件 — 工作交接与硬规则权威
  doc/
    old/editor_style.html          # 视觉调色板历史参考(只读,不改)

  tools/
    build.mjs                      # § 2.2 零构建承诺的载体:cat 带 banner,
                                   # 拼出轻量切片和 classic bundles;支持 --watch

  dist/                            # 已 commit 的 bundle 产物(保证零环境双击运行)
    aiditor-theme.js / .css        # 独立主题 runtime + tokens + 内置主题
    aiditor-mini.js / .css         # 独立网页常用 UI + 主题,不含编辑器重组件
    aiditor-editor.js / .css       # 独立完整通用编辑器 UI,不含 Dock/AI
    aiditor-kernel.js / .css       # Core services + tree + Dock runtime
    aiditor-ui.js / .css           # Kernel 之上的 UI/panel add-on
    aiditor-ai.js / .css           # AI Host + Extension Runtime add-on
    aiditor-core.js / .css         # Kernel + UI
    aiditor-full.js / .css         # Core + AI Host + Extension Runtime
    aiditor.js / .css              # core alias,保持经典路径可用

  .Codex/
    launch.json                    # Codex Preview 的 dev server 配置
                                   # (npx http-server -p 5570)

  src/
    core/                          # ⚠ 原 src/core/ 已并入这里(重构后的现状)
      signal.js                    # signal / effect / derived / batch / onCleanup
      log.js                       # aiditor.log signal + reportError + safeCall + 全局 window 兜底
      runtime.js                   # runtime script loader + owner-scoped contribution cleanup
      bus.js                       # aiditor.bus pub/sub + auto-unsubscribe
      registry.js                  # registerComponent / resolveComponent / componentDefaults
      context.js                   # ComponentContext 工厂(panel + dock + bus + signals)
    tree/
      tree.js                      # 不可变 N 叉树所有纯函数 + 框架级 transient 预览槽驱逐
    dock/
      runtime.js                   # PanelRuntime 生命周期 + activate + LRU + detached DOM
      render.js                    # reconcile / build / toolbar 两段渲染
      interactions.js              # splitter 拖拽 + 角拖 split/merge + 3×3 hover
      panel-drag.js                # tab tear-out + 跨 dock drop + pop-out(§ 4.14)
      migrate.js                   # 跨窗口 BroadcastChannel 协议 + serialize/deserialize
      layout.js                    # createDockLayout 入口胶水 + LayoutHandle(含 promotePanel)
    style/
      theme.css                    # 主题 v2 token(authoring → primitive/ramp → role → component)+ dark/dracula/light
      dock.css / component.css        # 框架自己的 dock + tab + toolbar 样式
      ui-base.css / ui-form.css / ui-property.css / ui-editor.css / ui-container.css / ui-data.css / ui-overlay.css / dock-tabs.css / ui-ai.css
    ai/
      permission.js                # 统一 permission resolver + audit + path rules
      store.js                     # agents/messages/quests/attachments 完整内存状态
      persistence.js               # IndexedDB 完整转录持久化 + localStorage 启动清单
      schema.js                    # tool/output 共用 JSON schema normalize/validate
      registries.js                # tools/skills/context/templates/bundles registry
      context.js                   # tool-call lifecycle + run context helpers
      request.js / runtime.js      # request assembly + scheduler/run/resume/tool approval
      checkpoints.js / evals.js    # 可选恢复检查点 + 基于 trace 的轻量确定性评估
      reference.js / change-set.js # references/operations + grouped review/apply
      target.js / rich-prompt.js   # add-to-chat targets + inline references
      provider*.js / adapter.js    # provider/connection/auth/transport/message tool protocol
      panels/                      # AI panel components(chat/transcript/settings/rich prompt 等)
    extensions/
      manifest.js                  # manifest normalize / public ids / trust + validation helpers
      install.js                   # contribution installers into existing registries
      runtime.js                   # Optional Extension Runtime lifecycle/review/storage/recovery/dock panels
      ai.js                        # Extension Runtime ↔ AI Host bridge(operations/tools)
    ui/                            # ⭐ UI 组件库(aiditor.ui.* 命名空间),按类别分目录
      _internal/                   # _portal / _floating / _drag / _signal / _overlay
      base/                        # button / iconButton / icon / tooltip / popover / kbd / badge / tag / spinner / divider
      form/                        # input / textarea / number / vector / slider / rangeSlider / checkbox / switch / radio /
                                   # segmented / select / combobox / colorInput / enumInput / tagInput / tab
      editor/                      # gradientInput / curveInput / codeInput / pathInput / fileInput
      container/                   # section / propRow / card / view / scrollArea / tabPanel
      timeline/                    # neutral numeric-axis layout + controlled Canvas/input surface
      data/                        # list / tree / table / breadcrumbs / progressBar(全部虚拟化)
      overlay/                     # menu / modal / drawer / alert / toast
      panel/                       # 能被 registerComponent 注册的 "panel 级" 内置 component
                                   # panel-list / inspector / dock-tabs(tab-standard/compact/collapsible/sidebar 预设) / log

  demo/                            # ⚠ 上一个 Codex 做了一次重构:单文件 ui-showcase.js 拆成 4 份
    catalog.js                     # 全部组件的 catalog(signals / mount / editFor)数据
    state.js                       # window.Demo 命名空间(selected / select / openCategory / signal cache)
    components/
      ui-gallery.js                # 组件库浏览与预览,展示内置/项目组件
      demo.css                     # demo component 的额外样式
```

**关键提示给下一个会话的 Codex**:
- **目录分层**:
  - `src/core/` = 零依赖底层 + component registry + context 工厂
  - `src/ai/` = Optional AI Host(agent/provider/tool/context/reference/operation/ChangeSet/permission/runtime)
  - `src/extensions/` = Optional Extension Runtime,安装 contribution 到已有 registry,并通过 owner 精确卸载
  - `src/ui/` = `aiditor.ui.*` 通用 UI 元件库(50+ 个)
  - `src/ui/panel/` = 内置 panel 级 component(panel-list / dock-tabs / history / log 等),用 `registerComponent` 注册,能直接塞进 dock
  - `demo/` = 用户层 demo,catalog+state 负责数据,components/ 负责示例面板
- 改完 `src/` 下任何文件**必须** `node tools/build.mjs` 重新生成 `dist/aiditor-core.*` / `dist/aiditor-full.*` / `dist/aiditor.*`,index.html 是直接引用 full dist 的,不重建就看不到改动
- **`demo/` 下的文件不进 bundle** —— index.html 直接 `<script>` 加载 demo/*.js,改完 reload 即可
- 写 dev server 时用 `.Codex/launch.json` 已配好的 `aiditor-demo`(端口 5570),不要自己拉新端口
- 文件加载顺序看 `tools/build.mjs` 的 `JS_ORDER` / `CSS_ORDER` 数组,**那是依赖序的唯一权威**

### 3.2 已实现的能力(完整清单)

**框架核心(§ 4 全部条款已落地)**:
- 不可变 N 叉分割树 + 所有纯函数写接口(addPanel/removePanel/movePanel/splitDock/mergeDocks/...,生成 id 的返回 `{tree, id}` 元组)
- 响应式核心 signal/effect/derived/batch/onCleanup,带依赖追踪
- 日志/错误系统 aiditor.log / reportError / safeCall + window error/unhandledrejection 全局兜底(§ 4.7)
- 通讯总线 aiditor.bus(pub/sub + auto-unsubscribe + 每个 handler 独立 safeCall 包裹)(§ 4.13)
- Component 注册表 + ComponentContext 工厂(§ 4.8 / § 4.9)
- Runtime loader `aiditor.runtime.loadScript` + owner-scoped cleanup。AI 新写 workspace panel 文件时,`aiditor.addPanelToDock({ component, dock, path })` 会先加载 `path` 注册 component,再放入 dock。
- History 支持同步/异步 apply;`jump/undo/redo` 返回 Promise,apply 失败时 index 不前移/后移。内置 `history` panel 是通用交互 primitive,只绑定 `aiditor.history` 实例,不接项目 saved/file journal 语义。
- Workspace v2 bounded contract:`readText/writeText`、二进制 blob IO、mkdir/copy/move/rename/delete、capabilities、稳定 stat(hash/mtime/size/kind)、object URL lease、bundle URL lease、snapshot/restore、可选 `revealInSystem` 平台能力。FSA/adapter IO 错误带稳定 `path/op/reason/code` 和 permission recovery 建议。它仍然只是文件边界,不是 project/asset 数据模型。
- Workspace 文本搜索使用统一 bounded walker:memory/FSA 直接复用,bridge adapter 在缺少 `search` 但具备 `list + readText` 时自动增强。结果统一返回 matches、部分错误、扫描统计和 limitHit,文件数/单文件大小/结果数均有界。
- `aiditor.shortcuts.dispatchKey(input, surface?)` 允许桌面/原生宿主直接派发结构化按键快照;DOM keydown 与宿主输入共用同一套归一化、context/scope、优先级、用户 override 和 command 路由,不模拟 KeyboardEvent,不维护残留修饰键状态。
- Dock 多 panel + detached DOM activate(§ 4.3)+ LRU dispose(§ 4.3)
- Toolbar 两段渲染(static + dynamic items)(§ 4.10)
- Focus mode / Collapsed / Transient(§ 4.4 / § 4.5)
- Blender 角拖 split + sibling merge + dirty 检查 + 3×3 hover(§ 4.1 / § 4.2)
- Panel 跨 dock 拖放(detach contentEl → re-attach,零重建)(§ 4.14)
- Pop out 独立窗口 + BroadcastChannel handshake + serialize/deserialize(§ 4.14)
- 内置 component:`panel-list` / `theme-config` / `tab-standard` / `tab-compact` / `tab-collapsible` / `tab-sidebar` / `history` / `log`

**UI 组件库(aiditor.ui.* 50 个组件)**:
- 全部基于 caller-owned `value: signal<T>` 的"信号优先"设计 —— 组件不持有自己的 state
- 全部走统一 cleanup 协议:`el.__aiditorCleanups: fn[]` + `aiditor.ui.dispose(el)`
- Overlay 走 `_portal.js` 的 `#aiditor-portal-root` 单例
- 数据组件(list/tree/table)直接虚拟化,tree 先 flatten 再复用 list 行;`fileBrowser` 是中性文件/列表/网格 primitive,`assetBrowser` 是兼容别名
- `aiditor.ui.timeline` 提供中性单数值轴布局与受控 Canvas/输入生命周期;数据、绘制、选择、命令、history、播放和领域语义由宿主拥有
- 全部 50+ 个组件 + 内部辅助 + 11 个 CSS 文件 = 已经 100% 编出 dist 并在 demo 里可以点
- `aiditor.inspector` 是 UI 层通用检查器协议:ordered targets + provider.inspect + 内置 `inspector` dock panel。多选显示第一个 target,只有所有 target 都有且可写的字段才可编辑;业务对象语义、校验、持久化由 provider 所属编辑器负责。

**AI Host(可选层)**:
- `src/ai/permission.js` 是统一 permission resolver / audit / path rule owner。Tools、operations、ChangeSet apply、workspace writes、extension install、host adapter 调用都走同一套 actor/target/scope 判断。
- `src/ai/registries.js` 统一管理 tools、skills、context providers、agent templates、bundles。Dotted name 是公开命名和筛选形状;extension 生命周期用 owner 精确清理。
- `src/ai/context.js` 只负责 tool-call lifecycle 和 run context helper,不再承担 registry。
- `src/ai/persistence.js` 将完整 JSON-safe 聊天转录异步保存到 IndexedDB;`localStorage` 只保存轻量 Agent 启动清单。模型上下文压缩只影响 provider request projection,不得删除 UI transcript。
- `src/ai/request.js` 负责请求组装:先解析 active skills/context,再只暴露 `agent.toolRefs + skill.tools + runtimeContext.tools` 的有效工具集;空引用不回退全量工具。Runtime kernel 保持最小,workspace/task/skill catalog 按需注入。
- Skill registry 使用统一 `SkillSpec` 和精确 owner/source/hash 生命周期;`skillRefs`、runtime intent、`auto(ctx)` 在每次请求只解析一次并写入 `skill_activated` trace。`src/ai/skill-packages.js` 可从 bounded workspace 加载标准 `SKILL.md` 并按需读取 `references/`,但绝不执行 package scripts 或绕过 Tool/permission registry。
- Agent 可声明 provider-neutral `outputSchema`;最终无工具调用的回复统一解析/校验到 `message.output`。Connection 层提供 bounded retry 与 health signal;流开始后不重放。可选 `ai.checkpoints` 只恢复安全排队状态,`ai.evals` 复用现有 trace 做确定性评估,都不引入项目 workflow 语义。
- Tool envelope (`toolProtocol`) 与参数可靠性 (`toolArguments: strict|structured|json|none`) 是正交能力。严格连接按请求期 Tool Schema 约束生成;Adapter 只做一次 wire decode;Runtime 先完整解码并保留整批结构化参数,再按原始 Schema 校验,最后才允许执行。完整 strict 批次可先做一次隐藏约束恢复;其他无副作用的参数错误以结构化 Tool Result 进入同一 run 的有限纠正状态机。重复指纹包含 Tool、规范化参数哈希和具体错误;trace/log 只保存有界摘要与哈希。联合 Schema 只有在所有分支共同声明唯一 `const`/单值 `enum` 判别字段时才展开具体分支错误;候选由 Schema 决定且与输入字段顺序无关,缺失分支属性只会使候选失效,不得抛异常或猜测业务形状。
- `src/ai/runtime.js` 负责 scheduler/run/resume/tool approval/continuation。Quest 使用统一 `{maxTurns,timeoutMs,maxTokens}` budget,达到边界后以稳定 `stopReason` 停止并通知父 Agent;模型委托另受 `maxDelegationDepth` 限制。
- `src/ai/orchestration.js` 的 `agent.create` / `agent.delegate` 共用父链解析:Agent 省略 `parentAgentId` 时默认创建自己的子节点,用户省略时创建根节点;Agent 不可逃逸到根级。`agent.read` 只返回有界 profile/树级摘要,`agent.configure` 只允许用户或父级修改后代的稳定配置,不能改自身或运行状态。Quest 用 `read/result/cancel` 形成精确任务闭环;`agent.stop` 只是当前 run 的紧急停止。模型控制工具用稳定 `outcome` 区分成功、无活动 run 和已终态 Quest,不存在/无权限仍然是错误。
- `src/ai/message-markdown.js` + `message-renderers.js` 负责安全 Markdown 文本渲染、主流 provider content block 归一化、结构化 message part renderer registry 和一致的复制文本;不执行 raw HTML,不把任意 JSON 猜成 UI。
- `src/ai/reference.js` 提供 references + operations 协议;`src/ai/change-set.js` 提供 grouped review/apply。
- Model-facing operation 只通过显式 `exposeToModel:true + inputSchema + preview/apply` 进入模型能力面;request builder 按当前 `available(ctx)` 把每个 operation 直接投影为 `operationId(inputSchema)` 并独立 strict 编译。模型看不到 preview/apply gateway、`{input}` 包装或 `previewId`;Runtime 以独立 `executorToolId/executorArgs` 隐藏路由回唯一 canonical preview/apply 生命周期,而日志、错误、权限审计和回放始终使用原 operation id 与直接输入。一个不兼容 schema 只能降级自己的投影。
- Rich prompt token 存 `refId`,不是 `resourceId`;chat attachments 是 runtime state,不是新的 model-facing registry。

**Extension Runtime(可选层)**:
- `src/extensions/manifest.js` 负责 manifest normalize、public id、trust 和结构校验 helper。
- `src/extensions/install.js` 负责把 contribution 安装进已有 component/AI/settings/commands registry。
- `src/extensions/runtime.js` 负责 review、install/update/uninstall、enable/disable、storage、safe mode、recovery、dock panel placement。
- `src/extensions/ai.js` 只负责把 Extension Runtime 的生命周期和 dock-panel 能力桥接成 AI operations/tools。
- Extension contribution 发布 dotted public name,例如 `sample.panel`;生命周期 owner 是 `extension:sample`。卸载/禁用优先 `unregisterOwner(owner)`,不能用裸 prefix 当最终生命周期边界。
- Extension Runtime 不创建第二套 component/tool/context/operation 模型,只把 contribution 安装到已有 registry。

**主题系统**:
- `src/style/theme.css` 采用 v2 分层:Authoring tokens(给主题作者改:`--aiditor-surface-*` / `--aiditor-text-*` / `--aiditor-stroke-*` / `--aiditor-brand*` / `--aiditor-state-*`) → primitive/ramp 兼容层(`--aiditor-c-00..11`) → role tokens(组件消费:`--aiditor-bg-N` / `--aiditor-fg-N` / `--aiditor-border*` / `--aiditor-accent*`) → 少量 component tokens(`--aiditor-toolbar-h` 等)
- 新代码不要直接在组件 CSS 里消费 `--aiditor-c-*`;组件只读 role tokens。`--aiditor-c-*` 保留给兼容和高级主题,不是用户自定义主题的主入口
- 派生半透明色优先从语义/role token 派生,例如 `--aiditor-scrim`、`--aiditor-accent-bg`;不要在 overlay / hover 里直接拿某个数字灰阶推断语义
- 主题能力不只包含颜色。shape / stroke width / elevation / root texture / corner accent 都属于通用 appearance role token。组件 CSS 只能消费这些通用 token,不能写 `.theme-neon .xxx` 这类主题专用分支,也不能在 JS 里判断 theme id 来改变组件结构
- 内置主题通过 `aiditor.theme.set(mode[, root])` 或 `data-aiditor-theme` 切换。`:root` 不写属性 = dark;`.aiditor-root[data-aiditor-theme=...]` 支持单实例 scoped theme
- 主题元数据只允许在 Core 的 `aiditor.theme` 配一次:mode id、label、color scheme 通过 `modes()` / `modeIds()` / `modeOptions()` / `hasMode(id)` 暴露。`src/style/theme-settings.js`、demo/AI theme schema、测试、文档示例都必须消费这份元数据,不要各写一份 `THEME_MODES`
- `src/style/theme.css` 只负责每个 `[data-aiditor-theme="<id>"]` 的视觉 token 实现,不要从 CSS selector 反解析主题列表
- 每个非默认主题块都**显式声明**自己的 bg/border 角色映射 + shadow 级别 —— 因为 :root 的 Godot"inset input"约定不是中性默认,被光亮 primitives 继承会反过来变成"raised in light"。各主题独立锁定,零耦合
- 内置 `theme-config` panel 复用 `src/style/theme-settings.js` 的主题设置实现,编辑 v2 authoring tokens,通过 `aiditor.theme` / `documentElement.style.setProperty` 写入并用 localStorage 持久化

**UX 微交互(2026-04-15 那一轮专门打磨)**:
- 按钮:hover 上抬 1px + 阴影,按下 `scale(0.985)` 内阴影,`::before` 伪元素 click flash,3px accent glow 替代实心 outline
- Primary button:accent 渐变背景 + hover accent glow 投影
- Checkbox:勾选符号 `cubic-bezier(.34,1.56,.64,1)` 弹簧 scale-rotate
- Switch:knob spring 滑动 + 按住时横向拉长(像被拽住)
- Radio:中心点 spring scale-in
- Slider thumb:hover/active 时 scale + 半透明 ring
- Segmented:active 项 accent 渐变背景 + 按下 `scale(0.96)`
- Menu item:hover 时 padding-left 微移 + active 项左侧 2px accent 标记条
- Tooltip / popover:`aiditor-fade-in` / `aiditor-pop-in` 入场动画
- Modal:backdrop `backdrop-filter: blur(4px)` 渐入 + modal 体微微上抛入场
- Toast / alert:进入 spring 缓动
- Dock tab:底部 2px 指示条用 `left/right` 滑动展开,hover 时撑到 30%,active 时铺满 + accent 光晕
- 全部包了 `@media (prefers-reduced-motion: reduce)` 降级

### 3.3 已知坑(给下一个 Codex 的避雷指南)

1. **`ui.bind(el, sig, fn)` 会同步触发一次 fn**。任何在 `fn` 里访问的变量必须在 `bind` 之前已经声明并初始化,否则 TDZ 报错。`src/style/theme-settings.js` 修过这个坑(`allSigs` / `refreshAll` 必须在绑定写入前定义好闭包关系)。
2. **`aiditor.effect(() => ...)` 也是同步触发**。如果 effect 体里向 `documentElement.style` 写 inline CSS variable,初次挂载那一刻就会把当前 signal 值写成 inline 样式,**inline specificity 会覆盖 `[data-aiditor-theme="light"]` 之类的属性选择器,主题切换从此失效**。修复模板:在 effect 里读 `getComputedStyle` 的 effective value,和想写的 literal 比较,相同就 return 跳过写入 —— 初次挂载零污染,只有用户真的编辑才写 inline。详见 `src/style/theme-settings.js` 的 `bindWriter`。
3. **不要在 component `factory(propsSig, ctx)` 里调 `ctx.panel.updateProps()` 高频化**(§ 4.9 已警告)。它写回 tree 触发 reconcile,keystroke 级别会卡。
4. **改了 `src/` 没 rebuild = 看不到改动**。每次都跑 `node tools/build.mjs`(或 `--watch`)。**改 `demo/` 不用 rebuild**,demo 是 `<script>` 直挂的,reload 即可。
5. **`registerComponent` 重名 throw**。同一个 component 不能注册两次,reload 时如果 demo component 文件被加载两次会炸。`index.html` 里 demo component 用 `<script>` 标签,默认不会重复。
6. **dist/aiditor-core.* / dist/aiditor-full.* / dist/aiditor.* 是已 commit 的产物**。改了源码之后 commit 时记得把 dist 一起 commit,否则克隆出去的人看不到效果。
7. **focus mode 有 CSS containing block 限制**(§ 4.5 已记录)。aiditor root 的祖先不能有 `transform/filter/perspective/will-change`。
8. **`addPanel(..., { transient: true })` 自动驱逐同 dock 已有 transient**(§ 4.4 框架级预览槽语义,2026-04-15 落地)。调用方不用自己写"找到现有 transient 再删"的胶水 —— tree 层已经做了。`LayoutHandle.promotePanel(panelId)` 负责"单击→preview / 双击→固定"的升级路径。
9. **所有可调常数的唯一存储是 `src/style/theme.css` 的 `--aiditor-*` token**。**不要**在 JS 里新写任何"默认时长 220ms / 默认阈值 6px / icon 映射表"。判据:"JS 要不要对这个值做数值运算?" 否 → CSS `var()`/`calc(var())`/`content: var()`;是 → `aiditor.ui.readNum('--aiditor-xxx', fallback)`。消费者看 `drawer.js` / `interactions.js` / `panel-drag.js` 的写法,不要复制旧习惯。

---

## 4. 架构决策(全部要遵守)

这些是和用户反复讨论后定下的,**不要再改**。如果觉得某条不对,先问用户,不要自作主张。

### 4.1 Split 角拖语义
分裂逻辑是**统一**的 —— 新 dock 是来源 dock 当前工作面的新视图:
- **有 active**:新 dock 初始 panel 克隆来源 dock 的 active `PanelData` 可序列化状态,但生成新的 panel id。也就是复制 `component`、`title/icon/dirty/badge`、`props`、`sourcePath/owner/extensionId`、`toolbarItems` 等数据,不复制 DOM/runtime。
- **无 active**(panels 为空):新 dock 也是空 `{ panels: [], activeId: null }`

来源 dock 的外壳配置也会复制到新 dock: `toolbar`、`accept`、`removeWhenEmpty:false`。不复制 `name`、`collapsed`、`focused`,避免稳定锚点冲突和显示状态泄漏。

也就是说,如果 active panel 是打开了 `foo.ts` 的编辑器,新 dock 出来的是另一个打开同一 `foo.ts` 的编辑器视图;如果 active panel 是打开了 `cube` 场景的 scene panel,新 dock 默认也是打开 `cube` 场景。它和 tab 拖拽不同:tab 拖拽是移动同一个 panel/runtime,角拖 split 是用同一份可序列化 PanelData 创建新的 panel/runtime。

### 4.2 Merge 语义
合并时**直接吞并**,被吞 dock 的所有 panels 默认**直接丢弃**。winner dock 保持自己的 panels 和 active 不变,只是面积扩大。这是 Blender-correct 的 —— merge 不是数据合并,是几何吞并。

**但是 dirty panel 不能静默丢失**(Blender 的 area 没有 dirty 概念,我们有,这里要偏离原版语义):
- 纯函数 `mergeDocks(tree, winnerId, loserId)` 的返回值是 `{ tree, discardedPanels }`(仍无副作用,`discardedPanels` 是被吞 dock 的 `panels[]` 快照)
- `interactions.js` 在 commit merge 前检查 `discardedPanels.some(p => p.dirty)`:
  - 无 dirty → 正常 commit
  - 有 dirty → 默认**阻止 merge**,preview 回滚,不弹任何 UI(框架不绑对话框)
- 可选钩子 `config.hooks.onDirtyDiscard?: (panels) => 'discard' | 'cancel'`:用户设置后,有 dirty 时调钩子,钩子返回 `'discard'` 才允许 merge。钩子缺省时永远当成 `'cancel'`

### 4.3 多 panel 的实现:Detached DOM(高效的本质)
这是最重要的性能决策,也是最简单的那个:**dock 的 content 容器在任何时刻只挂 active panel 的 contentEl,其他 panel 的 contentEl 从 DOM 完全 detach,保留在 runtime map 里作为 reference。**

切 active 的动作就三步:
1. 旧 active 的 `contentEl.remove()`(从 content 容器移除,**不销毁**,runtime map 还持有引用)
2. 新 active 的 `contentEl` 如果没创建过 → 懒调 `component.factory(propsSig, ctx)` 创建
3. `content.appendChild(newActive.contentEl)`

这就带来了架构性的收益:
- **浏览器对非 active panel 零渲染** —— 不 layout、不 paint、不触发 event handler、querySelector 查不到。对于 Monaco / WebGL / video 这种重组件,detached 状态下几乎不占 CPU
- **状态零丢失** —— DOM 节点和 JS 对象都还在,`<input>` 的 value、`scrollTop`、canvas 像素、Monaco 的撤销栈,全部保留。因为没销毁过
- **同窗口内的所有场景共享这一个机制**:切 active、tab 点击、跨 dock 拖放(§ 4.14)全部走同一条"detach → move reference → re-attach"的代码路径。不重建 component,不调序列化。跨窗口迁移是另一个场景,那个必须序列化(§ 4.14)
- **不需要 `content-visibility` / `display: none`** —— 这些还要走 style invalidation,detach 更彻底
- component 内部的定时器 / `setInterval` / WebSocket / `requestAnimationFrame` 框架不代管,component 作者订阅 `ctx.active`(一个 signal,见 § 4.9 ctx 表面)自己决定 detached 时是否暂停

**`ctx.active` 的精确语义**:"我的 `contentEl` 是否挂在 DOM 上"。注意它**不等于**"用户是否看得见我":
- **Collapsed dock**:只是 CSS `display:none` 整个 dock body,内部 contentEl 仍在 DOM 上。`ctx.active` **不受 collapsed 影响**,仍为 true
- 想对"看不看得见"做反应,component 自己订阅 `ctx.dock.collapsed` / `ctx.dock.focused`

"DOM 在不在" 和 "用户看不看得见" 是两个正交维度,框架各给一个 signal,component 按需订阅。

**LRU 作为"内存上限"策略**:
- 默认 `lru.max = -1`(不限,所有创建过的 panel runtime 一直留着)
- `lru.max = N`:runtime map 的条目数上限 = N
  - active panel 永远占一个名额,不会被淘汰
  - dirty panel 跳过淘汰(不丢未保存的东西)
  - 其他候选按 `lastActivatedAt` 升序,最老的**真 dispose**:调 `component.dispose(contentEl)` → 把 detached 的节点丢弃 → runtime map 删条目
  - **PanelData 仍留在 tree 里**,所以下次再 activate 这个 panel 时会重新走 `component.factory(propsSig, ctx)`,相当于"从 props 重建一次"。**只有 transient DOM state 会丢**:scroll 位置、未保存输入框草稿、撤销栈、canvas 像素、运行中的 WebGL context……凡是 component 没主动 `ctx.panel.updateProps(...)` 写回 tree 的东西都会丢。这是显式语义(像浏览器丢弃后台标签页),用户配 `lru.max` 就是在接受这个权衡
- `removePanel()` 也走 dispose 分支,且会**同时把 PanelData 从 tree 移除**(panel 被删就是被删,不是缓存淘汰)

这个模型的美感:**"非 active = detached"的规则统一了 LRU、切 active、tab 点击、跨 dock 拖放所有场景**。跨 dock 拖放的实现同样是 detach + move runtime + re-attach,零额外机制(§ 4.14)。

### 4.4 Transient Panel
**仅 API 层支持**,不做双击 / 编辑触发的自动升级:
- `addPanel(tree, dockId, panel, { transient: true })` —— 标记瞬态
- `ctx.panel.promote()` —— component 主动升级为常驻
- Tab 上瞬态 panel 标题用斜体显示
- 不监听双击事件,不监听编辑事件

### 4.5 Focus Mode
**仅 API 层支持**,纯 CSS 切换:
- `ctx.dock.toggleFocus()` / `ctx.dock.setFocus(bool)`
- 实现:给 dock 加 `data-focused` 属性,CSS `position:fixed; inset:0; z-index:100`
- **绝不绑任何快捷键**(参见 § 2.1)
- **已知限制**:当 aiditor root 元素的某个祖先设置了 CSS `transform` / `filter` / `perspective` / `will-change: transform` 时,`position:fixed` 会相对该祖先建立包含块而不是视口,focus 模式的 dock 不会铺满屏幕。这是 CSS 规范行为,框架不兜底;需要时调用方自己把 aiditor root 挂到 `<body>` 直接子级或用 `<dialog>` portal

### 4.6 Tab Component(就是一种 toolbar 组件,没特权)
Tab component **就是一个普通 toolbar 组件**,没有任何特殊 API。它"像 tab 栏"仅仅是因为:
- 它在 `factory(propsSig, ctx)` 里订阅了 `ctx.dock.panels`(signal)和 `ctx.dock.activeId`(signal)
- 它把每个 panel 渲染成一个按钮
- 点击调 `ctx.dock.activatePanel(id)`
- 关闭按钮调 `ctx.dock.removePanel(id)`

任何第三方 component 都能做同样的事。框架不给它开任何后门。

三个内置 tab component,**实现是同一个组件 + 三套预设默认 props**(不要写三份代码):
| Component | 关键默认 props |
|---|---|
| `tab-standard` | `closeButton:'hover'`;默认不显示加号 |
| `tab-compact` | `closeButton:'never', minShowCount:2`(单 panel 时 tab 栏隐藏) |
| `tab-collapsible` | `collapsible:true`(点击已激活 tab 折叠/展开整个 dock) |

tab 加号不是框架默认行为。需要加号时,宿主必须在 toolbar item 的 `props.addPanel` 显式声明要创建哪个 panel:
```js
{ component: 'tab-standard', props: { addPanel: { component: 'scene.empty', title: 'Scene' } } }
```
点击加号时 tab component 会读取 `componentDefaults(addPanel.component)`,再用 `addPanel` 覆盖 defaults 后调用 `ctx.dock.addPanel(...)`。空 dock 没配置 `addPanel` 时不显示无效加号。

**Tab component 永远是 static toolbar item**(写在 `dock.toolbar.items[]` 里,不随 panel 切换),因为它订阅的是整个 dock,不属于任何单一 panel。这意味着 tab component 的 ctx **没有 `ctx.panel`**,只有 `ctx.dock`,写代码时按这个约定即可。
Tab 上的 pointerdown 是 panel drag 会话的起点:tab component 在按钮上挂 pointerdown 监听,识别为 drag 后把控制权交给 `dock/interactions.js`,由它统一处理同 dock reorder / 跨 dock drop / pop out。tab component 自己**不做任何 drag 逻辑**,只负责"起个头"。

### 4.7 日志 / 错误处理系统
统一的日志流 + panel 错误隔离:
- `aiditor.log`:`signal([])`,每条 `{ id, time, level, source: { scope, dockId?, panelId?, component?, topic? }, message, error?, stack? }`
- `aiditor.log.push(level, source, message, error?)` / `aiditor.log.clear()` / `aiditor.log.dismiss(id)`
- `aiditor.reportError(source, err)` / `aiditor.safeCall(source, fn)`
- `aiditor.safeCall(source, fn)`:try/catch 包裹,失败 push 到 `aiditor.log` 的 `level:'error'` 条目并返回 `null`
- 所有 component `factory` / `dispose` 的调用都走 `safeCall` 包裹(同步边界)
- 单个 panel 出错只显示红色错误框,**不影响其他 panel**
- 内置 component `log`:订阅 `aiditor.log` 渲染日志/错误列表,用户可以把它放进任何 dock 当 "Problems" 面板用

**异步错误兜底**:`safeCall` 只抓同步调用栈,component 内部 `setTimeout` / `Promise` / `addEventListener` 抛的错抓不到。框架入口(`createDockLayout` 首次调用时)注册一次性的全局监听:
- `window.addEventListener('error', e => aiditor.reportError({ scope: 'global' }, e.error))`
- `window.addEventListener('unhandledrejection', e => aiditor.reportError({ scope: 'global' }, e.reason))`
- source 的 `scope: 'global'` 区分于 `scope: 'component'`,`log` component 可以按 scope 分组或过滤
- component 作用域的异步错误,component 作者可选地用 `ctx.safeCall(fn)` 手动包裹获得 panel-scoped 归因

**`signal.js` 的 effect cleanup 错误走 `console.error`,不路由 `aiditor.log`**。这是架构分层必然的裁决:`log.js` 用 `signal([])` 定义 `aiditor.log`,因此依赖 `signal.js`;反过来若 `signal.js` 调 `aiditor.reportError`,就成了循环依赖。边界划清 —— **signal.js 是零依赖底层,它的错误路径只走 `console.error`**。语义上也是对的:effect cleanup 失败是框架底层契约破裂(component 的 `onCleanup` 回调崩了),属于"fail-loud 到控制台"的范畴,不是归到某个 panel 的软日志列表里可以慢慢看的事。

### 4.8 Component 注册表(一种形态,无妥协)
**所有 component 必须先注册,panel 和 toolbar item 引用 component 只能用已注册名(string)。** 没有匿名 spec、没有 function 简写、没有未注册 component。这是一条硬规矩 —— 代价换来的是:tree 严格 JSON 可序列化、跨窗口迁移无需特殊分支、`accept` 白名单规则统一、文档只讲一种写法。

```js
aiditor.registerComponent(name, {
  factory: (propsSig, ctx) => HTMLElement,   // 必需
  defaults?: () => ({ title?, icon?, props?, toolbarItems? }),
  dispose?: (el) => void,
  serialize?: (el) => any,                // 可选,跨窗口迁移时用(§ 4.14)
  deserialize?: (el, state) => void,      // 可选,配对
})
```

- `factory(propsSig, ctx)` 必需,返回 component 的根 element;框架把它作为 `contentEl` 挂到 dock 的 content 区(panel component)或 toolbar 区(toolbar component)
- `defaults()` 可选,返回新建 panel 时的默认字段。§ 4.1 角拖分裂调这个。未提供则 fallback 到 `{ title: name, props: {} }`
- `dispose(el)` 可选,component 需要清理资源(取消订阅、关 WebSocket、terminate Worker)时实现
- `serialize` / `deserialize` 可选,**仅跨窗口迁移时调用**(§ 4.14)。同窗口内的 panel 切换、tab 点击、跨 dock 拖放都不经过序列化 —— 那些场景下 contentEl 本身就被整体迁移(detached DOM,§ 4.3)

**硬性 props 约束**:`panel.props` 和 `toolbarItem.props` 必须是 **JSON 可序列化的 plain object**。不许塞函数、DOM node、class instance、Map/Set、循环引用。理由:tree 要能跨窗口 `structuredClone`,要能 JSON 持久化。想传"行为"?通过 bus 发事件或订阅 signal,不要塞进 props。这条约束由调用方保证,框架不做运行时校验(§ 2.5 不写防御性代码)。

**配套 API**:
- `aiditor.registerComponent(name, spec)` —— 注册,重名 throw,名字必须是合法 string
- `aiditor.resolveComponent(name)` —— 查表返回 spec,未注册 throw
- `aiditor.componentDefaults(name)` —— `resolveComponent(name).defaults?.() ?? {}`,调用点不用判空
- 所有内置 component(`panel-list` / `theme-config` / `tab-standard` / `tab-compact` / `tab-collapsible` / `tab-sidebar` / `history` / `log`)本身也通过 `registerComponent` 注册,作为示范,不走后门

### 4.9 数据模型(完整,唯一来源)
本节是整个框架**所有数据结构的唯一定义处**。别的章节只讲行为,结构回这里查。

**数据分 6 层,依赖方向自上而下(下层只被上层引用,绝无反向)**:

```
Layer 1 │ LayoutConfig            ← 用户调 createDockLayout 的入参
Layer 2 │ Tree 结构(可序列化)    ← LayoutTree / SplitNode / DockData / PanelData / ToolbarConfig / ToolbarItemSpec
Layer 3 │ 注册表                   ← ComponentSpec(全局 Map)
Layer 4 │ 运行时外壳(内存)       ← LayoutRuntime / DockRuntime
Layer 5 │ Component 运行时            ← ComponentRuntime(3 种 kind 统一)+ ComponentContext
Layer 6 │ 纯函数 API(无 DOM)     ← tree/*.js 的全部写接口 + 返回值约定
```

**关键原则**(贯穿 6 层):
- **Layer 1~2 必须 `structuredClone` 安全**:tree 要能持久化、要能跨窗口 postMessage、要能 JSON 存盘。禁止塞函数 / DOM / Map / Set / class instance / 循环引用
- **Layer 4~5 绝不复制 tree 的状态**:所有"当前标题 / dirty / active / collapsed"都是从 tree 派生的 signal,不是 runtime 里另存一份。**single source of truth = tree**
- **Layer 6 结构共享**:所有写接口返回的新 tree 和旧 tree 共享未改动的子树,`===` 相等 → reconcile 零开销扫过

#### Layer 1 — LayoutConfig(入口契约)

```
createDockLayout(container: HTMLElement, config: LayoutConfig) → LayoutHandle

LayoutConfig = {
  tree:   LayoutTree,                // 初始布局(必填)
  lru?:   { max: number },           // 默认 { max: -1 }(不限,见 § 4.3)
  dockMenu?: boolean,                // 默认 false; true 时安装内置 Dock Menu contribution
  hooks?: {
    onDirtyDiscard?: (panels: PanelData[]) => 'discard' | 'cancel',   // § 4.2
  },
}

LayoutHandle = {
  tree():        LayoutTree,         // 只读快照
  setTree(t):    void,
  subscribe(fn): () => void,         // tree 变化订阅,返回退订

  // 便利 API = 对应纯函数 + setTree + 返回生成的 id
  addPanel(dockId, partial, opts?):               { panelId },
  removePanel(panelId):                           void,
  activatePanel(panelId):                         void,
  movePanel(panelId, dstDockId, dstIndex?):       void,
  splitDock(dockId, dir, side, ratio?, opts?):    { newDockId, newPanelId? },
  mergeDocks(winnerId, loserId):                  boolean,   // false = 被 dirty 阻止(§ 4.2)
}
```
LayoutHandle 是用户手里**唯一**持有的引用。想操作布局,就只用它 —— 不要直接持有 DockRuntime / ComponentRuntime(它们是内部实现)。

#### Layer 2 — Tree 结构(可序列化)

整个 tree 是**不可变 + 结构共享**的,任何写操作返回新根,未变的子树 `===` 旧引用。全部节点 JSON 可序列化。

- **LayoutTree**(根):
  ```
  LayoutTree = SplitNode | DockData
  ```
  根可以是一个 split,也可以是单个 dock(极小布局:只有 1 个 dock 填满容器)。

- **SplitNode**(布局骨架,N 叉):
  ```
  {
    type:      'split',
    direction: 'horizontal' | 'vertical',     // horizontal = 行(左→右),vertical = 列(上→下)
    sizes:     number[],                       // 归一化到和 ≈ 1,长度 = children.length
    children:  Array<SplitNode | DockData>,    // n ≥ 1;n = 1 时应被上游压平
  }
  ```
  **SplitNode 无 id** —— 它是结构骨架,没有"这个 split 是哪个"的需求。需要定位时走 `path: number[]`(从根到目标节点的 children 下标序列)。keyed reconcile 的稳定业务 key 仍然是 **dockId**;split/wrapper/splitter 是 renderer 专用 frame,按位置和子 dock 结构原地 patch,不把 split 暴露成公共 identity。

- **DockData**(叶子,矩形容器):
  ```
  {
    id:         string,                        // 框架生成 'dock-N'(§ 4.11)
    type:       'dock',
    panels:     PanelData[],                   // 0 ~ N 个
    activeId:   string | null,                 // panels 为空时必为 null;非空时必指向 panels 里某个 id
    toolbar?:   ToolbarConfig,                 // 无此字段 = 不渲染 toolbar,见 § 4.10
    accept?:    string[] | '*',                // panel 白名单(§ 4.12),默认 '*'
    removeWhenEmpty?: boolean,                 // 默认 true;false 时最后 panel 删除/移走后保留空 dock
    collapsed?: boolean,                        // 默认 false(§ 4.5 的 focused 和这个正交)
    focused?:   boolean,                        // 默认 false,全 tree 至多 1 个 dock 为 true
  }
  ```
  内置 dock 右键菜单启用时,`Panel -> Remove Dock When Empty` 切换同一个 `removeWhenEmpty` 标志。

- **PanelData**(内容单元):
  ```
  {
    id:            string,                     // 框架生成 'panel-N'(§ 4.11)
    component:        string,                     // 必须是已注册 component 名(§ 4.8)
    title?:        string,                     // 默认 = component 名
    icon?:         string,                     // 图标标识(emoji / 名字),可无
    dirty?:        boolean,                    // 默认 false,merge 时用(§ 4.2)
    badge?:        string | null,              // 小角标文字(未读计数等),默认 null
    props?:        object,                     // 传给 component.factory 的 props,默认 {},必须 JSON 可序列化
    transient?:    boolean,                    // 默认 false(§ 4.4,瞬态显示斜体)
    toolbarItems?: ToolbarItemSpec[],          // 此 panel 激活时贡献的动态工具栏项(§ 4.10)
  }
  ```
  **`props` 的硬性约束**(和 § 4.8 同一条):不许塞函数 / DOM / class instance / Map / Set / 循环引用。想传"行为"走 `aiditor.bus`,不要塞进 props。框架不做运行时校验 —— 这是调用方契约(§ 2.5)。
  **`lastActivatedAt` 不在这里** —— 它是运行时缓存,放 ComponentRuntime(见 Layer 5)。

- **ToolbarConfig**(dock 级 toolbar 配置):
  ```
  {
    direction: 'top' | 'bottom' | 'left' | 'right',    // 位置 4 选 1
    items:     ToolbarItemSpec[],                        // static 工具项(§ 4.10)
  }
  ```
  字段缺失 = 不渲染 toolbar。动态 items 来自 active panel 的 `PanelData.toolbarItems`,和这里的 static items 渲染时拼接(见 § 4.10)。

- **ToolbarItemSpec**(工具栏项,static / dynamic 共用同一结构):
  ```
  {
    id:     string,                            // 框架生成 'ti-N'(§ 4.11)
    component: string,                            // 必须是已注册 component 名(§ 4.8)
    props?: object,                            // JSON 可序列化,默认 {}
    align?: 'start' | 'end',                   // 默认 'start'(工具栏的起始端 / 结束端)
  }
  ```
  tab component 也是这种 spec,framework 不给它开任何特权字段(§ 4.6)。

#### Layer 3 — 注册表(ComponentSpec)

**ComponentSpec 的完整字段定义见 § 4.8**,这里不重复。

仅补一条:注册表是进程全局(`Map<string, ComponentSpec>`),不属于任何 LayoutRuntime —— 同一页面多个 createDockLayout 共享一套 component 定义。

#### Layer 4 — 运行时外壳(LayoutRuntime / DockRuntime)

运行时外壳**绝不重复存储 tree 里已有的状态**。它只存"为了给 DOM / component 服务必须有的东西":keyed reconcile 的锚点、component 实例的归属、全局计数器、hooks。

- **LayoutRuntime**(一个 createDockLayout 一份):
  ```
  {
    container:         HTMLElement,                // 调用方传入的根
    treeSig:           Signal<LayoutTree>,          // 唯一 tree 信号
    dockRuntimes:      Map<dockId, DockRuntime>,    // keyed reconcile 用:tree 里还在的 dock 才保留
    activationCounter: number,                      // 全局单调递增,给 ComponentRuntime.lastActivatedAt 派号
    lruMax:            number,                      // 来自 config.lru.max
    hooks:             LayoutConfig['hooks'],
    broadcastChannel:  BroadcastChannel | null,     // 首次 popOut 时懒建(§ 4.14)
  }
  ```
  - `dockRuntimes` 是 reconcile 循环的**keyed 查表**:新 tree 走一遍 → 见到某 dockId 已存在 → 复用 runtime + 复用 DOM;没存在 → 新建 runtime + 建 DOM;旧 tree 里有但新 tree 没有 → 等 surviving docks 完成 active panel re-home 后再 dispose runtime + 丢 DOM。这就是 § 4.3 "keyed reconciliation by dock id" 的实际载体。

- **DockRuntime**(每个 tree 里的 dock 一份):
  ```
  {
    id:                    string,                       // = DockData.id
    dataSig:               Signal<DockData>,              // tree 里该 dock 的实时镜像
    dockEl:                HTMLElement,                   // .aiditor-dock 根(keyed reconcile 的锚点)
    toolbarEl:             HTMLElement | null,            // .aiditor-toolbar(无 toolbar 时为 null)
    contentEl:             HTMLElement,                   // .aiditor-dock-content(active panel 的唯一挂载点)
    panelRuntimes:         Map<panelId, ComponentRuntime>,   // kind='panel',懒建懒销
    staticToolbarRuntimes: ComponentRuntime[],               // kind='toolbar-static',随 dock 生灭
  }
  ```
  - `dataSig` 是 dock 数据的 signal 镜像,`ctx.dock.panels` / `activeId` / `collapsed` 等都是 `derived(() => dataSig().xxx)`。reconcile 里只 `dataSig.set(newDockData)`,component 端自动沿链重算。
  - **动态 toolbar runtimes 不住在这里**,它们挂在贡献者 panel 的 ComponentRuntime 上(跟父 panel 走,见 Layer 5)。这是跨 dock 移动 panel 时"动态 items 自然跟着走"的根本原因。

#### Layer 5 — ComponentRuntime(三 kind 统一)

**所有 component 实例在运行时用统一的 `ComponentRuntime` 结构包装**,住在 DockRuntime 里(panel / static toolbar)或父 panel runtime 的 `dynamicToolbarRuntimes` 数组里(dynamic toolbar)。这是本架构的关键统一点 —— panel component / 静态 toolbar component / 动态 toolbar component **不是三种不同的对象**,而是**一个概念,三种生命周期**。

```
ComponentRuntime = {
  kind,                     // 'panel' | 'toolbar-static' | 'toolbar-dynamic'
  component,                   // string,已注册 component 名(§ 4.8)
  contentEl,                // HTMLElement | null,懒创建 + detached DOM(§ 4.3)
  ctx,                      // ComponentContext(kind 决定暴露哪些字段)
  cleanups,                 // Array<() => void>,dispose 时顺序跑完
  active,                   // signal<boolean>,§ 4.3 的"DOM 是否挂载"语义

  // 仅 kind === 'panel':
  data?,                    // signal<PanelData>,tree 中该 panel 的实时镜像
  dockRef?,                 // signal<string>,当前所在 dockId
  dynamicToolbarRuntimes?,  // Array<ComponentRuntime>,此 panel 贡献的 dynamic items
  lastActivatedAt?,         // 单调递增数字,用于激活选择 + LRU
}
```

**三种 kind 的归属与生命周期**(表):

| kind | 住在哪里 | 生命周期 | ctx 暴露字段 |
|---|---|---|---|
| `panel` | dock 的 `panelRuntimes: Map<panelId, ComponentRuntime>` | `addPanel` 建 / `removePanel` 或 LRU 销 | `ctx.panel` + `ctx.dock`(动态跟随 `dockRef`) |
| `toolbar-static` | dock 的 `staticToolbarRuntimes: ComponentRuntime[]` | 随 dock 整体生灭 | 仅 `ctx.dock`;`ctx.active` 常量 `true` |
| `toolbar-dynamic` | 父 panel runtime 的 `dynamicToolbarRuntimes` 数组 | 跟父 panel 走;父 panel 销则同销 | `ctx.panel` = 贡献方 panel;`ctx.dock` = panel 当前所在 dock(跟父 `dockRef` 动态) |

**三种 runtime 的架构统一点**(不是巧合,是设计):

1. **`cleanups` 字段三种 runtime 都有**。`ctx.bus.on(topic, h)` 永远把退订函数 push 进"当前 runtime 的 cleanups",runtime dispose 时一起跑 —— **订阅泄漏/static toolbar 没挂载点的问题从机制上不存在**。auto-unsubscribe 是 runtime 层的性质,不是 panel 层的特权
2. **`contentEl` 的懒创建 + detached DOM 规则(§ 4.3)三种 kind 共用**。切 active 的 detach / re-attach 代码路径一套
3. **跨 dock 移动**只操作 `kind='panel'` 的 runtime —— 把它从源 dock `panelRuntimes` 挪到目标 dock,改它的 `dockRef` signal。它的 `dynamicToolbarRuntimes` 是 panel runtime 的字段 —— 自然跟着走,零额外逻辑

#### `data` 是 signal,所有 panel 派生字段从它读

单一真源 = tree,单一更新路径 = `tree → data signal → derived`:

- reconcile tree 时,框架对每个 panel runtime 做 `runtime.data.set(newPanelData)`
- tree 是结构共享的不可变树,**未改动的 PanelData 对象在新旧 tree 里是 `===` 同一引用** —— signal 的 `Object.is` 脏检查自动跳过 notify。未变动的 panel 在 reconcile 里走过是零开销
- `ctx.panel.title` / `dirty` / `props` 全部是 `derived(() => runtime.data().xxx)` 形式,没有"runtime 里复制一份"的状态
- `ctx.panel.setTitle(s)` 的实现就是 `treeSig.set(updatePanel(treeSig.peek(), id, { title: s }))` —— 这次写入下一 reconcile 自动沿 data signal 传到 derived

**runtime 没有重复状态,没有同步问题**。

#### 为什么这样分

**为什么 `lastActivatedAt` 放 runtime 不放 PanelData?**
- 每次切 active 都触发 tree 不可变重建太重(新 dock、新 panels 数组、新 panel 对象)
- LRU 淘汰和"关闭 active 后新 active 选谁"两个用途都在 runtime 层查,不用碰 tree
- 跨窗口迁移时对端 `_activationCounter` 从 0 开始,丢失本端的激活历史是**正确的**(新环境自己产生新的"最近使用"序列)

**为什么 `dockRef` 是 signal?**
- Panel 跨 dock 移动(§ 4.14)时,component 订阅的 `ctx.dock.panels` / `activeId` / `collapsed` 等要自动指向新 dock
- 实现:`ctx.dock.panels = derived(() => tree.find(dockRef()).panels)`。dockRef 变 → derived 重算 → component effect 自动 re-run
- component 不感知自己被移动过,代码完全透明

#### ctx 完整表面

**共享部分**(三种 kind 都有):
```
ctx.active               // signal<boolean>
ctx.bus                  // auto-unsub 版 aiditor.bus,on() 自动挂当前 runtime 的 cleanups
ctx.onCleanup(fn)        // 手动注册 cleanup(给非 bus 的场景)
ctx.safeCall(fn)         // 异步回调归因到当前 runtime
ctx.dock = {
  id:        signal<string>,
  panels:    signal<PanelData[]>,
  activeId:  signal<string|null>,
  collapsed: signal<boolean>,
  focused:   signal<boolean>,
  activatePanel(id), removePanel(id), addPanel(partial),
  toggleFocus(), setFocus(b), setCollapsed(b),
}
```

**仅 `kind='panel'` 或 `kind='toolbar-dynamic'` 有**:
```
ctx.panel = {
  id,
  title: signal<string>,   setTitle(s),
  icon:  signal<string>,   setIcon(s),
  dirty: signal<boolean>,  setDirty(b),
  badge: signal<string|null>, setBadge(s),
  props: signal<object>,   updateProps(patch),
  promote(), close(), popOut(),
}
```

**`toolbar-static` component 的 ctx 没有 `ctx.panel`**,按约定使用即可,框架不做 null 检查(§ 2.5)。

#### `ctx.panel.updateProps(patch)` 是低频持久化点

这是 component 把"想持久化的状态"写回 tree 的**唯一通道**。每次调:浅合并到 `PanelData.props` → 提交新 tree → reconcile → derived 沿链传播。这是重操作,**不要在 keystroke / scroll / pointermove 等高频事件里调**。高频状态存 component 内部,在用户显式保存 / panel dispose / pop-out 之前 flush 一次即可。

典型例子:
- monaco component 打开新文件 → 调 `updateProps({ filePath: '...' })`(低频,用户动作触发)
- 实时光标位置 → **不调** updateProps,存在 component 实例字段里,需要时再 flush
- LRU 真 dispose 后重建,`factory(propsSig, ctx)` 拿到的 `props` 只包含被 updateProps 写回去的字段。其他全部是"只存在 contentEl 里"的临时状态 —— 这是显式契约,用户配 `lru.max` 时就在接受

#### Layer 6 — 纯函数 API(tree/tree.js,无 DOM)

所有写接口是**纯函数**:输入旧 tree,输出新 tree(结构共享);契约违反直接 throw(§ 2.5)。约定:**生成 id 的写函数返回 `{ tree, 新id }` 元组;不生成 id 的返回 tree 本身;`mergeDocks` 返回 `{ tree, discardedPanels }`(§ 4.2)。**

| 函数 | 签名 | 返回 | 备注 |
|---|---|---|---|
| `findDock` | `(tree, dockId)` | `{ node, path } \| null` | path = 从根到该 dock 的 children 下标序列 |
| `findPanel` | `(tree, panelId)` | `{ panel, dockId, path } \| null` | path 指向所在 dock |
| `getAt` | `(tree, path)` | `node \| null` | |
| `addPanel` | `(tree, dockId, partial, opts?)` | `{ tree, panelId }` | `opts.transient?: boolean`;`accept` 拒收则 throw(§ 4.12) |
| `removePanel` | `(tree, panelId)` | `tree` | 默认最后 panel 删除后移除非 root dock;`removeWhenEmpty:false` 时保留空 dock |
| `updatePanel` | `(tree, panelId, patch)` | `tree` | 浅合并到 PanelData |
| `activatePanel` | `(tree, panelId)` | `tree` | 只改所在 dock 的 `activeId` |
| `promotePanel` | `(tree, panelId)` | `tree` | 清 transient(§ 4.4) |
| `movePanel` | `(tree, panelId, dstDockId, dstIndex?)` | `tree` | 省略 `dstIndex` = append;`accept` 拒收则 throw;srcDock === dstDock 时退化为 reorder |
| `reorderPanel` | `(tree, panelId, newIndex)` | `tree` | = `movePanel(..., 同 dock, newIndex)` 的语法糖 |
| `updateDock` | `(tree, dockId, patch)` | `tree` | 浅合并 DockData(排除 panels / activeId) |
| `setCollapsed` | `(tree, dockId, bool)` | `tree` | |
| `setFocused` | `(tree, dockId, bool)` | `tree` | 置 true 时自动清其他 dock 的 `focused` |
| `splitDock` | `(tree, dockId, direction, side, ratio?, opts?)` | `{ tree, newDockId, newPanelId? }` | `side: 'before' \| 'after'`;`ratio ∈ (0,1)`,默认 0.5;runtime 默认按 § 4.1 传入 cloned active PanelData 和 dock shell;纯函数只消费 `opts.dock` / `opts.seedPanels` |
| `mergeDocks` | `(tree, winnerId, loserId)` | `{ tree, discardedPanels }` | 仅允许直接兄弟(§ 4.2);dirty 检查由调用方(`interactions.js`)做 |
| `resizeAt` | `(tree, splitPath, newSizes)` | `tree` | 直接改某个 split 的 sizes |
| `swapDocks` | `(tree, dockA, dockB)` | `tree` | 整个 DockData 对换位置(面板全跟着走) |

**id 生成永远走全局计数器**:`aiditor._nextPanelId()` / `aiditor._nextDockId()` / `aiditor._nextToolbarItemId()`,这三个计数器挂在 `aiditor` 上,进程内单调,跨 LayoutRuntime 共享。用户不传 id、不看 id(除非从返回值里读)—— 这是 § 4.11 的硬规矩。

### 4.10 Toolbar —— 单条,两种来源
**一个 dock 最多 1 条 toolbar,在 4 个方向(top/bottom/left/right)选 1 个。** 位置由 `dock.toolbar.direction` 决定。

**Toolbar 里的组件来自两种来源**(渲染时按顺序拼出来):
1. **Static items** —— 写在 `dock.toolbar.items[]` 里,随 dock 生命周期存在,不随 panel 切换变化
2. **Dynamic items** —— active panel 在 `PanelData.toolbarItems[]` 里声明,**panel 激活则显示,不激活则消失**

**ToolbarItemSpec 的字段定义见 § 4.9 Layer 2**。这里只讲它的行为。

**动态 items 的生命周期**:
- Dynamic items 的 contentEl 跟 panel contentEl 一样走**懒创建 + detached DOM**:panel 第一次激活时调 `component.factory(propsSig, ctx)` 创建,切走 active 时从 toolbar DOM detach(**不销毁**),切回来 re-append
- Panel 跨 dock 移动时,dynamic items 的 contentEl 跟着 panel 走,component 不重建;它们订阅的 `ctx.dock.*` signals 通过 PanelRuntime 的 `dockRef`(§ 4.9)自动指向新 dock
- Panel dispose 时,所有它贡献的 dynamic items 一起 dispose(走 `component.dispose`)

**渲染规则**:
- 切换 active panel 时 toolbar **不整体重建**,只 reconcile dynamic items 部分(static items 永远不动)
- **component 类型决定它能访问哪些 ctx 字段**(契约,不是检查):
  - **Panel component**(出现在 dock content 区):`ctx.panel` 指向自己 + `ctx.dock` 指向所在 dock
  - **Static toolbar component**(写在 `dock.toolbar.items`):只有 `ctx.dock`,没有 `ctx.panel`。tab component 属于这一类
  - **Dynamic toolbar component**(写在 `PanelData.toolbarItems`):`ctx.panel` 指向贡献它的 panel + `ctx.dock` 指向 panel 当前所在的 dock(跟随 `dockRef` 迁移)
  - component 作者按注册场景决定用哪些字段,框架不做 null 检查,不写 fallback

**Dock 无 toolbar 时**:整个 toolbar 区域完全不渲染,content 占满整个 dock。**不要默认插入 `tab-standard`**,这是显式契约 —— 用户没写 toolbar 就真的没有。dynamic items 此时也不显示(没挂载点)—— 这是 panel 作者应自检的边界(想显示就选个会配 toolbar 的 dock)。

**`activeId` 为 null 时**:content 区就是一个空 div,不显示提示文字、不接受 emptyContent 渲染器。想要占位,自己注册一个占位 component 当默认 panel。

### 4.11 id 生成 & active 自动切换
**所有 id 由框架生成,用户不传、不指定、不看见(除非主动从返回值读)。**
- `panel.id`:框架内部计数器 `panel-1 / panel-2 / ...`
- `dock.id`:框架内部计数器 `dock-1 / dock-2 / ...`
- 对应 API 改为只接 "部分 PanelData"(不含 id),执行后返回新生成的 id:
  - `addPanel(tree, dockId, partial, opts?) → { tree, panelId }`
  - `splitDock(tree, dockId, dir, side, ratio?, opts?) → { tree, newDockId, newPanelId? }`(分裂出空 dock 时 `newPanelId` 为 `undefined`)
  - `createDockLayout(config)` 顶层配置允许用"锚点名"(如 `name: 'sidebar'`)做稳定引用,内部依然生成 id 并维护一张 `name → id` 的查找表
- 重复 id 理论上不会发生(框架生成,全局单调)

**active 切换用激活计数,不用"前/后邻居"。**
- 每个 `LayoutRuntime` 内部维护单调递增 `activationCounter`。
- `activatePanel(panelId)` 执行时,在对应 PanelRuntime 上写 `runtime.lastActivatedAt = ++layout.activationCounter`(这是 runtime 的字段,不写回 tree)。
- 删除 active panel 后,纯 tree 层默认选择剩余 panels 的最后一个;运行时层负责 dispose 被删 panel 并保持 tree/runtime 一致。
- 同一个字段被 § 4.3 的 LRU 复用:`lru.max = N` 时,按 runtime 的 `lastActivatedAt` 升序挑最小的 **dispose**(真 dispose,不是 hide)

### 4.12 Panel 白名单(dock.accept)
每个 dock 可以声明自己接受哪些类型的 panel —— 这条规则同时服务于跨 dock 拖放的 drop zone 校验(§ 4.14):
- `dock.accept` 默认 `'*'`(接受任何 component 类型)
- 可以写成 `['monaco', 'preview']`(仅接受这两个注册名)
- **校验发生在纯函数层**:`addPanel(tree, dockId, partial)` 和 `movePanel(tree, srcDockId, panelId, dstDockId)` 在构造新 tree 之前检查目标 dock 的 `accept`,不匹配直接 throw(不是防御性代码,是显式契约)
- **拖放 UI 亮灯用的是同一份规则** —— preview 时查一次 `accept`,决定 drop zone 是绿还是红
- 因为 panel.component 字段永远是 string(§ 4.8),这条规则不存在边界 case

### 4.13 通讯总线 aiditor.bus(pub/sub,自动清理)
**所有 panel / dock / component 之间的解耦通讯走同一条总线。没人直接持有别人的 reference,也没人直接调别人的方法。**

API 极简:
```js
aiditor.bus.on(topic, handler) → unsubscribe fn
aiditor.bus.off(topic, handler)
aiditor.bus.emit(topic, payload)
```

- `topic` 是任意字符串,命名由用户自定(例如 `'file:opened'` / `'selection:changed'` / `'build:done'`),框架不限制也不校验
- `handler` 同步执行,`emit` 返回后所有订阅者都跑完了
- **每个 handler 都被 `safeCall` 单独包裹**:某个 handler 抛错只会路由到 `aiditor.log` 的 error 条目;通过 `ctx.bus.on` 订阅时 source 带订阅者所在 panel 的 dockId/panelId/component/topic,**不会中断同一 emit 的其他订阅者**。这是 § 4.7 错误隔离规则的延伸 —— 一条总线上互不信任的 component 需要有这个保证
- **auto-unsubscribe**:component 通过 `ctx.bus.on(topic, h)` 订阅时,框架自动把退订函数塞进 panel runtime 的 `cleanups` 列表,panel dispose 时一起清,**不存在订阅泄漏**。component 直接用 `aiditor.bus.on` 就没有自动清理,需要自己管
- 当前实现不内置系统 topic;应用需要系统事件时自行定义 topic。历史计划里的 `aiditor:*` topic 不再视为已实现 API
- **只做单向 emit**,不做 request/response。要"问/答"语义,自己定义两个 topic(`'foo:request'` / `'foo:response'`)+ 自带 correlation id
- bus 不做持久化,不做离线回放,订阅之前 emit 的事件收不到

**Bus vs Signal —— 用哪个?** 框架同时提供两套通讯原语,区分清楚不是教条而是性能和正确性问题:

| 维度 | `signal`(§ core/signal.js) | `bus`(§ 4.13) |
|---|---|---|
| 形态 | **状态**(有"当前值") | **事件**(只有"发生过") |
| 订阅时机 | 晚订阅也能通过 `signal()` 读到当前值 | 晚订阅收不到 emit 之前的事件 |
| 多源 | 一个 signal 一个 owner,谁拥有谁 set | 任何 component 都能 emit 到任何 topic |
| 重复值 | 同值 set 不通知(脏检查) | 每次 emit 都触发 |
| 数据流向 | 内层 → 外层(子组件读父 ctx 的 signal) | 横向(panel ↔ panel,跨 dock) |
| auto-cleanup | `effect` 通过 `onCleanup` 收集 | `ctx.bus.on` 通过 panel runtime cleanups 收集 |

**判定规则(一句话)**:**晚一点订阅的人,需不需要知道"我订阅之前发生的最后一次"?需要 → signal;不需要 → bus。**

具体落地:
- "当前 active panel 是哪个" → **signal**(`ctx.dock.activeId`),因为新挂载的 tab component 必须立刻知道当前是谁
- "panel 已激活" 通知 → **bus**(`aiditor:panel:activated`),因为这是事件,错过了不需要补
- "文件内容已修改"(dirty 标记) → **signal**(`ctx.panel.dirty`),状态
- "文件已保存" → **bus**(`'file:saved'` 之类),事件
- 跨 panel 共享的"当前选中的对象" → **signal**(由某个权威 panel 持有,放在 `aiditor` 上或通过 bus 广播一次拿到 ref;真要解耦还是首选"权威 panel emit 事件,关心方各自维护本地 cache")
- 一次性命令(关闭 / 保存 / 编译) → **bus**

### 4.14 Panel 跨 dock 拖放 & 弹出独立窗口
**这是核心能力,不是可选扩展。**

**跨 dock 拖放(同窗口内)**:
- Tab 栏上的 panel 可拖出,进入另一个 dock 的 tab 栏或 content 区 → 执行 `movePanel(tree, srcDockId, panelId, dstDockId)`
- 拖放过程中框架 hit-test 目标 dock,查目标 dock 的 `accept` 白名单(§ 4.12),不匹配 → 红色禁入 overlay;匹配 → 绿色 drop zone
- 实现上就是 **detach contentEl from src dock's content → attach to dst dock's content**,不重建 component,不丢状态。PanelRuntime 整个从源 dock 的 map 移到目标 dock 的 map
- 同 dock 内 tab reorder 是这个操作的退化场景(src === dst,只改 panels 数组顺序)

**弹出独立窗口**:
- `ctx.panel.popOut()` 或拖 panel 到窗口外触发
- 框架调 `window.open()` 开一个新 browser 窗口,加载同一份 `index.html`
- 新老窗口通过 `BroadcastChannel('aiditor-layout')` 建立连接
- 源窗口:调 `component.serialize(contentEl) → state`(可选 hook,不提供则 `state = undefined`),`BroadcastChannel.postMessage({ action: 'migrate', panelData, state })`,等目标窗口 ack,收到后才在源窗口 `component.dispose(contentEl)` + `removePanel`
- 目标窗口:收到 migrate → 决定落点 dock → `addPanel` → 懒激活时调 `component.factory(propsSig, ctx)` + 若有 state 再调 `component.deserialize(contentEl, state)`,完成后发 ack
- **Component spec 新增两个可选字段**配合跨窗口:
  ```
  serialize?:   (el) => any         // 返回一个可 structured-clone 的 state
  deserialize?: (el, state) => void // 在新创建的 el 上恢复 state
  ```
- 两个字段都没实现的 component **仍然可以跨窗口迁移**,只是状态只保留 `props`(等于重建)
- 细节:跨窗口期间的"谁拥有这个 panel"竞态用"源先冻结 → 发消息 → 目标 ack → 源清理"的两阶段 handshake 处理
- 独立窗口关闭时,它里面的 panel 消失(跟浏览器关标签页一样,框架不做自动回流)

### 4.15 明确不做的(out of scope)
**下列能力项目不做,即使功能上看起来"顺手就能加",也不做。** 要做请单独立项讨论。
- **Transient panel 的双击 / 编辑自动升级**:不监听双击、不监听编辑事件。只保留 `ctx.panel.promote()` API
- **Overlay / Peek**(VS Code 风格的悬浮编辑器):不在范围
- **命令系统**(command palette):框架不提供,用户自己搭
- **菜单系统**(右键菜单、顶栏菜单):同上
- **主题切换运行时 API**:只保证 CSS 变量命名规范(§ 5),不做 `setTheme()`
- **任何内置快捷键**(再次强调,见 § 2.1)

---

## 5. 目录与维护规则

目录边界必须表达概念边界:

```text
src/core/          Core primitives, registry, context, settings, commands, workspace
src/tree/          Immutable dock tree pure functions
src/dock/          Dock runtime, render, interactions, drag, migration, layout
src/ui/            Generic UI component library
src/ai/            Optional AI Host
src/extensions/    Optional Extension Runtime + AI bridge
demo/              Host/demo app code, not framework design
```

唯一权威映射:

- 文件职责看 `doc/implementation-map.md`。
- 架构边界看 `doc/architecture.md`。
- AI registry / permission / context 细节看 `doc/ai*.md`。
- Extension 最终语义看 `doc/extensions.md`。
- 构建加载顺序看 `tools/build.mjs` 的 `JS_ORDER` / `CSS_ORDER`。

维护规则:

- 改 `src/` 后必须跑 `node tools/build.mjs`,并提交 `dist/aiditor-core.*` / `dist/aiditor-full.*` / `dist/aiditor.*`。
- 改 `demo/` 不需要 rebuild,但需要 reload demo 验证。
- `src/` 继续保持 IIFE + `window.aiditor` 单命名空间;不写 `import/export`。
- 新 framework 能力必须进正确层:Core/UI、AI Host、Extension Runtime、Demo Runtime 不能互相偷概念。
- Extension contribution 发布 dotted public name,但生命周期 owner 是 `extension:<id>`;卸载/禁用用 owner 精确清理。
- AI Host 的 model-facing 主概念保持 Agent / Tool / Context Reference / Operation / ChangeSet;targets、attachments、rich prompt、quests、bundles、templates 是 runtime/UX 细节。
- 所有组件和 toolbar item 引用 component 都只能用已注册 string name。
- CSS 可调常数优先放在 `src/style/theme.css` 的 `--aiditor-*` token;JS 只有需要数值计算时用 `aiditor.ui.readNum(...)`。

## 6. 验证入口

常规门禁:

```powershell
node tools/build.mjs
npm.cmd run check
npm.cmd run check:dist
git diff --check
```

当前 `npm.cmd run check` 覆盖语法检查、signal/tree/theme/history/i18n/settings/commands/workspace、UI scope/edit session、project runtime、ChangeSet、AI provider/retry/stream/tools/workdir/orchestration/quest/structured output/checkpoint/eval/persistence/compaction/target/reference/resource permission、Extension Runtime、rich prompt 等测试。

`git diff --check` 在 Windows 上可能打印 LF/CRLF 替换 warning;只要没有 whitespace error 即可。

## 7. 与用户协作的方式

- 用户用中文,你也用中文回复。
- 用户要求审查时,先讲真实问题,不要为了显得乐观把风险淡化。
- 用户要求按最终形态判断时,不要用“暂时不做”当理由;只判断最终模型是否简洁、优雅、稳定。
- 非平凡改动遵守 design-first:先列模型/API/文件清单,等用户明确说“开始”再改代码。
- 如果文档和代码冲突,先确认是文档落后还是代码没实现;不要盖错方向。
- 有更好的方案要直说,但保持简洁,不要长篇自我解释。
- 新踩的坑要回写到 § 3.3 或对应 `doc/*.md`,不要留在对话里。

## 8. 当前交付状态

当前主线已经完成:

- Core/UI、AI Host、Extension Runtime、Demo Runtime 的边界已对齐。
- Extension Runtime 已位于 `src/extensions/`,不再挂在 Core 目录下。
- `src/extensions/manifest.js` / `install.js` / `runtime.js` 分别负责 manifest helper、registry install、lifecycle/recovery/dock panel placement。
- `src/extensions/ai.js` 负责 Extension Runtime 与 AI Host 的 operations/tools bridge。
- `src/ai/permission.js` 负责统一 permission resolver/audit/path rules。
- `src/ai/registries.js` 负责 tools/skills/context/templates/bundles registry。
- rich prompt 使用 `refId`;chat attachments 不是新的 model-facing registry。
- Extension 卸载按 owner 精确清理,并有 nested extension 回归测试覆盖主要 registry。
- 发布边界的最终目标是 core/full 双 bundle 和干净 npm runtime 包;当前代码优化计划应优先落这里。

2026-05-22 最新一轮 workspace/framework 优化已完成但接力时需要确认是否已 commit/push:

- `src/core/workspace.js` 已升级为 Workspace v2 bounded contract:
  - `capabilities()` 返回适配器能力 truth table。
  - `mkdir(path)`、`copy(from,to)`、`move(from,to)` / `rename(from,to)`、`delete(path,{recursive})` 已接入 memory/FSA adapter。
  - `readBlob(path)` / `writeBlob(path, blob)` 支持二进制 IO;`stat(path)` 对文件返回稳定 `hash/mtime/size/kind`。
  - `createObjectUrl(path)` 返回 `{ url, path, hash, size, mime, release }` lease;支持 owner cleanup。
  - `createUrlBundle(paths)` 为 glTF/模型这类多文件预览提供 bundle lease,但框架不解析资源图。
  - `revealInSystem(path,{select})` 是可选平台 adapter capability;memory/FSA 默认 `false`,desktop/native bridge 可实现,返回稳定 reason,不暴露绝对路径。
  - `snapshot(path[, { binary:true }])` / `compareSnapshot()` / `restoreSnapshot()` 只提供 undo/redo 的底层快照存储和 CAS,不是 FileOperationJournal。
- `src/ai/workdir.js` 新增 model-facing 通用 workspace tools:
  - `workspace.capabilities`
  - `workspace.mkdir`
  - `workspace.copy`
  - `workspace.move`
  - `workspace.delete`
  这些都是泛用文件操作,带 preview/apply;没有引入 project/asset 概念。
- `src/ui/data/fileBrowser.js` 实现中性的 `aiditor.ui.fileBrowser`;`aiditor.ui.assetBrowser` 是同一份通用合同的别名,不引入第二套 asset 模型。
- 文档已对齐:
  - `doc/workspace.md`
  - `doc/resource-versioning.md`
  - `doc/ui.md`
  - `doc/ai.md`
  - `doc/implementation-map.md`
- 测试已通过:
  - `node tests/workspace.test.mjs`
  - `node tests/ai-workdir.test.mjs`
  - `node tools/check-syntax.mjs`
  - `node tools/build.mjs`
  - `npm run api:docs`
  - `npm run check`
- 本轮改了 `src/`,所以 dist 已重建。若回家电脑拉取后看不到这些改动,优先检查当前电脑是否已 commit/push。

本轮刻意没有接入的内容:

- 不把 project/asset 数据库、资源导入规则、业务级 file operation journal 放进 Core。
- 不让 workspace 解析 glTF、图片、场景、schema 或任何业务资源格式。
- 不把 UI tree/inspector 的领域语义和 workspace 文件语义耦合。

2026-05-26 已补充 Workspace V2 定稿设计文档:

- `doc/workspace-v2.md` 是目标设计,不是当前实现清单。
- 最终边界:Core 提供 bounded file access、operation review、CAS/version check、snapshot storage primitive、object URL lease、permission recovery;Game Aiditor/宿主负责 EditorCommand、HistoryService、FileOperationJournal、FileIndex 刷新、引用更新、冲突 UI 和 domain validation。
- `previewOperation/applyOperation` 是文件系统操作 review primitive,不是 editor history,不是事务数据库,不承诺跨文件原子性。
- `doc/workspace-v2.md` 只描述最终模型:统一使用 `readText/writeText`、Core `previewOperation/applyOperation`、严格 CAS/overwrite 语义和完整 snapshot/URL lease 测试矩阵;不要把兼容/过渡层写进最终设计。
- `doc/host-file-workflow.md` 描述宿主推荐方案:FileIndex、reference repair、FileOperationJournal、HistoryService、冲突 UI 和 domain validation 都属于宿主层,不能回流进 Core workspace。

提交/交接时必须确认新增文件已纳入版本控制:

```text
src/ai/permission.js
src/ai/registries.js
src/extensions/runtime.js
src/extensions/ai.js
```

---
> Source: [jinruozai/aiditor](https://github.com/jinruozai/aiditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
