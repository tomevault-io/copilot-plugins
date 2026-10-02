## travelweaver

> TravelWeaver 是面向长程旅行规划 Agent 的确定性、可回放环境。环境、Function Calling

# TravelWeaver 仓库协作指南

## 项目目标与当前阶段

TravelWeaver 是面向长程旅行规划 Agent 的确定性、可回放环境。环境、Function Calling
协议、通用任务规格、确定性 Reward、可审计任务合成、Tool-Graph SFT、在线 GRPO 和固定
ChinaTravel 官方评测链路已经可用。
当前工作重心是：

1. 扩展 official-recombined 合成任务与可执行 Tool-Graph ReAct 数据，并继续提高公开证据质量；
2. 在 Qwen3.5-4B 上迭代 SFT、hard-first Reward 和严格 on-policy GRPO；
3. 用固定官方评测、内部 replay/failure-stage 诊断和同配置 ablation 验证改动，而不是混合指标。

ChinaTravel benchmark 用于约束分布和表达风格参考，以及最终效果评估；合成训练数据不要求
逐字或逐题复制 benchmark，也不得直接复用 benchmark 原句。训练数据应覆盖相近能力分布，
同时保留更丰富的组合和表述。

## 两套隔离环境

### 根目录：环境、合成与 API rollout

- Python 固定为 3.10，使用 `uv`，不要向系统 Python 安装依赖。
- 初始化子模块：`git submodule update --init --recursive`
- 安装普通开发环境：`uv sync --dev`
- 安装 API rollout 依赖：`uv sync --extra api --dev`
- 准备 ChinaTravel 数据：`uv run travelweaver bootstrap chinatravel`
- 导入 benchmark：`uv run travelweaver import-tasks --split benchmark`
- 环境检查：`uv run travelweaver check-env`
- 根目录测试：`uv run pytest`
- 根目录静态检查：`uv run ruff check .`

根目录环境必须保持轻量，不得引入 PyTorch、CUDA、veRL、vLLM 或训练框架。

### `training/`：SFT 与 GRPO

训练栈是独立的 uv project，不得从根项目导入 GPU 依赖：

```bash
uv sync --project training --dev
uv run --project training python training/scripts/check_environment.py
```

当前训练环境：

- Python `3.10.19`
- veRL `main` commit `4a2cba76f7f605d2b9f56e640faaeaa71c2c7f71`（`0.9.0.dev`）
- PyTorch `2.10.0`，CUDA runtime `12.8`
- Transformers `5.5.3`，vLLM `0.19.1`
- FlashAttention `2.8.3`
- Flash Linear Attention `0.5.1`，FlashInfer `0.6.6`
- FSDP/FSDP2 训练后端，vLLM rollout 后端
- 当前主机 8 张 NVIDIA A800 80GB SXM4，compute capability `8.0`；已审计的 SFT/GRPO
  launcher 使用 2 卡或 4 卡配置，当前正式实验使用 GPU 0-3 的 4 卡配置

FlashAttention 在当前 glibc 2.31 主机上使用 CUDA 12.6 toolkit 本地编译；相关构建变量已写入
`training/pyproject.toml`。不要单独升级 PyTorch、vLLM、FlashAttention 或 CUDA 组合。当前不使用
SGLang、Megatron-LM、Apex 或 TransformerEngine。

修改训练依赖后应同步更新 `training/uv.lock`，重新运行环境检查。新增训练代码后使用：

```bash
uv run --project training pytest training/tests
uv run --project training ruff check training
```

## 当前代码结构

- `src/travelweaver/env/`：episode 状态机、15 个公开工具、稳定 ID、Scenario 和
  ChinaTravel backend。
- `src/travelweaver/data/`：数据库准备、校验和 benchmark 任务快照导入。
- `src/travelweaver/tasks/`：`TaskBlueprint`、`TaskSurface`、通用 `TravelTaskSpec`、编译与解析。
- `src/travelweaver/synthesis/`：配额目录、可行 witness、canonical 渲染、LLM polisher、
  preference audit 和版本化产物。
- `src/travelweaver/llm/`：provider-neutral OpenAI-compatible client 和 DeepSeek 配置适配。
- `src/travelweaver/rollout/`：Agent 循环、单任务 API rollout、可恢复的生成任务批量 rollout
  和完整轨迹。
