## vrplayer

> 本仓库采用 AI 协作开发（Codex / Cursor）。


# AGENTS.md — vrplayer Agent Constitution (HARD RULES)

本仓库采用 AI 协作开发（Codex / Cursor）。
以下规则为强约束，默认始终生效。

---

## 0) 协作契约（HARD）

- 用户职责：只执行 Cursor 与 PowerShell 发布命令。
- Agent 职责：给出可执行、可落地、确定性的实现与发布结果。

Agent MUST NOT:
- 要求用户打开 DevTools / Console 进行调试。
- 要求用户手工修改任何配置文件。
- 使用“try / maybe / 看看是否可行”这类不确定表达。
- 提供多选方案让用户自行排雷。
- 在未确认发布完成前要求用户线上验收。

Agent MUST:
- 明确修改文件与预期结果。
- 先实现、构建、发布、确认部署，再让用户做线上目视验证。

---

## 1) 仓库事实（Ground Truth）

- 工作目录：`D:\Projects\vrplayer`（真实 git 仓库）。
- 分支：`main`。
- 托管：Cloudflare Pages / GitHub Pages 以 `docs/` 为发布目录。
- 构建产物：`dist/`（Vite）。
- 结论：发布链路必须是 `dist -> docs -> commit -> push`。

---

## 2) 配置单一事实源（HARD）

- 唯一可编辑配置源：`public/config.json`。
- 禁止手改：`dist/config.json`、`docs/config.json`。

配置流：
- `public/config.json` -> `npm run build` -> `dist/config.json` -> `robocopy` -> `docs/config.json` -> 线上 `/config.json`

---

## 3) 允许与禁止修改范围（HARD）

允许修改：
- `src/**`
- `public/**`（含 `public/config.json`）
- 根目录工程文档（如 `README.md`、`AGENTS.md`）

禁止手工修改：
- `dist/**`
- `docs/**`

说明：`docs/**` 只能通过 `dist -> docs` 同步生成。

---

## 4) 发布 SOP（唯一正确流程，HARD）

环境要求：
- Windows + 系统 PowerShell（非 VSCode 内置终端优先）。

步骤：
1. `cd D:\Projects\vrplayer`
2. `git checkout main`，确认非 detached HEAD
3. `git pull --rebase origin main`
4. `npm run build`
5. `robocopy .\dist .\docs /MIR`
6. `git add -A`
7. `git commit -m "<提交信息>"`
8. `git push origin main`
9. 确认远端最新 commit 为刚推送 commit，且 Pages 已部署到该 commit
10. 仅此之后才允许用户做线上验证

若用户反馈“线上未变化”，第一假设必须是发布链路问题：
- 漏了 robocopy / 无新 commit / push 失败 / Pages 未部署到该 commit / 缓存命中旧资源

---

## 5) Git 与网络约束（HARD）

- Git/SSH 操作使用系统 PowerShell。
- SSH 走 `ssh.github.com:443`（兼容受限网络环境）。
- 非必要不做破坏性 git 操作；遇到冲突先保证可追溯与可恢复。

---

## 6) 构建产物冲突规则（docs/assets）

- `docs/assets/*` 是构建产物，不是源码。
- 冲突时禁止手工逐文件合并构建产物。
- 正确做法：以源码为准，重新构建并同步 `dist -> docs`。

---

## 7) 默认架构原则（静默执行）

无需在回复里长篇论证，默认按以下原则实现：

### A) SOLID
- SRP：单一职责，单一主要变更原因。
- OCP：优先扩展而非破坏既有核心行为。
- LSP：替换子类型不改变外部可观察行为。
- ISP：接口小而专，避免胖接口。
- DIP：在降低耦合时依赖抽象而非具体实现。

### B) 高内聚、低耦合
- 变更局部化，控制影响面。
- 避免循环依赖与新增全局状态捷径。
- 保持单一真相源，避免并行“第二套规则”。
- 仅在“小且安全”前提下做必要重构。

### C) 输出与沟通
- 优先直接给最终可执行改动。
- 非明确要求时，不输出冗长自检报告。
- 若规则冲突或仓库现实不确定：先停下并提问，不猜。

---

## 8) 坐标系统宪法（HARD）

定义：
- `yaw / pitch / northYaw`（配置与 URL）均为世界角度（world）。
- three.js 内部使用 internal 角度。

规则：
- world -> internal 转换只允许在单一入口执行一次。
- 唯一允许形式：`internalYaw = -worldYaw`。

禁止：
- 多处重复取反。
- 组件内私有“再反一次”修补。
- fallback 分支里再做符号补偿。

---

## 9) 罗盘系统三实现同步（HARD）

方向相关改动必须同步评估以下三处：
1. `src/ui/CompassDisk.ts`
2. `src/ui/GroundHeadingMarker.ts`
3. `src/viewer/NadirPatch.ts`

禁止“先改一处试试看”。

---

## 10) 罗盘产品决策（锁定）

- `initialView.yaw`：初始视角方向（world）。
- `northYaw`：真实北向（world）。
- Pointer 表示当前相机朝向。
- Dial 与 Pointer 职责分离，不可混用。

验收不变量：当 `cameraYaw == northYaw` 时，必须同时满足：
- pointer angle == 0
- pointer 指向盘面 N
- 初始朝向与配置北向一致

---

## 11) North Calibration（如功能存在）

- 仅用于配置辅助：读取当前 yaw -> 产出 northYaw。
- 禁止引入运行时隐式自动校正或隐藏状态变异。

---

## 12) Pick 规则（HARD）

- Pick 的 yaw/pitch 必须基于 `Raycaster.ray.direction` 计算。
- 禁止依赖 intersection point 作为唯一来源。

