## 9reader

> Hoshi Reader Windows（HSW）是 Hoshi Reader 阅读体验的 Windows/Tauri 实现项目。目标是在 Windows 桌面平台完成稳定、可维护的阅读、词典查词与 Anki 工作流，并在明确需要时对齐 Hoshi Reader Android（HSA）的用户可见行为。

# Hoshi Reader Windows Agent 指南

Hoshi Reader Windows（HSW）是 Hoshi Reader 阅读体验的 Windows/Tauri 实现项目。目标是在 Windows 桌面平台完成稳定、可维护的阅读、词典查词与 Anki 工作流，并在明确需要时对齐 Hoshi Reader Android（HSA）的用户可见行为。

本文件只记录需要在未来会话中长期遵守的仓库级规则。当前实现状态、功能完成度、风险和任务进度不以本文件为准，必须回到当前代码和对应 `docs/` 文档核实。

## 工作原则

- `hoshi-reader-windows` 当前代码是“当前实现事实”的唯一真源：用于确认现有数据流、状态所有者、模块边界和兼容风险。它不自动定义目标用户体验；当任务涉及用户可见行为对齐时，应以 HSA 的已验证行为为主要参考，并结合 HSM 的桌面端行为与 Windows 平台约束决定最终方案。
- 当前 HSW 行为如与 HSA 的已验证核心体验冲突，不得仅因“现有代码如此实现”而保留；应先明确差异属于 bug、未完成迁移、合理的 Windows 平台适配，还是 HSW 有意形成的独立产品决策。
- 开始修改前，先检查相关 Svelte、TypeScript、Rust command、storage 和测试/probe。
- HSA 是需要对齐用户可见行为时的主要事实参考；HSM 仅作为补充参考。不要默认继承参考项目的产品定位、架构、平台 API、存储方案或实现细节。
- 需要对齐 HSA/HSM 时，先确认任务涉及的具体交互、状态流、边界行为和可观察结果，再决定 HSW 的最小 Windows/Tauri 实现；不要只复制代码表面结构。
- 修复跨层、状态相关、可重复出现或原因不明的问题时，先追踪状态来源、生命周期、调用链和失败边界，优先修正根因。对范围明确、无状态影响的局部样式、文案或静态渲染缺陷，可在确认影响面后直接做最小修复。
- 当任务引入或修改不熟悉、版本敏感、平台相关或安全边界相关的 Tauri、Rust、Svelte、Bits UI、Windows 文件系统、网络、打包或依赖 API 时，优先查官方文档或仓库已有用法。对已有稳定调用点的局部样式、文案、纯逻辑修复，先以当前代码和已有测试为准，不要求为此额外调研。
- 修改范围应贴合当前任务。除非用户明确要求完整功能切片，否则不要将 reader、storage、dictionary、Anki、settings、packaging 等无关区域混入同一改动。
- 主路径优先保证闭环：bookshelf -> import EPUB -> open reader -> select text -> lookup -> optional Anki export。
- 预计跨会话、跨模块、需要多个独立 commit，或会影响 Reader、storage、dictionary、Anki、sync 等高风险边界的功能，应先创建或更新 feature plan。
- 小型且边界明确的两三步改动，可直接在任务说明、issue 或 `docs/TODO.md` 中记录最小计划；不要为了形式单独创建长期路线文档。
- 在同一功能中优先按可验证的 slice 推进，但用户明确要求一次完成的紧密耦合改动可以作为一个整体交付。
- 不得把未核实的项目状态、参考项目行为或运行时结论表述为已验证事实。当这些结论会写入真源文档、交付说明、验证结果或影响后续决策时，应明确标记为 `unknown` 或 `not verified`。

## Reference Submodules

- 长期需要对照的上游参考项目通过 Git submodule 固定在 `reference/` 下：`reference/Hoshi-Reader-Android` 是 HSA 主参考，`reference/hoshi-reader-mac` 与 `reference/ttu/ebook-reader` 是补充参考。

- 新 clone 或缺少参考源码的 worktree 先从 HSW 根目录执行：

  ```text
  git submodule update --init --recursive
  ```

- 需要更新 HSA/HSM 对照版本时，在对应 submodule 内 fetch 并 checkout 目标 commit，再在 HSW 中提交更新后的 gitlink；不要只依赖本机未追踪 checkout、在线版本、过期摘录或臆测行为。

- 参考项目只作为用户可见行为对照来源，视为只读。不要修改 `reference/` 下的参考项目，也不要把它们的源码展开复制、构建产物、个人本地配置或其他临时参考目录提交到 HSW。

- 除 HSA/HSM/TTU 外，不要向 `reference/` 添加新的长期参考项目，除非任务明确扩展参考范围并同步更新 `.gitmodules`、`reference/README.md` 与本规则。

## 架构与领域边界

- 当前架构事实以 `docs/ARCHITECTURE.md` 和当前代码为准；未来重构方向以 `docs/ARCHITECTURE_REFACTORING.md` 为准。
- HSW 的基础分层是 Svelte 5/TypeScript/Vite frontend、Tauri 2 desktop shell、Rust native backend。用户可见功能应由明确的 frontend state、Tauri command surface 和 Rust storage/backend 边界组成。
- 改动 reader、storage、dictionary、Anki、audio、settings 或 Tauri command 前，先查对应当前架构边界；除非先记录缺口，否则不要绕过既有 command、storage、controller、probe 或 backend 边界。
- app-owned 数据、文件导入/删除、dictionary runtime、AnkiConnect、audio 和 reader layout 都是高风险边界。修改时优先保持 containment、旧数据兼容、失败可恢复、错误可行动。
- `third_party/hoshidicts` 是第三方构建输入。除非任务明确涉及依赖升级、桥接修复或上游同步，不要直接修改其源码来规避 HSW 的边界问题。
- Settings 只暴露用户确实需要配置的行为；不要为了调整内部常量提前扩张设置面板。