- `src/travelweaver/reward/`：确定性约束验证、Reward 和严格 RFT 接纳过滤。
- `src/travelweaver/sft/`：程序化 Tool-Graph teacher、可见 rationale 润色、批次审计、轨迹重放
  和 trainer-neutral SFT。
- `src/travelweaver/evaluation/`：ChinaTravel 官方计划导出与审计，以及与训练 Reward 分离的
  盲测 LLM Judge。
- `src/travelweaver/cli/`：数据、合成、重写、rollout 和环境检查命令入口。
- `training/`：隔离的 SFT/GRPO 依赖、Qwen adapter、在线 AgentLoop、训练侧 Reward/sampler、
  checkpoint 评测、配置和 launcher。
- `tests/`：按根项目模块镜像组织的离线测试。
- `docs/`：架构、协议、Reward、合成计划和人工审计结论。
- `vendor/ChinaTravel/`：固定版本上游子模块；除非任务明确要求，不要直接修改。

## 协议与产物版本

修改序列化结构或行为时，检查并按需升级对应版本：

- Environment：`travelweaver-environment-v0.8`
- Observation：`travelweaver-observation-v5`
- Tools：`travelweaver-tools-v5-agent`
- Plan snapshot / evidence：`travelweaver-plan-snapshot-v2` / `travelweaver-evidence-v3`
- TaskSpec：`travelweaver-task-spec-v3`
- Blueprint / Surface：`travelweaver-task-blueprint-v2` / `travelweaver-task-surface-v3`
- Reward：`travelweaver-reward-v9`
- Outcome contract：`travelweaver-outcome-contract-v2`
- Trajectory：`travelweaver-trajectory-v12`
- Model tool response：`travelweaver-model-tool-response-v4`（默认 `delta`，兼容 `snapshot`）
- Model context policy：`travelweaver-model-context-policy-v4-commit-first`
- Scenario：`travelweaver-scenario-v1`
- Synthesis / artifacts：`travelweaver-synthesis-v50` /
  `travelweaver-synthesis-artifacts-v39`
- Programmatic policy / artifacts：`travelweaver-programmatic-policy-v75` /
  `travelweaver-programmatic-artifacts-v43`
- Hierarchical Tool Graph：`travelweaver-hierarchical-tool-graph-v4`
- Tool-call / generation graph：`travelweaver-tool-call-graph-v17` /
  `travelweaver-generation-dependency-graph-v9`
- Rationale polisher / prompt：`travelweaver-trajectory-rationale-polisher-v16` /
  `travelweaver-trajectory-rationale-prompt-v11`
- SFT：`travelweaver-sft-v7`
- GRPO prompts / split report / split artifacts：`travelweaver-grpo-prompts-v3` /
  `travelweaver-grpo-prompt-split-v4` / `travelweaver-grpo-prompts-split-v4`
- GRPO training Reward：`travelweaver-grpo-training-reward-v3`
- ChinaTravel official export / audit：`travelweaver-chinatravel-export-v1` /
  `travelweaver-chinatravel-official-audit-v1`
- Polisher prompt：`travelweaver-zh-polisher-v7`

不要在不升级版本和补兼容测试的情况下静默改变字段含义。读取旧快照时保持显式兼容或明确
拒绝，不要猜测缺失字段。

## 核心设计约束

1. 环境必须保持确定性和可回放。排序、分页、ID、路线、Scenario 和 Reward 不得依赖未固定
   的随机状态。
2. `tool_schemas.py` 是面向模型的统一 Function Calling 协议，不是 ChinaTravel 原始 API 的
   逐字段复制；上游字段转换集中在 backend。
3. Agent 只能引用本 episode 已展示的实体；计划只能引用已保存候选和已查询路线。不得绕过
   evidence contract 直接读取 oracle 数据。
4. ChinaTravel 只是首个任务来源。新约束进入通用 `TravelTaskSpec`，不得把 benchmark 特有
   逻辑硬编码进 Reward。
5. 训练 Reward 必须确定、可审计。LLM Judge 只做离线主观评估，不参与训练 Reward，也不与
   Reward 合并成单分数。