---

## 13) MCP / Skills 验证要求（HARD）

- 调试与验收优先使用 MCP/skills，不允许只凭代码推断。
- 任何“修复成立”结论前，至少执行一次 `chrome-devtools MCP` 的以下任一组合：
  - 页面快照 + Network 采样
  - 页面快照 + Console 采样

---

## 14) README 持久化知识（REQUIRED）

当发现新的关键坑、约束或 SOP 变化时：
- 必须更新 `README.md` 的 `Agent Notes (Persistent)`。
- 内容保持简短、可检索、可执行。

---

## 15) 输出语言与风格

- 输出、总结、提交信息默认使用简体中文。
- 未被明确要求时不切换英文。
- 默认采用“最小、安全、可验证”的改动策略；非必要不做大范围重构。

---

## 16) 日志记录规范（HARD）

- 新增或更新任何日志、工作记录、发布记录时，必须写明完整日期和时间。
- 时间格式统一为：`YYYY-MM-DD HH:mm:ss`（24 小时制）。
- 禁止仅写“今天 / 刚刚 / 下午”等相对时间描述。

---

## 17) 工具概览
MCP 工具使用约定
MCP 全量使用表
MCP/工具	何时使用（只在这种情况）	不该用的时候
shell_command	读取/执行命令、构建、测试、git、发布流程	能直接用已有结果回答时
filesystem	结构化读写文件、批量查改、安全编辑	只需一两行命令可完成时
github	查 PR/Issue/远端仓库状态、自动化仓库操作	纯本地开发阶段
chrome-devtools	页面快照、Console/Network采样、性能证据	纯后端/纯脚本任务
playwright	需要自动化真实用户流程回放	只需一次静态页面采样时
memory	存关键决策/坑点/偏好/发布结果，跨会话复用	存临时噪音日志
deepwiki	快速理解仓库架构、模块关系	已有本地代码足够且问题很小
repomix	里程碑时打包代码上下文、做全局索引	日常小改动
context7	查库/API官方文档与配置写法	不涉及第三方库知识
openaiDeveloperDocs	OpenAI API/SDK/Apps/Codex 相关问题	非 OpenAI 技术问题
tavily	需要最新公开信息或站外研究	可从本地和官方文档解决
exa	代码语义检索、公司/网页研究	本地仓库已有答案
fetch	精确抓取某个URL原文	不需要网页正文时
shadcn	组件库检索、示例代码、add命令	项目不使用 shadcn 时
sequentialthinking-tools	复杂多步决策、需要严密分解时	简单直改任务
list/read_mcp_resources	先探测 MCP 资源再读取	已知目标文件路径明确
update_plan	多步骤任务需要显式进度跟踪	单步小任务

Skills 全量使用表
Skill	何时使用（只在这种情况）
brainstorming	需求不清、需要先澄清目标与边界
cloudflare-deploy	仅当改动 Cloudflare 平台配置/部署架构
develop-web-game	仅做 web game 循环开发/渲染调试
dispatching-parallel-agents	可并行的独立子任务 >=2
executing-plans	已有计划文档，按计划执行
finishing-a-development-branch	开发分支收尾、准备合并
frontend-design	UI界面设计与视觉升级
interaction-design	动画、微交互、过渡与反馈设计
openai-docs	OpenAI 产品/API文档查询与落地
planning-with-files	复杂任务建 task_plan/findings/progress
planning-with-files-othmanadi	同上，二选一使用，不要双开
playwright	浏览器自动化操作与流程验证
receiving-code-review	接收并处理代码评审意见
requesting-code-review	任务完成后请求质量审查
screenshot	用户明确要求桌面/区域截图
security-best-practices	用户明确要求安全最佳实践审查
security-ownership-map	用户明确要求安全责任图/Bus factor 分析
security-threat-model	用户明确要求威胁建模
subagent-driven-development	需要子代理分工执行计划
systematic-debugging	所有 bug/failure 场景优先启用
test-driven-development	功能/修复需先测后改
ui-ux-pro-max	需要高强度 UI/UX 风格化输出
using-git-worktrees	需要隔离工作树并行开发
using-superpowers	会话起始需要 superpowers 流程时
vercel-react-best-practices	React/Next 性能与模式优化
verification-before-completion	声称完成前的证据化验证（必用）
web-design-guidelines	UI/可访问性/设计规范审查
writing-plans	多步骤需求先写计划再动手
writing-skills	创建或修改 skill 本身
skill-creator	设计新 skill 框架与内容
skill-installer	安装/管理 skills
任务类型速查表（你要的“项目理解/性能/文档查询”等）
任务类型	首选 MCP	首选 Skills
项目理解	shell_command + filesystem + deepwiki	writing-plans（复杂时）
代码实现	shell_command + filesystem	test-driven-development
Bug 调试	chrome-devtools + shell_command	systematic-debugging
代码质量与性能	chrome-devtools + shell_command	vercel-react-best-practices + verification-before-completion
UI 视觉优化	chrome-devtools + playwright	frontend-design + interaction-design + ui-ux-pro-max
文档查询	context7 / openaiDeveloperDocs / deepwiki	openai-docs
发布上线	shell_command + github	cloudflare-deploy（仅平台改动）
经验沉淀	memory	无（直接写记忆）

// ## 请忽略这条：未来新功能开发完成正式上线后，实现这个功能：当用户空闲无操作时，自动缓存高清全景原图jpg，当用户下一次再进入已经加载过高清全图的场景时，直接加载缓存的全图，而不是ktx块图

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wokao4360-rgb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wokao4360-rgb)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
