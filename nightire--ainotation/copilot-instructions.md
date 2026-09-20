## ainotation

> 框架无关的 DOM 标注与原生动画检查工具，通过 Script/SDK 接入开发者项目。

# Ainotation

框架无关的 DOM 标注与原生动画检查工具，通过 Script/SDK 接入开发者项目。
以下选型已确定，无需在后续会话中重复确认。

## 技术栈

- 核心：TypeScript（strict），不依赖宿主渲染框架。
- UI：Lit + Shadow DOM；原生 CSS / CSS Variables；Lucide 图标。
- 动画与捕获：Web Animations API、Screen Capture API 等原生浏览器能力，渐进增强。
- 截图编辑：位图底图 + SVG 标记；Canvas 负责裁剪与图片合成。暂不引入 Konva。
- 图片标注从 marker popover 的 Screenshot action 进入实时页面绘图，Option/Alt 临时穿透、松开恢复绘制；支持粘贴、拖入、选择 PNG/JPEG/WebP 图片。完成生成 PNG 附件，先存入当前标注草稿，随 feedback 保存。图片元数据进入反馈契约，位图使用 IndexedDB Blob 与独立的鉴权传输，不塞入 JSON 同步正文；带图导出为 JSON/Markdown/PNG 的 ZIP，MCP 通过 ainotation_get_image 按需读取。
- 数据契约：Zod + JSON Schema；反馈文档独立于 UI、SVG 和浏览器运行时对象。
- 本地存储：IndexedDB + idb，图片使用 Blob，不放入 localStorage。
- 工具链：Vite+；应用使用 `vp dev` / `vp build`，SDK 与 MCP 包使用 `vp pack`。
- 组件开发：Storybook + `@storybook/web-components-vite`。
- 测试：Vitest + Playwright；保留小型 Playground 验证真实宿主、动画和屏幕捕获。
- 包管理与发布：pnpm workspace + Changesets，结合 Vite+ 管理。
- Agent 集成：独立 Node.js LTS 进程 + 官方 MCP SDK；浏览器通过 HTTP/SSE 通信。

## 实现边界