6. 默认使用进程内 Function Calling，不引入 MCP，除非项目范围明确改变。
7. Preference-like 的 `unscored_preferences` 当前不进入训练 Reward；只能通过独立 preference
   audit 或离线 Judge 分析，不能宣称 Reward 证明了偏好最优。
8. Scenario 是 episode 开始前冻结的替代世界快照，不是 rollout 中途随机注入的故障。
9. 模型上下文保持 budget-blind：有效动作上限、剩余步数、API turn 上限和连续错误计数只能进入
   operator audit，不能出现在 system prompt、user message 或 tool response。
10. `travelweaver-reward-v9` 的 Task Reward、训练侧长度/重复 shaping 和 ChinaTravel 官方指标是
    三层不同信号。训练侧 shaping 必须保留原 Task Reward，不得影响 SFT admission 或官方结果。

## ChinaTravel 官方 Benchmark 评测

- 任何运行、重算、比较、诊断或报告 ChinaTravel benchmark 的任务，必须先读取并使用仓库内
  `.agents/skills/chinatravel-official-benchmark/SKILL.md`。
- ChinaTravel 对外 benchmark 结果以固定上游 evaluator 的 `EPR-micro`、`EPR-macro`、
  `C-LPR`、`FPR`、`DAV`、`ATT`、`DDR` 和 `Overall Score` 为准。使用 Skill 内的仓库脚本
  统一导出计划、检查 654 题完整覆盖并生成可审计 summary。
- TravelWeaver 的 Reward、`all_hard_pass`、`rft_accepted`、evidence contract、invalid action、
  replay 和 failure stage 只作为内部训练门槛与诊断，不得冒充、替代或并入 ChinaTravel 官方
  指标。LLM Judge 同样保持独立，三类结果不得融合为单一分数。
- 对比 checkpoint 时固定 benchmark 快照、官方 evaluator revision 和 rollout 配置；若配置不同，
  只能报告端到端系统差异，不能宣称是纯权重因果效果。未完成的异步 rollout 只能标为 provisional，
  不得用先完成的样本外推完整成功率。
- 多卡评测使用 `training/scripts/run_chinatravel_checkpoint_eval_4gpu_tp1.sh` 的固定分片和
  `merge_chinatravel_eval_shards.py` 合并；缺失 task、API error 和 export failure 都保留在 654 题
  分母中。改变 shard assignment、采样参数或预算后必须标为新的端到端配置。

## 任务合成约定

- 通用混合数据使用 `--profile chinatravel_blended_v1_2`；新正式 SFT/GRPO 实验数据使用
  `--profile chinatravel_official_recombined_v1`。两种 profile 不得在未记录来源、配额和版本的
  情况下静默混合。
- official-recombined 只使用固定 benchmark 的匿名聚合能力统计，禁止复制题面、UID、实体、数值
  或单题联合签名。当前锁定 cohort 为 SFT 2,000 条（4 个 500 shard，seed `20260901`）和 GRPO
  1,000 条（2 个 500 shard，seed `20260911`）；后续 shard 使用 `--exclude-task-dir` 排除已完成 shard。
- 单一 seed 必须派生槽位、城市、人数、天数、约束、偏好、Scenario 和表层风格；不得混入
  未记录的随机状态。
- 先从固定世界构造可行 witness，再派生 Blueprint 和题面。LLM 只改写 `TaskSurface`，不能
  改动 Blueprint、witness、数字、实体或约束方向。
- 默认使用 `minimal_semantic` polisher validation。自然数字、同义表达和可修复 mention 可
  放宽，但城市、实体、归一化数值、交通方式、上下限方向、去返程作用域和硬约束弱化仍是
  hard error。
- 餐厅每餐预算必须明确至少安排一顿用餐；市内交通方式必须明确至少安排两个市内地点，防止
  约束因没有对应餐厅或路线而被架空。
- 普通 profile 的 polisher 失败时允许使用相同 Blueprint 的自然 canonical fallback，但必须记录
  完整 audit。official-recombined 必须使用 `--require-llm-polish`，只接纳最终 polish outcome 为
  `accepted` 的题面；GRPO prompt builder 会拒绝任何 canonical fallback。
- 每批产物必须检查：数量和配额、query/Blueprint 唯一性、benchmark 原句复用、witness 与
  materialized TaskSpec 的硬 Reward、alignment、fallback 率、候选实体频率和分类预览。