## 真源文档

- `docs/VALIDATION.md`：长期有效的验证命令、专项 probe、真实运行时验证前置条件和验证结果记录规则。
- `docs/ARCHITECTURE.md`：当前 HSW 架构事实、模块边界和数据流；它不是未来计划或任务日志。
- `docs/ARCHITECTURE_REFACTORING.md`：未来架构债务和重构方向；完成后的基线应移回 `docs/ARCHITECTURE.md` 或本文件。
- `docs/PROJECT_STATUS.md`：当前实现状态、not verified 项、known issues 和 risk areas。它不是产品路线图，也不能代替当前代码核实。
- `docs/TODO.md`：当前正在进行的 slice、紧接着一个可执行的 next slice 和 active blockers；完整长期路线不得堆进此文件。
- feature plan / roadmap 文档：记录某一功能的目标、非目标、前置条件、完整 slice 顺序、验收、验证和状态。涉及多 slice 工作时，它们是该功能路线真源。
- `docs/LOOKUP_ROADMAP.md`、`docs/REAL_LOOKUP_VALIDATION_PLAN.md`：lookup 与真实词典验证相关路线。
- `docs/ANKI_HSA_WINDOWS_ALIGNMENT.md`、`docs/ANKI_MEDIA_EXPORT_PLAN.md`、`docs/ANKI_AUDIO_SYNC_PLAN.md`：Anki 对齐、媒体和音频相关路线。
- `docs/READER_ENTRY_CHARACTERIZATION.md`、`docs/reader-layout-baseline.md`：reader 行为基线与已验证边界。
- `STATUS.md`：仅作为文档索引和当前优先级提示，不记录详细实现事实。
- `AGENTS.md`：只记录未来 agent 需要长期遵守的仓库级规则，不记录短期任务状态或会快速过期的实现结论。

只有任务实际改变某个文档所负责的真源内容时，才更新该文档。

## 经验沉淀

- 如果一次错误定位出未来会反复出现的正确做法，将最小、可执行、低歧义的规则写入正确的真源文档。
- 需要所有会话长期遵守的跨任务规则才写入 `AGENTS.md`。
- 验证步骤写入 `docs/VALIDATION.md`；当前架构事实写入 `docs/ARCHITECTURE.md`；未来重构方向写入 `docs/ARCHITECTURE_REFACTORING.md`；当前实现状态和风险写入 `docs/PROJECT_STATUS.md` 或对应 feature 文档；完整路线写入 feature plan；当前交接写入 `docs/TODO.md`。
- 不要把一次性调查日志、临时 workaround、命令输出、个人机器路径、任务进度或只对当前 bug 成立的细节写进 `AGENTS.md`。
- 沉淀规则前先检查是否已有等价规则；存在时更新原规则，避免重复和互相矛盾。

## 用户可见 UI

- 保持 HSW 当前 minimal desktop app 的视觉与交互范围，除非任务明确要求更大 redesign。
- 用户可见文案应沿用当前界面的语言和表达风格；不要在同一页面混入无关语言、内部术语或原始异常文本。
- 用户可见错误必须可理解且可行动。主 UI 应展示明确状态、下一步或恢复方式，而不是直接渲染 native exception、Rust debug text 或路径细节。
- 对话框、menu、Tabs、Select、Switch、Tooltip 和 popup 必须维持鼠标、键盘、Escape、focus restore 和窄窗口边界行为。
- 新增图标时优先使用既有图标方案或明确的品牌资产；不要用临时 Unicode、拼接字符或难以识别的自绘图标代替交互含义。
- 修改 reader、dictionary popup、TOC、Anki panel、dictionary management 或设置面板时，必须考虑窄窗口与宽窗口；不得只验证当前开发窗口尺寸。

## 验证与提交

- 声明实现完成前，按 `docs/VALIDATION.md` 选择并运行必要验证；修改已有专项 probe 覆盖的功能时，通用检查不能替代专项行为验证。
- 涉及真实字典、AnkiConnect、媒体、音频或 sync 的改动，只有在本机依赖和测试数据确实可用时才运行 runtime validation；不可用时如实记录 `not verified`，不要伪称完成。
- `npm run package` 属于打包验证，不是每次普通改动的默认检查；仅在用户要求打包、发版验证或任务明确涉及 packaging 时运行。

- Commit message 使用 Conventional Commits。

- 完成一个可独立交付的功能 slice 并通过必要验证后，自动创建一次仅包含本次任务相关文件的 commit；用户明确要求暂不提交、保留未提交改动或仅调查时除外。

- 不要将无关既有 worktree 改动、参考项目改动、构建产物、个人本地配置或 ignored test data 混入 commit。

- 小型、低风险、边界清楚的修复可在当前分支完成；较大功能、跨模块重构、高风险 reader/storage 改动应使用独立 `codex/` 分支或 worktree。

---
> Source: [Dr-TNineS/9Reader](https://github.com/Dr-TNineS/9Reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
