## selfcam

> - 当用户提出“提交 git”“帮我提交”“发版提交”“类似提交 git 的话术”这类请求时，默认按以下顺序执行。

# selfCam 协作规则

## Git 提交流程

- 当用户提出“提交 git”“帮我提交”“发版提交”“类似提交 git 的话术”这类请求时，默认按以下顺序执行。
- 第一步：先检查文档是否需要同步更新，再决定是否提交。重点检查 `CHANGELOG.md`、`VERSION.md`、`docs/`、`PRDS/` 以及其他明显和当前改动相关的说明文档。
- 第二步：确认文档已同步或确认本次无需改文档后，再执行本地 git 提交。
- 第三步：本地 git 提交完成后，必须单独询问用户是否需要上传 GitHub；未得到用户明确确认前，不要主动 push，不要主动创建远程发布动作。
- 当下一步准备进入本地 git 提交时，必须先给出建议版本号、建议提交备注和简短理由，供用户确认；未得到用户明确确认前，不直接提交。
- 提交备注默认使用中文；如需保留 `feat`、`fix`、`chore`、scope、change id、spec id 等规范标识，可保留英文标识，但主体说明优先使用中文。

## 工作目录与 worktree 约束

- 默认只在 `D:\project\selfCam` 这一个工作目录内开发、提交、合并和验证。
- 不主动创建新的 git worktree，也不主动使用 `D:\project\selfCam-*` 这类旁路工作目录，除非用户明确要求。
- 如果当前分支被其他 worktree 占用，先向用户说明原因和可选方案，不为了方便自动新建或切换到其他 worktree。
- 如确需临时使用 worktree，必须先说明目录名、用途、清理方式，并在任务结束前询问是否删除。

## 执行细则

- 每次执行这类请求时，都要在回复里明确说明“已检查文档”或“文档已同步/本次无需同步”。
- 如果发现代码已改但文档未同步，优先先补文档，再提交本地 git。
- 如果文档是否需要修改存在歧义，先基于当前改动做最小范围检查；能直接补齐的就直接补齐，再继续提交。
- 如果当前改动涉及业务流程、页面交互、缓存结构、配置体系、技术架构或 UI 文案，检查文档时不能只看 `docs/`，还必须显式检查 `PRDS/`。
- `PRDS/` 中优先检查 `PRD.md`、`UI.md`、`tech.md`；如果当前改动已经影响这些文档描述，应在提交前同步补齐，不能遗漏。
- “上传 GitHub”包括但不限于 `git push`、创建远程分支、发起 PR；这些动作都需要在本地提交完成后再次征求用户确认。

## 测试与验证分级

- 默认只运行和本次改动直接相关的最小测试集，不因“更稳妥”而自动扩大到全量测试或微信开发者工具自动化。
- 小文案、小样式、小 UI 状态调整：
  - 优先运行语法检查或相关单测。
  - 不默认运行全量 Jest。
  - 不默认运行微信开发者工具 automator。
- 单页面交互、状态流小改：
  - 运行对应页面或模块的相关单测。
  - 仅当改动涉及微信原生 API、页面生命周期、相机组件、路由跳转或用户明确要求时，才运行 automator。
- 缓存结构、上传流程、接口映射、多车辆流程、跨页面流程：
  - 运行相关单测。
  - 必要时运行全量 Jest。
- 相机尺寸、特殊分辨率适配、取景框坐标、AI 自动拍照门控：
  - 运行相关 layout / AI / camera 单测。
  - 需要真机或微信开发者工具验证时，先说明原因和预估耗时；用户确认后再跑重型 automator。
- 发版、合并前或用户明确要求全面验证时，才运行全量 Jest、automator smoke 或更重的 e2e。
- 用户明确说“不跑测试”“直接提交/上传”时，不运行测试；最终回复必须明确说明本次未运行测试。
- 不把历史测试通过当成本次通过。凡是声称“通过”，必须是本轮实际执行过的命令结果。

## 常用验证命令选择

- JS 语法检查：`node --check <file>`
- 单个 Jest 文件：`npm test -- --runInBand __tests__/<file>.test.js`
- 全量单测：`npm test -- --runInBand`
- 微信开发者工具 smoke：`$env:WECHAT_DEVTOOLS_CLI='D:\environment\wechat-devtools\cli.bat'; npm run test:automator:smoke`

