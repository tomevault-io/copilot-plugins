## ma

> **`easysdd/architecture/DESIGN.md` 是整个项目的架构权威文档。**

# March — 项目规则

## 架构中心

**`easysdd/architecture/DESIGN.md` 是整个项目的架构权威文档。**

- 所有架构决策、模块划分、数据结构设计，必须能从 `easysdd/architecture/DESIGN.md` 的核心理念中追溯出来
- 子系统设计（file watcher、context builder、TUI 等）是 `easysdd/architecture/DESIGN.md` 思路的延伸，不是独立存在的
- 新增功能或改动方向前，先对照 `easysdd/architecture/DESIGN.md`，确认与核心设计一致；若有冲突，先更新 `easysdd/architecture/DESIGN.md`，再动代码
- 子系统若有独立架构文档，放在 `easysdd/architecture/` 目录下，并在 `easysdd/architecture/DESIGN.md` 中引用
- 不要在代码注释或其他文档里另起炉灶地描述架构——架构讨论的落点是 `easysdd/architecture/`
- 注意区分：`easysdd/architecture/*.md` 是**项目级架构权威**（长期存在，跨 feature 共享）；`easysdd/features/{name}/design.md` 是**单个 feature 的方案 doc**（由 easysdd-feature 工作流产出，和同目录下的 `prd.md` / `acceptance.md` 聚合在一个 feature 目录，生命周期跟随这个 feature）。两者不是同一类文档，不要混
- 代码实现应保持较丰富的注释，优先解释设计意图、关键流程和不直观的实现细节。

## UI 约定

- 按钮类功能在不影响可理解性的前提下，优先使用 Icon 实现，避免用文字按钮破坏界面密度与整体观感

### UI 克制原则

**界面默认要克制，不要往用户脸上堆说明文字。** 具体落到几件事：

- 页面 / 区块的标题下面**不要默认配一段解释性小字**（"这里放的是……"、"xx 用来管理……"）。标题自己够说明白就不写；一定要说明的，优先用空状态文案、tooltip、或文档承接，不要塞在永久可见位置
- 已有配置项只渲染"能让用户认出这一项是谁"的最少信息，不要把协议 / URL / Key / 附加元数据全部铺在列表上。详情留给编辑态
- 列表项的右侧操作区优先用"整行 hover + 点击进入编辑"代替显式"编辑"按钮；只保留破坏性或次级动作（删除、复制等）
- 新增一个永久可见的文字块、badge、hint 前先问：没有它用户会不会真的用不明白？如果不会，就不加
- 同一页面里不要出现两段语义高度重叠的描述（比如面包屑级标题旁 + 侧栏顶部各写一句介绍）

### 视觉 Token 三层体系

样式走 **primitive → semantic → component** 三层 token，详见 [easysdd/architecture/design-system.md](easysdd/architecture/design-system.md)。写样式前先对照此文档，硬性规则：

1. 组件 / utility class 里**不准写** hex、rgb、rgba、`#xxx` 颜色字面量，也不准裸写除 `0` 以外的 px 间距
2. 消费端（Vue 组件、Tailwind class、`@utility`）只能引用 **semantic** 或 **component** 层 token，不能直接用 primitive（`--color-gray-*`、`--color-error` 这类原子色）
3. 主题切换只重写 semantic 层；看到硬编码深色/浅色值（如 `rgba(20,20,19,0.94)`、`border-white/8`）就是 bug，要改成 semantic token
4. component 层 token 只在"semantic 组合不出来"时才新增，不要随手造

## 代码质量

- 前后端代码都要持续关注可维护性，避免把过多视图、状态、交互逻辑、业务流程、数据转换或基础设施细节堆进单个文件
- 当单个前端 Vue SFC / 模块，或后端 Rust 模块开始同时承担壳层、业务状态、弹窗流程、应用服务编排、provider 适配、存储读写、数据转换等多种职责时，应主动按职责边界拆分为组件、composable 或独立模块
- 拆分时优先按职责边界组织代码，避免只为”切文件”而切文件；拆出来的模块命名要清晰，接口要收敛

### 数据流拆分

