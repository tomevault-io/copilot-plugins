## rl-sac

> > **作用域**：本文件适用于 `/home/sun/phdproject/dqn/RL_sac/**`。仓库通用说明见 `../AGENTS.md`。若冲突，以本文件为准。

# RL_sac/AGENTS.md（Ubuntu 24.04 + ros2py310）

> **作用域**：本文件适用于 `/home/sun/phdproject/dqn/RL_sac/**`。仓库通用说明见 `../AGENTS.md`。若冲突，以本文件为准。

## 0. 总原则（必须遵守）

1) **先计划后动手**

   - 写/改任何文件前：必须先输出 3–7 步「实施计划」+「将改动的文件清单」+「风险点」+「验证方式」。
   - 输出计划后：默认等待你明确确认（例如“开始/实现/按计划执行”）。如你在当次任务中明确允许“自检后直接开始”，则在完成约定自检并通过后开始实际修改。
   - 未进入实施前，仅允许做非侵入式探索（读文件、搜索、运行不改代码的检查/测试）。
2) **小步提交与可回滚**

   - 每次改动尽量小、单一目的；避免“顺手重构/顺手格式化全仓库”。
   - 涉及大量文件或大范围行为变化：拆成多个可独立评审的步骤。
3) **最小高置信变更**

   - 修 bug：优先“加失败测试 → 修复 → 全绿”。
   - 重构：必须保证行为不变（见验收标准），并说明如何验证。
5) **称呼约定**

   - 每次回复默认以“`帅哥，`”开头（除非你明确要求不需要）。

5.1) **沟通口吻与标识符释义**

- 默认用“研究生汇报”口吻：先说清楚我对问题的理解、假设与不确定性，再给可执行步骤与验证方式；避免营销式或口号式表达。
- 文本中首次出现不直观标识符（函数/类/变量/参数/CLI 选项/文件名等）时，在其后用括号补一句短解释（不超过 1 句）；重复出现可省略。
- 示例：`epsilon`（ε-greedy 的探索率/随机动作概率）、`gamma`（折扣因子）、`rollout_agent(...)`（用当前策略在环境里采样轨迹/回合）、`--runs 20`（评测重复次数）。
  5.2) **回复语言（硬约束）**
- 默认仅使用中文回复（包含计划、过程更新、结果总结、风险说明与验证结论）。
- 仅当你明确要求英文或中英双语时，才允许切换输出语言。
- 命令行、文件路径、参数名、代码标识符可保留英文原文，不视为违反“仅中文回复”。
  5.3) **`AGENTS.md` 与 `CLAUDE.md` 同步（硬约束）**
- 两个文件必须保持**逐行一致**，不得出现长期漂移。
- 任一文件发生修改时，必须在同一轮次同步修改另一文件。
- 提交前必须执行 `diff -u AGENTS.md CLAUDE.md`；若存在差异，禁止结束该轮修改。

6) **可复现性（配置）**

   - 默认规则：每次完成任何**代码改动**（含重构/性能优化/数值变更）后，都要新增一份“可复现配置”，保证你本人可复现（你可在当次任务中明确豁免）。纯文档改动默认不强制。
   - 推荐做法：在 `configs/` 新增 `repro_YYYYMMDD_<topic>.json`，并在其中记录：
     - 复现实验/自检命令（推荐 `conda run -n ros2py310 ...`）
     - 关键超参/seed、输入数据/地图、评估指标口径
     - 变更点摘要（可用 `_meta` 字段，避免被严格解析为 CLI 参数）
7) **环境约束**

   - 环境默认为 Ubuntu 24.04；Python 默认使用 conda 环境：`ros2py310`。
8) **仓库研究目标**

   - 最终目标：在森林场景车辆运动规划任务中，提出可复现、可验证的强化学习方法；在统一评测口径下相对多类基线（强化学习基线、传统规划+MPC 基线）达到更优综合性能，重点指标为 `success_rate`（越大越好）、`avg_path_length`（越小越好）、`path_time_s`（越小越好）。
   - 当前主线：`v7p1`（CNN-DDQN 稳定基线，short/long SR=100%）；SAC 迁移分支已推进到 `v8p2`（CBF-Safe SAC），当前 smoke SR=0% 但训练信号正向（best_return=477.5）。
   - 当前状态：最终目标尚未达成；`v7p1` 为 CNN-DDQN 阶段性最优结果。SAC 迁移链：`v8`（训练发散）→ `v8p1`（稳定性修复）→ `v8p2`（CBF 安全过滤 + 奖励塑形），下一步为 `v8p3`（课程学习 + alpha 下界）。
   - 方法边界：不限制于 `CNN-DDQN`；允许新增/替换模块或引入其他强化学习算法，但必须满足学术定义合规、可复现留档与门槛验证要求（见第 9、13、16 条）。
9) **学术定义合规（硬约束）**

   - 任何算法名/术语（如 DQN/DDQN/Double Q-learning、CNN 架构、Hybrid A*/RRT*、MPC 等）必须满足其学术或原论文定义；禁止“只改风格但不符合定义”的实现。
   - 若对定义或标准模糊：优先下载或整理对应论文 PDF 或官方实现仓库作为参考，并在说明中引用；建议放入 `paper/`（或记录可追溯链接/出处）。GitHub 仓库可单独建立文件夹。