- MCP 是首版正式能力，但基础标注不依赖其运行；未连接时可复制反馈、导出附件。两种出口共用数据契约。
- SDK 支持 `mcp: false` 的实例级仅本地模式：不读取或修改 MCP 凭据、不连接或同步、不显示 MCP 连接配置与状态，保留标注、图片、复制和导出；不能与 `development` 同时设置。官网演示使用该模式，默认和 Vite 自动配对行为保持兼容。
- 普通标注开放持久化 CRUD 与批量复制/导出；通用多轮对话模型保留在内部，UI 与 MCP 不开放回复或处理状态工作流。UI Variants 使用独立的结构化探索状态与专用工具。
- UI Variants 需要支持 uiVariants:1 的 MCP 连接，mcp:false 不显示入口；每个项目和完整 URL 同时只有一个活跃探索，普通标注仍可使用。默认三个候选、最多六个，原版另计；目标集合固定且根节点不得重叠，多目标按完整方案联动，允许结构变化和切换时重置局部状态。宿主通过框架无关的 defineVariants 订阅快照并显式 bind DOM 根节点，单分支渲染；普通标注身份规则保持严格。Vite 提供 virtual:ainotation/variants 和生产原版回退，候选实现/CSS 需开发条件导入。保留原始快照，暂停关联目标样式覆盖且保留草稿；缺失、重复、混合轮次或渲染失败不得报告就绪，失败切换回退上一方案。用户采用/重新生成/取消仅持久化决定，由用户主动告知 Agent 继续；不挂起工具调用、不自动唤醒会话。MCP 以轮次/版本校验登记候选及确认最终源码清理，指南同时通过 resource 和 tool 提供。取消到 Agent 完成清理期间仍占用探索；删除标注只结束预览，不自动修改宿主源码。
- 删除尚未清理的 UI Variants 标注（包括批量清除和 MCP 删除）会取消探索并恢复原版预览，在 document.variantCleanups 中独立保留原标注 ID、原始页面／目标上下文、候选、取消决定和删除原因。标注与 marker 立即移除，页面提示等待 Agent 清理；MCP 列表／反馈发现待办，get_variants 和 complete_variants 继续接受原标注 ID。完成前仍占用项目和完整 URL；完成记录仅作回执，不再次要求源码修改。刷新、服务重启和浏览器恢复不得静默丢弃待办，旧请求不得复活已删标注；携带清理记录的同步需要 uiVariantsCleanup:1 能力。删除不自动唤醒 Agent 或修改源码。
- UI Variants 完成清理后，原标注开关恢复关闭并可再次开启；显式开启并保存创建新探索 ID，从第 1 轮开始，不沿用旧候选、报告或决定，普通保存保留已完成记录。重新请求需匹配旧探索的完成快照，拒绝过期请求和同 URL 的并发占用。旧草稿残留的开启状态在完成与刷新时清理，显式新请求的草稿另存原探索身份以便恢复。
- UI Variants 控制器使用上一项／下一项循环导航，只显示当前方案和页码（包含原版），省略就绪说明。默认页面底部居中，窄屏避让工具栏；非交互空白区域支持拖拽，聚焦面板后方向键移动，输入／按钮／触摸滚动与拖拽后防误触规则沿用标注编辑器。位置按项目和完整 URL 本地持久化，刷新恢复且限制在可见视口内，与标注编辑器的位置独立，不进入反馈或 MCP 数据。
- UI Variants 控制器右上角只保留最小化／展开按钮，最小化只显示可拖动的标题行；保留当前方案、反馈草稿和内容展开状态，不改变探索状态或恢复拾取。最小化是当前探索的本地界面状态，切换页面或开始新探索时恢复展开；标注编辑器仍不提供最小化按钮。结束探索放在底部按钮组最后，使用 destructive 样式并通过模态对话框二次确认；默认聚焦继续比较，Escape 返回，仅确认后记录取消。对话框不持久化，探索版本变化、离页或隐藏面板时关闭，避免取消新的探索状态。
- UI Variants 当前轮候选发布后，比较期间暂停全页拾取并隐藏选取框与页面 marker，让宿主原生点击、输入、Shift 与 Escape 直接生效；控制器及“查看标注”面板保持可操作。暂停状态独立于手动 Alt 穿透和用户的选取开关，不清除草稿与已选目标；确认、重新生成、取消、删除或离页后解除，下一轮发布再暂停。暂停开始时取消未完成的选取手势，原生手势可跨恢复边界完成；收起检查器时不得自动重新开启选取。
- 标注与草稿按项目和完整页面 URL 隔离；复制汇总当前项目所有已保存页面并按 URL 分组，JSON 导出与清除操作作用于当前页面。路由往返时重新校验目标身份并恢复对应页面的 marker。
- 支持正文文本选区标注与有边界的通用 DOM 上下文采集。Settings 提供 Compact / Standard / Detailed / Everything 四档 Markdown 输出，默认 Standard；档位不裁剪持久化、JSON 导出或 MCP 数据。React 组件链和源码定位不属于当前阶段。
- 标注面板提供 Feedback / Styles 页签。样式按页面目标共享，以 document.targetStyles 为唯一数据源，目标 ID / styleTargetId 引用，inline styleChanges 是兼容性投影；多个 Marker 指向同一真实 DOM 节点时共享草稿和已保存样式，不按选择符相同直接合并。工具栏全局预览默认开启，局部开关仅影响当前目标/多选范围；全局关闭还原整页并禁用局部开关，重开恢复局部选择。保存、关闭编辑器、收起工具保留整页预览，取消回到当前编辑范围的已保存值；离页/卸载清理覆盖且保留宿主新写入，样式漂移需确认，缺失/替换节点不重绑。Shift 多选默认批量编辑，支持混合值、相对键盘/滚轮微调、黄色圆点各自恢复、四边联动和原子撤销；局部预览支持部分选中态。共享草稿/预览设置按项目和完整 URL 隔离，删除最后一个 Marker 引用时清理共享样式；原始快照不变，贯通 Markdown/JSON/ZIP/MCP，父级导航保留修改且单标注最多 20 个目标。旧 MCP 服务缺少 styleSuggestions/sharedStyles 能力时暂停相关同步并保留本地数据。
- 核心逻辑、反馈文档、UI、集成分离；无需提前把每个模块拆成发布包。
- 选择元素即持续观察其子树中的动画；多选取并集，不扩大到共同祖先。观察不暂停页面，显式检查才接管动画。
- Inspector 展开为 toolbar 并持续选取，Shift 临时多选、Option/Alt 临时穿透真实页面交互；标注通过页面 marker/popover 就地增删改。Toolbar 提供复制、导出、清除当前页面全部标注、Settings 和关闭，连接配置在 Settings popover 中。收起隐藏页面标记并停止拾取，不删除持久化数据。
- 编辑器打开时普通外部点击只收起并保留草稿，下一次点击才选取；Shift 多选、文本选区和 Option/Alt 穿透保留。按 Marker、项目和完整 URL 记忆编辑页签、拖拽位置及目标范围，保存新 Marker 时继承草稿的编辑上下文；父子导航不隐式扩大批量范围。初始隐藏/未挂载目标出现后重试严格身份校验，已绑定节点被替换时不重绑。样式输入失焦不得同步重入正在移除节点的渲染过程。
- 标注编辑器不显示最小化按钮或拖拽把手，拖动非交互空白区域即可移动面板；输入、选择、按钮、标签、展开标题、定位文本及滚动条保留原有交互，触摸滚动区域优先滚动。外部点击收起编辑器；聚焦面板本身时可用方向键移动。拖拽结束不得误触按钮，离开、失去指针捕获和卸载时清理拖拽状态。
- Padding / Margin 分成独立间距组，默认各一个四边联动输入；两个模式按钮分别展开水平/垂直联动或四边独立输入，再次点击激活项回到默认。展开时保留四边总输入，切换模式不修改样式；混合值、相对微调、黄色圆点按范围各自还原及原子撤销同时覆盖各边与多选目标。
- 本地 editorViews 另存选择项及各自父子返回路径（最多 20 项、每项 64 层），与完整关联修改目标分离。保存/重开/刷新恢复导航结构，单条父子导航不转成并列多选；显式 Shift 多选保持独立。返回前验证真实节点身份及直接父子关系，旧记录/无效导航元数据或已改变的关联目标集合回退目标列表，不根据 DOM 嵌套推断操作历史。导航上下文不进入 Agent 反馈契约。
- Trigger 与 toolbar 共用移动锚点；位置和展开方向按项目持久化，卸载、重新挂载和刷新后以收起态恢复，并限制在可见视口内。
- Ainotation UI 支持 Light / Dark 主题，默认 Light，在 Settings 切换并按项目持久化；主题覆盖 toolbar、trigger、Settings 和 marker/popover，不修改宿主页面配色或反馈快照。
- UI 国际化使用 SDK 内部类型安全字典与实例级语言状态，不新增 i18n 运行时库。Settings 支持 zh-Hans / zh-Hant / en / ja / ko，首次匹配浏览器语言、回退英文，手动选择按项目持久化；切换不重建编辑器，不翻译用户反馈、页面原文、JSON/MCP 标识或 Markdown 交接结构。
- 暂不做浏览器扩展、任意 JS 动画倒放、录像/GIF、云端账号和多人协作；源码定位为可选增强。
- 工具运行时依赖由 SDK 自行提供，不要求宿主安装 Lit；重模块按需加载。
- 原生资源生命周期独立于 UI 渲染；销毁时清理监听器、动画控制和媒体流。
- MCP 服务默认仅监听 loopback，并实现本地鉴权、Origin 校验和会话隔离。
- MCP 使用数据目录外的私有 IPC 协调器保护共享服务归属；缺失运行信息、注册表和已加载反馈可从内存重建，损坏 JSON 保留后尝试备份。恢复同步通过存储代次、版本校验及缺失图片列表避免静默覆盖；Settings 支持重试、项目恢复和版本选择。`doctor` 只读，`repair` 默认保留数据，显式重置与外部备份恢复要求服务停机；详见 `RECOVERY.md`。
- 标准接入通过 Vite 插件 `ainotation({ name, id? })` 声明项目，不要求独立身份文件或 init。插件启动时注册本机项目；MCP 按 roots/目录限定工作区，通过 `ainotation_list_projects` 发现 app，每次调用用可选 `project` 参数按名称、稳定键或 UUID 选择。单项目自动选择，多项目或同名歧义返回候选；不使用全局“当前项目”，不越过工作区范围。显式目录匹配 app 时固定限定该 app。name 默认兼作稳定键，显式 id 允许重命名；旧 UUID 与旧配置入口保留兼容。

## 当前仓库

```text
packages/
  schema/              共享数据契约
  sdk/                 核心逻辑与 Lit UI
  mcp/                 独立 MCP 服务
  vite/                开发环境注入与项目自动配对插件
apps/
  playground/          SDK 集成开发与验证
  storybook/           UI 组件开发
  website/             单页产品官网，GitHub Pages 部署
pnpm-workspace.yaml    Workspace 配置与依赖版本 catalog
```

使用 `vp install` 安装依赖，`vp run dev` 启动 Playground，`vp run storybook` 启动组件开发环境；提交前运行 `vp run ready`。
`vp <命令>` 是工具链内置命令，`vp run <脚本>` 执行 workspace 脚本；新增功能应补对应测试。

---
> Source: [nightire/ainotation](https://github.com/nightire/ainotation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
