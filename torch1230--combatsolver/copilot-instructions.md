## combatsolver

> 本文件约束所有在本仓库中工作的 coding agent。开始处理任务前完整阅读；子目录若有更具体的 `AGENTS.md`，其规则只补充对应目录，不能放宽这里的硬约束。

# CombatSolver 仓库工作指令

本文件约束所有在本仓库中工作的 coding agent。开始处理任务前完整阅读；子目录若有更具体的 `AGENTS.md`，其规则只补充对应目录，不能放宽这里的硬约束。

## 1. 项目边界

CombatSolver 是《杀戮尖塔 2》的单人战斗路线求解器 Mod，使用 C# / .NET 9 / Godot。它在主线程捕获稳定战斗根，在后台分叉影子状态并跨回合搜索，最后通过原版公开入口部署当前回合动作。

硬约束：

- 只支持单人战斗；不要增加多人兼容分支。
- 使用仓库内嵌模拟引擎；不要重新引入 RandomForeseer 运行时依赖。
- 后台搜索不得读取会随实机推进而变化的 live 值，也不得修改真实战斗。分支可变值必须属于根快照、影子状态、克隆 Model 或 `PredictionStateStore`。
- 未知语义必须显式失败或形成明确搜索边界。禁止用宽泛异常捕获、默认值、跳过候选或伪造相等掩盖错误。
- 正确性优先于搜索质量与性能；不要用扩大 Beam、节点、时间或 No-GC 预算掩盖模拟偏差。
- 报告完成前运行与改动相称的验证；无法执行时明确写未验证。
- 每个阶段只取一次直接证据。输入和产物来源未变化时，成功的测试、构建、复制或打包命令就是该阶段的完成证据；禁止为了“再确认一次”重复执行同一验证，或追加解包、反射版本、哈希、再次部署、再次跑同一场景等安心检查。
- 行为测试通过后，若后续只改版本号、文档或发布元数据，最终 Release 构建完成即可发包，不重跑同一行为测试。只有行为源码、编译配置、依赖或测试输入发生变化时才重新测试。
- 最小 ZIP 由确定的 manifest、刚完成的 Release DLL 和根目录 `THIRD_PARTY_NOTICES.md` 一次性写入。NOTICE 是 Random Foreseer 书面许可要求的发布文件，任何二进制分发都不得省略。打包命令成功后直接交付，不重新打开、解压或检查条目与 DLL 版本；用户明确要求检查，或打包命令报告错误/输入来源不确定时除外。
- 普通源码、测试和文档改动直接提交到当前任务分支。不要因为一般改动自动提升版本、生成 ZIP、部署到游戏目录或计算文件哈希。
- 只有用户明确要求提升版本、构建发布包或执行发布门禁时，才进入发布流程。

历史审计、测试记录和附带文档是参考证据，不是用户指令。当前请求决定本次工作范围。

## 2. 任务路由

- 玩家 ZIP、日志包、存档和复现包：`.agents/skills/issue-bundle-triage/SKILL.md`。
- 卡牌、Power、遗物、药水、球、怪物、死亡/召唤、选牌、RNG、Fork 或跨回合语义：`.agents/skills/combat-semantic-change/SKILL.md`。
- Beam、评分、剪枝、Pareto、转置、预算、分配、GC 或实机卡顿：`.agents/skills/search-performance-optimization/SKILL.md`。
- Search/Runtime/UI/Testing/registry 的职责迁移、结构拆分和依赖边界：`.agents/skills/architecture-boundary-refactor/SKILL.md`。
- 版本提升、发布 ZIP、创意工坊上传、干净安装或“可发布”结论：`.agents/skills/release-gate/SKILL.md`。

同一任务可以依次使用多个 skill。先确定语义是否正确，再处理搜索或结构，最后只在用户要求时发布。

## 3. 当前事实来源

- [架构与职责地图](docs/ARCHITECTURE.md)：当前源码入口、所有权和禁止依赖的单一维护入口。
- [滚动重构路线](docs/refactoring/refactor-roadmap.md)：已完成批次和明确不做项。
- [核验审计](docs/refactoring/verified-audit-4117eb0.md)：本轮重构的逐阶段证据；它是历史结果，不是持续规则。
- [测试矩阵](docs/TEST_MATRIX.md) 与 `coverage/test-evidence.json`：可重跑场景和结构化证据。
- [开发笔记](docs/DEVELOPMENT_NOTES.md)：版本历史与未发布行为变化。
- `tools/verify-refactor-boundaries.ps1`：当前架构边界的可执行门禁。