## 审查材料约定

- 仅当用户明确要求生成审查材料时，才生成 `review.diff` 和 `review-summary.md`。
- 用户未明确要求时，修改 `AGENTS.md`、全局配置、长提示词、系统/项目协作指令、Codex 工作流文档等“指令层内容”，不默认生成额外审查文件。
- 如需生成审查材料，`review.diff` 只包含本次指令或文档改动相关文件，不要混入业务改动。

## OpenSpec / Comet 使用规则

- 本项目已接入 OpenSpec 与 Comet：OpenSpec 根目录为 `openspec/`，Comet 项目配置为 `.comet/`，Codex 侧技能与规则位于 `.codex/`。
- 本项目的实施请求默认先由 Comet 根据任务性质路由：新增 capability、公共接口、schema、跨页面状态流或架构调整使用 `full`；不新增 capability 的明确既有行为修复使用 `hotfix`；文档、配置、prompt 和可收敛为单一 change 的轻中量修改使用 `tweak`。
- 用户明确指定 `comet`、`comet-hotfix`、`comet-tweak` 或要求不使用 Comet 时，以用户选择为准；只读检查、分析和方案输出不启动实施 workflow。
- 如果存在与当前请求匹配的 active change，默认恢复该 change；如果 active change 不匹配或同时存在多个候选，先让用户确认继续哪个 change 或创建新 change。
- 涉及新功能、业务流程、接口、配置体系、技术架构或跨页面状态流的改动，优先先创建或更新 OpenSpec change，再进入实现。
- 当前能力基线位于 `openspec/specs/`；`openspec/changes/archive/2026-07-10-baseline-existing-selfcam/` 保留现有成果物基线化过程、来源映射和历史验收场景。
- 需要理解现有业务规则时，先阅读 `openspec/specs/` 下的对应能力规格，再通过归档基线定位并回源阅读 `docs/` 或 `PRDS/` 下的原文件；不要把原文档全文迁移进 OpenSpec。
- 使用 Comet 时遵循阶段守卫、脏工作区检查和验证要求；不要绕过 Comet/OpenSpec 流程直接扩大改动范围。
- 接入 OpenSpec / Comet 不改变本项目既有安全红线：未经用户当前任务明确授权，不自动 commit、push、打 tag、创建或切换分支；不修改无关业务代码，不新增无关文档。
- 每次任务的最终汇总必须在最后一行说明本轮 Comet、OpenSpec、Superpowers 的使用情况；已使用时写明阶段或用途，未使用时明确写“未使用”。
- 凡是本轮涉及 Comet 工作流的对话，最终汇总必须在工具使用情况之前主动说明下一步建议处理内容；如果当前阶段已完成，应说明建议进入下一阶段、提交/推送、验证、归档、暂停观察或等待用户确认中的哪一种。
- 工具使用情况固定使用格式：`工具使用情况：Comet（...）；OpenSpec（...）；Superpowers（...）。`

### 命名与语言规则

- 项目内所有文字内容，在不影响代码标识符、协议字段、第三方专有名词、模板关键字和工具要求的前提下，能使用中文的都必须使用中文。
- 项目文档、需求说明、设计说明、测试说明、变更记录、版本说明和提交备注必须优先使用中文。
- OpenSpec 文档正文、标题、背景、设计说明、验收场景默认使用简体中文。
- OpenSpec 的 change id、spec id、目录名默认使用英文短名或 kebab-case，例如 `baseline-existing-selfcam`、`camera-capture`。
- OpenSpec 模板中的固定结构关键字保留英文，例如 `Requirement`、`Scenario`、`ADDED`、`MODIFIED`、`REMOVED`。
- Comet、OpenSpec 相关文档中的 tag、id、目录名、状态值、模板关键字和机器可读字段，按对应工具或模板要求选择合适语言，不为了中文化破坏工具兼容性。
- 不使用中文目录名作为 OpenSpec change/spec 标识，避免 CLI、脚本、Git、CI 或跨平台路径兼容问题。
- 现有 `docs/`、`PRDS/` 中文文档保留原路径，不为了接入 OpenSpec 强制改名或搬迁。
- OpenSpec 只做结构化索引、能力规格、验收标准提炼和变更归档。

---
> Source: [rylzx000/selfCam](https://github.com/rylzx000/selfCam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