10) **规则可追加**

- 以上规则可随时追加；新增规则以最新说明为准。

11) **README 同步（训练/推理命令）**

- 每次涉及代码或配置行为的改动后，默认同步更新 `README.md` 与 `README.zh-CN.md` 中“最新训练/推理命令”（含对应 profile 名）。
- 若命令未变化，也应明确检查并保持中英文两份 README 一致，避免漂移。

12) **时间优先验证流程**

- 深度强化学习迭代默认采用两阶段流程：`self-check -> smoke -> full runs=20`。
- 未经明确说明，每轮先完成 smoke（短训练+短评测）再决定是否进入 full 评测，避免直接长时全量实验。
- 推荐 smoke 命令使用 `conda run -n ros2py310 ...`，并限制在可快速回路内完成。

12.1) **远端优先执行（ubuntu-zt）**

- 训练/推理（含 `self-check`、`smoke`、`full`）默认先在 `ssh ubuntu-zt` 上执行；仅当远端不可用（如 SSH 失败、远端环境异常、远端资源不足）时，才回落本地执行以避免本机卡顿。
- 每次远端运行前，必须先执行“本地仓库 -> 远端仓库”同步；此处同步口径固定为本地覆盖远端（不包含 `runs/`）。
- 远端运行完成后，必须将远端 `runs/` 对应结果目录回传到本地 `runs/`，再进行后续分析与归档。

12.2) **版本标准工作流（默认）**

- 每次创建新版本（`vxpx`）前，必须先完成 GitHub 快照：`git status` clean、`git add/commit`、`git push` 成功。
- 每个新版本默认先跑固定 smoke 门：
  - 训练：`episodes=150`
  - 推理：`runs=3`（short/mid/long）
- 若 smoke 未显示明确收益（按当前目标：路径更短、计算时间更短、路径更平滑），则：
  - 不进入 full 评测；
  - 主线代码/默认命令回退到上一稳定版本（例如回退到 `v7p1`）；
  - 当轮作为失败版本归档（四件套必须记录失败原因与证据路径）。
- 下一轮在失败版本基础上递增命名继续（例如 `v7p2p1` 失败后进入 `v7p2p2`）。

13) **最终研究门槛（硬约束）**

- 最终结论必须在 `short/long` 双套件、各 `runs=20` 条件下汇报。
- 对标 `Hybrid A*-MPC` 时，最终目标模型（当前规划为 `SAC-GlobalCNN`）至少满足：
  - `success_rate(SAC-GlobalCNN) >= success_rate(Hybrid A*-MPC)`
  - `avg_path_length(SAC-GlobalCNN) < avg_path_length(Hybrid A*-MPC)`
  - `path_time_s(SAC-GlobalCNN) < path_time_s(Hybrid A*-MPC)`
- 若任一套件未满足上述三条，视为未通过最终门槛。

14) **推理期策略口径（命名必须一致）**

- 本仓库允许在**推理阶段**使用 `admissible-action masking`、安全过滤、`top-k` 重选、`stop override`、replacement、fallback、规划器（如 A*/MPC）接管（takeover）等混合策略，以提升成功率与安全性。
- 但必须显式区分并正确命名（论文/文档/版本留档必须与实现一致）：
  - `strict`（兼容旧口径 `strict-argmax` / strict no-fallback）：推理阶段必须使用模型原生直接输出动作（DQN 为 `argmax(Q)`，SAC 为确定性策略输出）；允许计算 mask 仅用于统计/诊断，但不得影响最终动作。
  - `shielded` / `masked` / `hybrid`：推理期允许上述干预（masking/top-k/override/replacement/fallback/takeover 等）；此时不得宣称 `strict-argmax` 或 strict no-fallback。
- 训练期仍可使用合法数据筛选、课程、辅助损失；若推理期启用干预，必须在配置与版本四件套中写清楚具体策略与开关。

15) **版本命名与留档（硬约束）**

- 版本命名统一采用 `vxpx`：**大改动**执行 `v+1`；**小改动**执行 `p+1`。
- 每个版本必须使用独立文件夹：`docs/versions/<version>/`。
- 每版至少包含：`README.md`（版本总结）、`CHANGES.md`（具体改动）、`RESULTS.md`（结果对比）、`runs/README.md`（对应 run 路径与口径）。
- 每个版本必须写 MD 留档（方法、详细修改点、参数、命令、结果、结论、下一步）。
- 推荐在新一轮开始前先读取上一版本留档，防止重复试验与口径漂移。

15.1) **版本更新前 GitHub 快照（硬约束）**