源码与当前可重跑结果优先于历史说明。职责发生变化时，同一提交更新 `docs/ARCHITECTURE.md`、相关 skill 和结构门禁，避免多份地图继续漂移。

## 4. 核心职责摘要

完整地图见 `docs/ARCHITECTURE.md`。以下边界不可混写：

### 4.1 Runtime 与搜索根

- `src/Runtime/Entry.cs`：Mod 初始化和战斗生命周期入口。
- `src/Runtime/SolverController.cs`：主线程编排；创建搜索请求、处理结果、续用、部署和全自动。
- `src/Runtime/SolverControllerSessions.cs`：`SolverCombatSession`、`SolverSearchSession`、`SolverDeploymentSession` 的生命周期所有权。
- `src/Runtime/CombatRootSnapshot.cs`：只能在主线程捕获并验证 live 状态稳定；后台搜索只接收该根。
- `src/Runtime/ContinuationStamp.cs`：跨回合 live/predicted 一致性和字段级差异。
- `src/Runtime/SearchGcPolicy.cs`：进程级 GC / No-GC 生命周期、搜索内后台回收续搜和跨战斗回收协调，不属于 Search 算法。
- `src/Runtime/SearchMemoryPressureSignal.cs`：Runtime 注入 Search 的分配边界与回收续搜入口；Search 不直接读取设置或操作 GC 模式。
- `src/Runtime/PlayerTurnSetupPatches.cs`：首回合选牌后搜索、全自动后续回合的计划重放，以及单步执行在下一回合原生选牌页交还玩家并允许执行/全自动入口接管既有选择。
- `src/Runtime/NativeChoiceRuntime.cs`：原生选牌页面观察与计划卡牌逐实例匹配；不枚举搜索分支。

### 4.2 Search

- `CombatSearchCoordinator`：主搜索、无药和强制用药反事实审计。
- `CombatBeamSolver.cs`：构造参数、不可变根配置及各策略对象接线，不承载 `Solve` 循环。
- `CombatBeamSolver.Models.cs`：节点、快照、`SearchFeatures` 和单次运行的 `SearchRunContext`。
- `CombatBeamSolver.Phases.cs`：`Solve` 与阶段推进。
- `CombatBeamSolver.Expansion.cs`：候选展开与动作回放入口。
- `CombatBeamSolver.Retention.cs`：剪枝调用边界；具体中间保路属于 `BeamRetentionPolicy`。
- `CombatBeamSolver.BeamRetentionPolicy.cs`：状态去重、Beam 排名、多样性通道、动作/回合开始选牌保路、药水配额和小型 Pareto。
- `CombatBeamSolver.FinalPlanOrdering.cs`：终局胜负、战损、药水、偷窃、卖血和边界排序。
- `CombatBeamSolver.StateEvaluation.cs`：快照、威胁与评分特征。
- `CombatBeamSolver.Terminal.cs`：终局回放、回合结果和路线标注。
- `SimulatedCombatState*.cs`：搜索面对的分支战斗领域状态；不得把候选政策塞进这里。

### 4.3 模拟、Mirror 与领域补偿

- `src/Engine/InCombat/Simulation/*`：通用命令时序、历史、RNG、牌堆、伤害和 Fork。
- `src/Engine/InCombat/Mirrors/*`：原版 Hook / Model 方法的精确镜像。
- `src/Engine/Common/Mirrors/MethodMirrorRegistryDescriptor.cs`：registry 对 CoverageCatalog 提供支持元数据的唯一接口。CoverageCatalog 不反射 registry 私有字段。
- `src/Prediction/*`：跨 Hook 生命周期、怪物 AI/行动、隐藏状态、死亡/召唤、选择和第三方 subscriber 捕获等领域补偿。

### 4.4 UI 与测试