- 设计前后端状态流时，先按业务事实和用户可感知的交互边界拆分数据源，不要为了省事把多个区域绑到同一个粗粒度 snapshot / store / view model 上
- 聊天区、右栏上下文、Debug、任务列表、输入框草稿这类状态默认应各有自己的数据源和更新节奏；只有在“首次加载 / 切换对象 / 显式 resync”这类语义明确的场景下，才允许共享基线快照
- 能增量更新的 UI，就不要用整块回填；能字段级 merge 的状态，就不要整对象替换
- 任何会同时影响多个面板的数据结构，在落地前都要先问一句：这是不是业务上真的同一份状态，还是只是实现上偷懒耦合在一起
- 如果某个模块既消费运行时事件，又消费持久化 snapshot，要优先防止双写、重复渲染和“一个区域更新导致另一个区域重建”的问题
- 对“后端完整 snapshot + 前端 runtime 增量事件”混合驱动的对象，合并时要先明确哪些字段允许被显式清空，哪些字段只是“这次未返回”。像 `runtime`、`debug_trace` 这类右栏/调试面板依赖字段，若本次 payload 未提供，就必须保留旧值，不能被 spread 默默覆盖成 `undefined`
- 不要同时维护“source state”和“派生 view state”两份同构缓存；优先只保留 source of truth，再用 computed / selector 派生视图。若确实要缓存派生结果，必须说明它和 source 的更新边界不同在哪里，以及为什么不能现算
- runtime 事件如果可能早于首次 hydrate / snapshot 到达，不能静默丢弃；至少要记录可检索的 debug 日志，必要时提供待补 hydration 的暂存或回放机制
- 多个事件分支有相同的“判空/忽略/取当前对象/合并回写”前缀时，不要靠额外花括号或复制粘贴继续扩张；应及时抽成小的共享 helper，保证状态机入口一致

### 实现功能时的动态重构

**实现任何功能前，先读涉及文件，识别重构机会，再动手写新代码。** 具体做法：

1. **边实现边识别**：在阅读待修改文件时，若发现以下信号，立即记录并纳入本次改动范围：
   - 函数/方法职责超过一件事（命名需要”And”或注释才能说清楚）
   - 重复逻辑出现两次及以上
   - 模块已承担多种职责，新功能会进一步加重
   - 参数列表过长（>4个），或数据在多层之间原样透传
   - 魔法字符串/数字未具名

2. **重构与功能同步交付**：不要把重构推迟到”以后”。若重构范围合理（不超出本次功能的影响面），直接在同一次修改中完成；若范围过大，先拆出最小必要的重构让新功能落地干净，其余作为后续任务明确提出。

3. **重构优先于功能堆叠**：若不重构就无法让新功能结构清晰，则重构是前置条件，不是可选项。

4. **不做无关重构**：重构范围限定在本次功能的直接触及路径上，不扩散到无关模块。

### 流式输出与进程收尾

- 长时间运行的命令流若会持续推送 stdout/stderr，UI 协议和命名必须明确这是“完整快照”还是“增量片段”；若传的是截至当前的完整内容，命名、注释和订阅侧处理都必须按 snapshot 语义设计，避免被误当 delta 重复拼接
- 流式输出节流不能建立在“每次都对完整累积 buffer 重新 decode / strip / transform”之上；只要输出可能到 MB 级，就必须采用增量处理、游标缓存或等价方案，避免 O(N²)
- 子进程取消、超时、正常结束三条路径的 reader 收尾流程必须共用同一套函数，避免在多个 `select` 分支里重复 drain / wait / join / drain；一旦某个分支需要“多补一遍”才能安心，说明生命周期设计还没收敛
- 不要用“24 小时 sleep”这类魔法 sentinel 表达“当前没有活跃计时器”而不写注释；优先用显式的 `Option<Timer>` / 状态位表达意图。若因框架限制只能用 sentinel，必须在代码旁说明为什么
- 任何外部终止动作（如 Windows `taskkill /T /F`）都要设超时，不能让 cancellation path 反过来卡住主流程

---
> Source: [liuzhengdongfortest/MA](https://github.com/liuzhengdongfortest/MA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