- 普通批次不为覆盖率硬塞动作。缺口使用独立 top-up profile，例如 pagination、cuisine、
  candidate review、verified recovery、hotel feature、preference compare 或 Scenario recovery；合并后
  按新总样本数重新计算覆盖率。目录返回值必须被后续搜索实际消费，分页必须由题面约束和当前页
  的公开证据驱动，不能因为隐藏 witness 位于后页而翻页。

常用命令示例：

```bash
uv run travelweaver synthesize-tasks \
  --profile chinatravel_official_recombined_v1 \
  --count 500 \
  --cohort-size 2000 \
  --shard-count 4 \
  --shard-index 0 \
  --seed 20260901 \
  --max-api-calls 1000 \
  --llm-concurrency 256 \
  --require-llm-polish \
  --validation-policy minimal_semantic \
  --output-dir data/generated/official-recombined-sft-2k/shard-00
```

## 程序化 Tool-Graph SFT 约定

- 程序化 teacher 从可行 witness 和题面义务构造 action，但每一步都必须在真实环境中执行；LLM
  不能决定工具、参数、候选取舍、action 顺序或最终计划。
- Hierarchical Tool Graph v4 允许 transport-first、local-first 和 interleaved 宏观顺序，但禁止
  相同 `search_*` 工具与 arguments 的重复调用；最多保留一个未消费 discovery，普通候选先 save，
  再查询必要 incoming route，且第一次 `submit_plan` 必须是唯一 terminal submission。
- 工具参数只能来自题面、当前或历史公开 observation、已保存候选、已查询路线或固定公开策略常量。
  隐藏 witness 只能证明任务可行，不能作为名称、ID、价格、筛选值或翻页目标的秘密来源。
- 轨迹接纳要求不超过 50 个有效动作、零 invalid action、无完全相同 action、正常
  `plan_submitted`、Reward 1.0、全部硬约束通过和 replay/evidence audit 一致。15 个工具的 10%
  覆盖率是批次补采建议，不是单轨迹或单批拒绝条件。
- 发布前运行 `audit-chinatravel-official` 和 `audit-programmatic-batch`。官方 exporter schema 是
  hard gate；固定 Common Sense audit 默认是必须显式报告的非阻塞 warning，只有使用
  `--require-official-commonsense` 时才成为 SFT admission gate。
- ReAct rationale 只能在 action 冻结并成功执行后润色。严格 official-recombined 数据使用
  `--require-all-rationales-polished`；润色按 sample transactional 立即保存成功样本，失败样本写入
  sibling quarantine/checkpoint，不得让一个失败样本阻塞或重请求其他已成功样本。

## DeepSeek 与批量 rollout 约定

- API 配置只从 `.env`/环境变量进入适配层，不得写入代码、日志或提交。
- 当前 rollout 基线为 `deepseek-v4-flash`、thinking enabled、`max_tokens=16384`、请求超时
  `600s`、每题一次 rollout。
- 环境默认允许 50 个有效工具动作，批量和单题 API rollout 默认允许 60 个
  API turn；连续 3 个 invalid action 仍终止 episode。
- Surface 和 programmatic rationale polisher 始终使用 thinking disabled；供应商特有 thinking
  参数只能留在 LLM 配置适配层。
- 所有支持并发的 DeepSeek 批处理默认使用 256 并发；当前 `repolish-tasks` 的
  `--llm-concurrency` 和 `rollout-generated` 的 `--concurrency` 都应设为 256。新增批量入口也
  使用相同默认值，除非用户明确修改。
- 批量 rollout 必须可恢复：按 `task_id` 跳过已有结果，将 API 错误单独写入 errors JSONL，
  不因单题错误丢失整批结果。
- 小批量抽查使用 `--limit N`；候选集合必须先按 seed 确定并保持题型与 Scenario 的比例，再根据
  已有输出恢复，不能让重复执行滑动到下一批任务。
- 模型可见工具返回默认使用 `--tool-response-mode delta`，只发送本轮结果和错误；有效动作上限、
  已用/剩余步数、API turn 上限和连续错误计数只保留在环境与 operator audit，禁止进入模型上下文。
  完整 StepResult 仍写入轨迹用于回放和审计。`snapshot` 只用于复现旧实验，v4 snapshot 同样预算盲化。