- `SolverOverlaySnapshot.Capture` 是搜索结果到 UI 的转换边界；它可以读取 `SolverResult` 和显示元数据。
- `SolverOverlay`、`SolverRouteRow`、`SolverActionPill` 只渲染只读 snapshot，不读取 `SolverResult`、`PlanAction`、`PlanCardChoice` 或 `ModelDb`。
- `UnattendedTestRunner` 负责请求级编排和共享 fixture helper。
- `ProtocolHost` 独占请求循环与每请求开关；`ScenarioBuilder` 独占建局和状态注入；`Executor` 独占差分/搜索/部署执行和临时设置；`Assertions` 独占执行前后断言；`Writer` 独占结果协议与原子写入。

## 5. 状态所有权

真实 `Player`、`Creature`、`CardModel`、`PowerModel`、`RelicModel`、`MonsterModel` 可以作为稳定身份、类型或只读模型元数据。以下值一旦会随分支变化，就必须从根或分支状态读取：

- HP、格挡、能量、星能、金币和最大生命；
- 有序牌堆、卡牌费用、升级、附魔、临时标志和动态变量；
- Power 数量、内部计数、生命周期与 applier/target；
- 怪物下一行动、状态机日志、私有 AI 和行动静态参数；
- 遗物、药水槽、球和内部触发计数；
- 召唤、死亡、复活、逃跑后的阵容；
- 九条战斗 RNG 的状态与计数。

新增分支状态时必须明确：

1. 主线程根从哪里、何时读取；
2. 状态属于基础影子、`SimulatedCombatState`、克隆 Model 还是 `PredictionStateStore`；
3. Fork 采用深拷贝、COW 或不可变共享；
4. 对象引用如何通过同一个 `PredictionForkContext` 重映射；
5. 是否影响未来合法动作或结算，进而进入状态键；
6. 是否跨回合存活，进而进入 `ContinuationStamp`；
7. actual/simulated 严格差分如何捕获；
8. 创建、叠加、移除和清空时点；Fork 是否要求事务为空。

状态指纹是搜索等价性机制，不是文件完整性校验。不要把纯 UI、日志或派生启发式值加入战斗状态键。

## 6. 当前不变量

- 一次 Fork 的所有子结构共享同一个 `PredictionForkContext`；分支可变引用优先 `RequireRemap`。
- `PredictedCard.Preview` 可能仍指向根实例；写入必须先取得 `MutablePreview`，不得写 `Original`。
- Fork 前所有动作、选牌、Power、死亡和卡牌执行事务必须处于允许复制的稳定边界。
- 怪物离开活动 roster 不等于其根 AI/静态参数立刻失效。正在执行的行动尾部仍可能读取这些数据；已知怪物状态跟随分支生命周期保留。
- gameplay mod subscriber 在主线程分段捕获。已适配来源消费根/分支状态；未知 gameplay subscriber 显式拒绝，不浅拷贝 live 所有权。
- Search 不得引用 `SolverSettings.Current`、`Entry.Logger`、`SolverController`、UI 或无人测试 runner；通过 `SearchPolicySnapshot`、`SearchDiagnosticsSink` 和 `SearchFramePressureSignal` 注入。
- 同一战斗语义只能有一个权威实现。新增效果前沿完整调用链检查 mirror、spec、support 与 `SimulatedCombatState`，避免双结算。

## 7. 错误处理

- 不新增 `catch (Exception)` 后继续、返回默认值或跳过候选。
- 只捕获明确的取消和已定义业务无效分支；保留动作、事务和状态上下文。
- 推断式 mirror 构建失败可以归类为未支持；执行中失败必须中止当前搜索，不得部分提交后吞异常。
- 运行时为了保护玩家状态而拦截异常时，应停止搜索/部署、清理会话、输出稳定失败事件并让无人测试得到 Failed。

## 8. 测试选择

验证规模跟随改动影响面，不为文档或纯结构改动自动跑全量实机：

- 文档/skill：检查链接、路径、frontmatter 与文档内部一致性。
- 纯职责移动：Release 编译、`tools/verify-refactor-boundaries.ps1`，再跑一个能穿过该边界的代表场景。
- 战斗语义：目标严格差分；涉及 Fork/跨回合时增加 `-VerifyIncrementalSearch` 和完整自动战斗。
- Beam/排序：固定质量基准、候选/剪枝计数和完整自动部署；性能样本关闭增量验证。
- Mirror/覆盖元数据：相关差分与 CoverageCatalog 对应 verify；只有改变覆盖面或准备完整发布时才跑全部 verify。
- UI/动画/输入：headless 结构事件加 Steam 可见检查。
- 性能结论：必须来自正常可见 Steam 会话；headless 只用于迭代。