- 适用范围：任何准备发布新版本（`vxpx`）的代码或配置改动，在**进入实施**之前。
- 开始实施前必须确保 `git status`（工作区状态）为 clean；若不 clean，先 `git add/commit`（提交本地改动形成可回退点）或 `git stash`（临时存放改动）处理到 clean。
- 必须执行 `git push`（推送当前分支到远端 `origin`）并确认成功后，才允许开始改代码、改配置、创建新版本目录。
- 推荐：同时打快照 tag（`git tag -a <version>-pre -m "pre-change snapshot"`）并 `git push --tags`（推送标签），便于一键回退。
- 若 `git push` 或 `git push --tags` 失败：立即停止实施，先排查 SSH/权限/网络/远端分支保护等问题，确保远端有可回退快照。

16) **版本归档标准（执行细则，硬约束）**

- 适用范围：`v1` 起全部版本（含历史整理与后续新增）。
- 目录结构必须是：`docs/versions/<version>/`；禁止仅保留单文件 `docs/versions/<version>.md` 作为正式留档。
- 每个版本目录必须包含且仅少不了以下四件套：
  - `README.md`：版本目标、方法摘要、关键命令、代表 run、结论、下一步。
  - `CHANGES.md`：相对上一版的代码/配置改动明细（推荐 `old -> new` 口径）与受影响文件清单。
  - `RESULTS.md`：`table2_kpis_mean_raw.csv` 来源路径、short/long 指标、基线对比、门槛检查、`failure_reason` 分布。
  - `runs/README.md`：代表 run 的 `run_dir`/`run_json`/`kpi` 路径，以及该版本可追溯 run 列表。
- 归档入口统一为仓库根目录 `README.md` 中“版本总索引（v1 -> ...）”；`docs/versions/README.md` 作为子目录镜像索引同步维护。
- 指标与路径必须可追溯到真实文件；禁止编造历史结果。无法确认的数据必须写 `N/A` 并注明原因。
- `docs/versions/README.md` 必须同步更新（版本号、目录路径、short/long 最佳 SR、baseline、状态）。
- 历史版本整理时，允许保持算法不变，仅做文档结构与口径标准化；不得借机改写实验结论。
- 归档语言要求：`docs/versions/**` 下版本留档四件套默认必须使用中文撰写（含 `README.md`、`CHANGES.md`、`RESULTS.md`、`runs/README.md`）。
- 仅当你明确要求时，才允许对应版本留档使用英文或中英双语。
- 为保证可执行与可复现，命令行、文件路径、参数名、代码标识符可保留英文原文，不要求翻译。
- 每次运行完都要求归档：任何一次运行（含 `self-check`、`smoke`、`full`、训练命令、推理命令）结束后，必须在同一工作轮次内更新对应版本留档。
- 最小归档内容：在对应 `docs/versions/<version>/` 四件套中至少记录本次运行命令、`run_dir`、`run_json`、`kpi` 路径、short/long 关键指标与 `failure_reason` 分布（若该次运行产出该字段）。
- 运行失败也必须归档：若结果缺失，必须写 `N/A` 并注明失败原因（如报错中断、超时、人工终止）。

17) **文件写入限制（Claude Code 客户端问题，硬约束）**

- 单次 Write / Edit 工具调用写入内容不得超过 50 行；超过时必须拆成多次调用（先 Write 前 50 行，再用 Edit 追加后续内容）。
- 原因：Claude Code 客户端在单次写入过大时会静默报错或进入死循环，导致工具调用反复失败。
- 同理适用于生成大段代码、长 Markdown 文档等场景：宁可多调用几次，不要一次性写完。

18) **联网调研注意事项（Claude Code 客户端问题）**

- WebFetch 和 WebSearch 不混在同一批并行调用（WebFetch 403 会级联拖垮同批 WebSearch）。
- 每批并行最多 2 个同类调用。
- 优先 arXiv / GitHub 等开放源。
- PDF 链接大概率解析失败，优先用 HTML 版本（如 `arxiv.org/html/`）。
- **付费墙站点（tandfonline / sciencedirect / springer）**：WebFetch 会 403，改用 Playwright：
  1. `browser_navigate` 打开 URL
  2. `browser_wait_for` 等 5 秒（Cloudflare 自动验证）
  3. `browser_snapshot` 获取页面内容

## 默认环境

- 操作系统：Ubuntu 24.04
- Conda 环境：`ros2py310`
- 命令风格：优先 `conda run -n ros2py310 ...`
- 安装/快速开始/成功判定：优先参考 `../AGENTS.md`（避免重复维护）。

## 最小自检

```bash
conda run -n ros2py310 python train.py --self-check
conda run -n ros2py310 python infer.py --self-check
```

## 验收标准（最小）

- 文档层面
  - `AGENTS.md` 结构清晰、无明显歧义；与 `../AGENTS.md` 不冲突（重复信息尽量引用上层）。
  - `AGENTS.md` 与 `CLAUDE.md` 保持逐行一致；可用 `diff -u AGENTS.md CLAUDE.md` 验证。
  - 引用的关键路径存在：`train.py`、`infer.py`、`configs/`、`paper/`。
- 过程层面
  - 后续任何实现改动均遵守：先计划后动手、小步变更、学术定义合规、可复现性约束（除非你明确豁免）。
  - 默认执行两阶段验证（smoke 优先），最终结论使用 short/long + runs=20 的硬门槛口径。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/unstun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/unstun)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