- 只有用户明确要求时才能调用付费外部 API。普通单元测试使用 fake client 或 mock。

```bash
uv run travelweaver rollout-generated \
  --input-dir data/generated/<batch-name> \
  --output data/trajectories/<rollout-name>.jsonl \
  --concurrency 256 \
  --max-api-turns 60
```

## SFT 数据转换约定

SFT 转换使用版本化、可审计的 `travelweaver-sft-v7`，不直接把原始 JSONL 临时拼成训练输入。

- 工具覆盖率是训练混合的补采目标，不是单批有效轨迹的拒绝条件。覆盖不足时保留已通过 Reward、
  协议和证据审计的样本，输出机器可读的 `top_up_required` 计划；定向追加新样本后按新分母重新
  计算覆盖率，直至 `ready_for_training_mix=true`。
- 只接纳正常 `plan_submitted`、`reward_valid=true`、全部硬约束通过且 `rft_accepted=true`
  的轨迹。
- 最终 Reward 为 1.0 的恢复型轨迹可以用于 SFT。action-only 清洗从 reset 开始跳过 invalid
  action，重放有效 action 并重生成 observation；不得直接从原 messages 中删行后继续使用旧
  observation。
- 默认采用 action-only SFT：删除全部 `reasoning_content`。system、user 和 tool observation 只作
  上下文，不计算 loss；仅正确 assistant tool call 是监督目标。
- ReAct SFT 必须显式使用 `supervision_mode=react`，只接纳来源为 thinking disabled、Reward=1、
  全程零 invalid action 且 assistant message 与 action 严格对齐的轨迹。保留可见
  `assistant.content` 并参与 loss，但仍禁止供应商私有 `reasoning_content`。
- ReAct Recovery 按 `docs/react-sft-recovery-v1.md` 实现：保留 invalid assistant/tool-error 回合
  作为上下文，对整个 invalid assistant message 设零 loss，监督后续可见反思和正确工具调用。
  转换时按原顺序重放全部 action 并重新生成 observation，不能删掉错误后继续使用旧上下文。
  当前 SFT 使用显式 `assistant_loss_mask`，不能根据错误文本隐式推断监督范围。
- `action_selective` 只在显式选择监督回合时使用：可以保留合法 teacher-forced context，但 mask
  必须与 assistant turns 一一对应，最后一次提交必须监督；不得从文本或错误类型隐式生成 mask。
- 模型侧工具 schema 和 arguments 必须按 schema 递归使用 required-first 顺序；模型 JSON 禁止
  `sort_keys=True`。canonical hash 可以独立排序，但不能改变模型实际看到的字段顺序。
- rollout 遇到 malformed 或非 object arguments 时，将模型历史规范为 `{}` 并作为 invalid action
  继续，把原始坏字符串只写入 trajectory audit，不能让下一轮 OpenAI-compatible 请求因历史坏
  JSON 返回 400。
- 首条 user content 使用真人题面的纯自然语言 `query`，不包装 JSON observation，不暴露
  episode ID、协议版本、合成类型或 Blueprint/Surface ID。工具 observation 仍使用版本化 JSON。
- SFT 中间 tool message 默认复用 `travelweaver-model-tool-response-v4` 的 `delta` 序列化，
  不重复 task、全部 candidates 或 visible ID 集合；旧 v3 轨迹可通过重放生成该格式。
- 删除最终 `submit_plan` 之后包含 Reward 明细的 tool response；Reward、隐藏 TaskSpec、oracle
  witness 和验证明细不得进入模型输入，只能写入 audit sidecar。
- 先产出 trainer-neutral JSONL，再通过目标模型官方 chat template 转换为 veRL Parquet；工具
  arguments 必须保持 JSON object，不能携带供应商原始字符串编码。
- Qwen3.5-4B 使用 `enable_thinking=false`。模板生成的空 thinking wrapper 是协议结构且须 mask，
  不属于 reasoning 训练内容；不要手工拼接 Qwen XML 工具调用。
- 清洗后校验 tool-call ID、消息配对、loss mask、协议版本和最大序列长度。超长轨迹隔离，
  不得截掉早期证据后只保留最终计划。