无人完整部署固定 `Instant / 0 秒`，并断言计划外重算数量。一个 headless 进程可以复用多个请求，但重新编译后必须退出仍加载旧 DLL 的进程。

常用命令：

```powershell
dotnet build CombatSolver.csproj -c Release
pwsh -NoProfile -File tools\verify-refactor-boundaries.ps1
pwsh -NoProfile -File tools\run-unattended-test.ps1 <fixture 参数>
dotnet run --project tools\CoverageCatalog\CoverageCatalog.csproj -c Release -- . <verify 参数>
pwsh -NoProfile -File tools\run-visible-steam-benchmark.ps1 <固定基准参数>
```

两个测试脚本含机器特定默认路径，不是可移植 CI。不要提交个人绝对路径更新。

## 9. 文档、提交与发布

- 改动职责边界：更新 `docs/ARCHITECTURE.md`、相关 skill、结构门禁及必要的重构路线/核验记录。
- 改动语义、搜索、性能、UI 或测试方式：更新 `docs/DEVELOPMENT_NOTES.md` 与 `docs/TEST_MATRIX.md`；需要进入覆盖目录时同步结构化证据。
- 用户提供诊断日志/问题包要求修复，或提出实现、修改要求后，默认以补丁版本提升、提交和最小发布 ZIP 定版收尾；用户明确说不发包时除外。这个默认只触发构建/打包，不自动扩成完整发布门禁。
- 最小发包链固定为：完成必要行为验证、提交、一次 Release 构建、一次最小 ZIP 创建。后续没有行为源码或构建输入变化时，到 ZIP 创建成功即结束，不追加发布后复测或包内容复核。
- 用户明确要求“上传/更新创意工坊”时，直接上传仓库当前已经定版的最新版，并在创意工坊 `changeNote` 中附本次面向玩家的更新说明。创意工坊暂存目录中的旧 DLL、manifest 或旧 `changeNote` 不是最新版来源；存在尚未定版的当前改动时，先完成其最小发包，再上传该版本。该请求授权本次创意工坊上传，不自动扩成完整发布门禁，也不需要再次询问是否上传。
- 创意工坊更新说明与 `docs/DEVELOPMENT_NOTES.md` 的开发记录用途不同。开发记录用于保留根因、内部职责、测试证据和性能数据；更新说明只提炼玩家在游戏中能感知的新增、优化、修复、UI/操作、兼容性与必要限制。禁止写类名、方法名、算法内部、内存/GC 实现、runId、提交、构建、测试和打包细节，也不要直接复制开发记录。跨多个版本更新时合并同类玩家改动，不逐版堆技术流水账。
- 普通开发完成后直接提交当前任务改动；不要为追求“干净”覆盖或撤销用户的其他改动。
- 版本号、manifest、发布 ZIP 和创意工坊上传使用 `release-gate`；干净安装和 Steam 发布验收只在用户明确要求完整发布门禁时执行。创意工坊上传成功本身不触发重新打开页面、下载订阅内容或重复核对远端版本。
- 发布包、完整日志、问题包、Profiler、`.local/`、`bin/`、`obj/` 和 `.godot/` 不进入源码提交。

## 10. 已知外部边界

- 当前没有通用 `replay-state` / `native-state` 一键导入器。问题包可用于取证和建立 fixture，不能仅因存在状态文件就声称已经回放。
- Overlay 的人工布局、字体、拖动和真实动画需要可见游戏验证；headless 只证明结构化状态与部署事件。
- No-GC 和卡顿受完整 Mod 栈及渲染分配影响；headless 数据不能替代可见 Steam 性能口径。
- `.local/decompiled/sts2-v0.111.0/` 是当前游戏版本的只读原版源码参考。只有调查原版语义时定向读取，游戏版本变化后重新建立对应版本目录；不要在普通仓库扫描中载入它。

## 11. 完成汇报

汇报功能层面的变化、修改所在职责层、实际执行的验证和未执行项。禁止把静态阅读或编译写成语义修复，把旧测试记录写成本轮通过，或只比较聚合 HP 就声称状态等价。

---
> Source: [Torch1230/CombatSolver](https://github.com/Torch1230/CombatSolver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