- 用 `training/scripts/split_qwen_sft.py` 做确定性 train/validation 切分，记录 seed、精确 assignment
  和输入 hash，并拒绝跨 split 的 Blueprint semantic hash 复用。official-recombined 2K cohort 使用
  1,800/200；该 validation 只作同分布训练诊断，654 题 benchmark 始终是独立盲测。

## 在线 GRPO 约定

- GRPO 输入先由 `prepare_grpo_prompts.py` 生成 prompt-only Parquet，再由
  `split_grpo_prompts.py` 确定性切分。manifest 必须证明 `contains_witness=false`、
  `contains_reward_labels=false`，记录 model-context policy/hash，且 train/validation task ID 不重叠。
  当前 official-recombined 1K 配置固定 900/100，并显式使用 seeded `--strategy shuffle`；对比实验
  不得擅自改为 stratified 或重排既有 assignment。
- Task Reward 使用 `travelweaver-reward-v9`：全部 hard-valid plan 统一为 `1.0`，失败按最早
  failure stage 落入确定性负区间，content quality 只进 audit。Reward 版本变化必须启用新的 run
  directory，禁止沿用旧 optimizer、sampler、rollout 或 W&B state。
- 训练标量使用独立的 `travelweaver-grpo-training-reward-v3`：保留原 Task Reward，再加入只作用于
  hard-valid 长输出的长度惩罚和互斥的 `normal` / `redundant_repeat` / `pathological_loop` 重复惩罚，
  最终 clip 到 `[-1, 1]`。原 Task Reward、两项 penalty 和分类必须分别记录。
- 当前严格 on-policy profile 固定 group size 8、每 step 8 个 prompt group、PPO mini-batch 8、
  `ppo_epochs=1`，即每 64 条真实 trajectory 只更新一次 actor；`norm_adv_by_std_in_grpo=false`。
  只允许一个 in-flight generation batch，不能通过异步积压引入 policy staleness。
- sampler 按最终 training reward 过滤所有精确 constant-reward group（全负和全正都过滤），并按缺失
  group 数同步 refill；连续 10 个有效零方差 group 时保存 checkpoint 和 stop report。不得把
  `Unsolved=0/8` 误判为无信号，只要八条负 Reward 不完全相同就仍可训练。
- 32,768-token cap 覆盖 prompt、assistant 输出和 tool observations。超长、step-limit 或无 terminal
  plan 轨迹必须得到确定性失败 Reward 并保留审计；不能因为其为负样本就从 raw policy-quality
  统计中删除。所有完成的 AgentLoop attempt（包括之后被过滤/refill 的 group）写入
  `${RUN_DIR}/rollout-traces-all/`。
- GRPO 训练的私有运行预算默认 `MAX_VALID_STEPS=70`、`MAX_AGENT_TURNS=80`；它们与 API rollout/
  官方 benchmark 常用的 50/60 配置不同，但两者都不能泄漏进模型上下文。
- 当前审计配置使用 Qwen3.5-4B、thinking disabled、train sampling `temperature=1.0/top_p=0.95`、
  validation sampling `0.5/0.95`、actor LR `1e-6`、frozen-reference low-variance KL `0.01`。
  先运行 4 卡 launcher 的 `--dry-run`；预检失败时不得绕过 manifest hash、模型族、版本 hook、
  batch/on-policy、工具数量、context 或 GPU topology 检查。
- 普通 rolling checkpoint 与 validation-best checkpoint 分开管理。Step 0 validation 也参与 best
  选择；比较 checkpoint 时优先使用 `best_checkpoint.json` 记录的 metric、Reward version 和来源
  step，不要只根据最新 step 或训练 Reward 推断官方效果。

```bash
PYTHONPATH=training/src:src uv run --project training python \
  training/scripts/prepare_grpo_prompts.py \
  --input-dir data/generated/<batch-1> \
  --input-dir data/generated/<batch-2> \
  --output data/grpo/<batch>/all.parquet

PYTHONPATH=training/src:src uv run --project training python \
  training/scripts/split_grpo_prompts.py \
  --input-parquet data/grpo/<batch>/all.parquet \
  --output-dir data/grpo/<batch>/split \
  --validation-count 100 \
  --seed 20260921 \
  --strategy shuffle

TRAIN_FILE=data/grpo/<batch>/split/train.parquet \
VAL_FILE=data/grpo/<batch>/split/validation.parquet \
MODEL_PATH=training/outputs/<sft-run>/<checkpoint>/huggingface \
  bash training/scripts/run_qwen3_5_4b_travelweaver_grpo_4gpu.sh --dry-run
```

## 修改规范

- Python 使用 4 空格缩进、完整类型标注和 100 字符行宽。
- 优先复用已有 dataclass、模型和错误类型，不新增重复结构。
- 新增或修改公开工具时，同时更新 schema、环境执行与状态逻辑、协议版本、测试和文档。
- 修改 TaskSpec、Blueprint、Surface、Reward、轨迹、合成或训练数据格式时，检查序列化兼容、
  版本常量、manifest 和已有快照。
- 外部模型调用保持 provider-neutral；供应商参数只放配置适配层，不污染通用 Agent 循环。
- 修改 GRPO AgentLoop、training Reward、sampler 或 trainer 时，保留 Task Reward 与 shaping 的字段
  边界、raw-initial 与 refill 指标边界、严格 on-policy batch 语义及全量 rollout trace；相关训练
  协议变化必须升级版本并使用新 run directory。
- GPU handoff launcher 只能停止已经核验 owner、CUDA binding、tmux session 和完整命令的已知进程；
  遇到零个、多个或未知占用者必须拒绝接管，不能扩大 kill 范围。`--dry-run` 不应改变 GPU 状态。
- 一次性验证逻辑要么沉淀为可复用脚本并配测试，要么在验证结束后删除；不要长期保留用途不明
  的临时脚本、兼容 wrapper 或死代码。
- 不覆盖或回退与当前任务无关的用户改动。工作区可能非 clean，修改前后都要检查 status 和
  diff，只提交本任务文件。

## 数据与安全

- 不提交 `.env`、API Key、模型权重、数据库副本、下载缓存、生成任务或 rollout 轨迹。
- `data/generated/`、`data/trajectories/`、`data/sft/` 和 `data/grpo/` 是本地生成产物；结论应
  来自 manifest/audit，不把大文件纳入 Git。
- `training/.venv/`、`training/checkpoints/`、`training/logs/` 和 `training/outputs/` 不提交。
- `rollout-traces-all/`、普通/最佳 checkpoint、W&B、Ray 和评测 shard 产物都属于可恢复运行状态，
  不提交；清理前先确认对应 run 已完整归档或可重建。
- 删除或覆盖生成批次、轨迹、checkpoint 前先确认精确路径；优先保留可审计原始产物。

## 验证要求

- 小改动至少运行相关测试。
- 环境协议、Reward、TaskSpec、synthesis 或 rollout 改动运行完整 `uv run pytest` 和
  `uv run ruff check .`。
- 涉及真实 ChinaTravel backend 时再运行 `uv run travelweaver check-env`。
- 训练依赖或 GPU 接口改动运行 training 环境检查；训练预处理和 adapter 改动运行 training
  tests 与 Ruff。
- 修改 SFT/GRPO launcher、split、AgentLoop、training Reward、sampler、trainer 或 checkpoint 评测时，
  运行对应 `training/tests`，再运行目标 launcher 的 CPU-only `--dry-run`；不要用一次真实长训练代替
  可重复的 preflight 和离线回归测试。
- 修改 ChinaTravel 官方导出、分片或汇总逻辑时，运行
  `tests/evaluation/test_chinatravel_official_skill.py`，并按官方 Skill 做 654 覆盖审计。
- Bug 修复必须补充能复现问题的回归测试。

## 文档与提交

- 设计决策优先写入 `docs/`，README 保持为安装和入口说明，训练环境细节写入
  `training/README.md`。
- 注释解释约束和原因，不重复代码表面行为。
- 提交按逻辑拆分，使用简洁 Conventional Commit，如 `feat:`、`fix:`、`test:`、`docs:`、
  `chore:`。
- 不提交与当前任务无关的用户改动，不修改已发布提交历史，除非用户明确要求。

---
> Source: [YoungZSh/Travelweaver](https://github.com/YoungZSh/Travelweaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
