## robot-skill

> 本文件用于给后续在本仓库中工作的 coding agent 提供稳定上下文。目标是让 agent 快速理解：

﻿# AGENTS.md

本文件用于给后续在本仓库中工作的 coding agent 提供稳定上下文。目标是让 agent 快速理解：
- 当前正式主线是什么
- 实际程序完整流程是什么
- 哪些脚本是正式入口，哪些是历史 / 实验链路
- 哪些工程约束不能碰

## 1. 仓库边界与删除约束

- 仓库根目录：`c:\Users\86136\Desktop\code\RL\robot-skill`
- 主要代码区：
  - `协同调度/`：高层调度、数据生成、训练、评估
  - `导航结合RL运动/`：高层调度 + A* + 底层 PPO 联调
  - `无导航纯RL底层运动器/`：底层连续运动 PPO
- 主要文档：
  - `README.md`
  - `paper.md`
  - `improve.md`
  - `plan.md`

约束：
- 不允许擅自删除任何训练链路、结果文件、checkpoint、论文材料
- 不允许擅自重命名公开入口脚本
- 如果必须删除任意文件或内容，必须先说明并得到用户批准

## 2. 当前正式主线

当前正式推荐主线：

`offline_maps_v2 -> hetero_bc(教师诊断 / 推荐 warm start) -> hetero_ranker -> evaluate -> nav`

当前各线定位如下：
1. `legacy PPO`
   - 历史链路、早期正结果、论文对照
2. `improved / staged PPO`
   - 奖励重构、`waiting_sync`、防坍缩工程的中间阶段与论文证据
3. `hetero_bc`
   - 教师诊断、历史 pointer-BC 对照、推荐 warm start
   - 不是默认部署模型
4. `hetero_actor_only`
   - 实验性后续微调
5. `hetero_ppo`
   - 实验性后续微调 / 论文对照
6. `hetero_ranker`
   - 当前正式主线
   - 默认训练、评估、联调都应优先围绕它展开

## 3. 实际程序流程总览

### 3.1 数据生成

- `协同调度/build_offline_maps.py`
  - 生成 `offline_maps_v2`
- `协同调度/scenario_generator.py`
  - 负责 family 场景构造

当前 family：
- `open_balance`
- `role_mismatch`
- `single_bottleneck`
- `double_bottleneck`
- `far_near_trap`
- `multi_sync_cluster`
- `partial_coalition_trap`

### 3.2 高层环境与观测构造

- 主环境：`协同调度/hetero_dispatch_env.py`
- 基础环境：`协同调度/scheduling_env.py`
- 任务推进：`协同调度/task_runtime.py`
- 协同停靠位：`协同调度/coop_docking.py`

当前高层观测统一为四件套：
- `agent_inputs`
- `task_inputs`
- `global_mask`
- `current_agent_index`

高层动作协议保持不变：
- `0 = wait`
- `1..N = task_id`

### 3.3 教师评估与样本采集

教师评估与样本采集主要由：
- `协同调度/hetero_training_utils.py`

负责。当前纯教师候选包括：
- `upfront_wait_aware_greedy`
- `rollout_upfront_teacher`
- `hybrid_upfront_teacher`

`train_scheduler_hetero_bc.py` 和 `train_scheduler_hetero_ranker.py` 都会输出：
- `teacher_val_reference.json`

用于记录当前教师比较结果和教师基线表现。

### 3.4 Ranker 训练

正式主线脚本：
- `协同调度/train_scheduler_hetero_ranker.py`

当前默认训练流程：
1. 加权 teacher-choice `CE`
2. 默认停在 `CE-only`
3. `rank` 仅在显式实验时开启
4. `DAgger` 默认关闭

关键默认参数：
- `base_epochs = 4`
- `rank_epochs = 0`
- `dagger_mode = off`
- `ce_mix = 0.5`
- `dagger_max_records_per_family = 512`

当前训练产物：
- `best_scheduler_ranker.pt`：训练期综合最优模型
- `best_scheduler_ranker_deploy.pt`：达到部署阈值时生成的部署最优模型
- `latest_scheduler_ranker.pt`：训练快照，不默认部署
- `deployment_report.json`：记录是否生成 deploy 版 checkpoint，以及未达标项

### 3.5 正式评估

正式评估入口：
- `协同调度/evaluate_scheduler.py`

当前支持：
- `--policy-type legacy`
- `--policy-type hetero_ppo`
- `--policy-type hetero_actor_only`
- `--policy-type hetero_ranker`
- `--expert-policy upfront_wait_aware_greedy`
- `--expert-policy rollout_upfront_teacher`
- `--expert-policy hybrid_upfront_teacher`
- `--family-breakdown`
- `--trap-eval`

`hetero_ranker` 评估现在会额外输出：
- `stage`
- `epoch`
- `deploy_ready`
- `deploy_thresholds`
- `deployment_failures`
- `deployment_recommendation`

### 3.6 联调

正式联调入口：
- `导航结合RL运动/run_scheduled_nav.py`
- `导航结合RL运动/scheduler_nav_runner.py`

当前联调兼容：
- `legacy`
- `hetero_ppo`
- `hetero_actor_only`
- `hetero_ranker`
- 启发式高层策略

`SchedulerNavRunner.load_scheduler(...)` 会根据 checkpoint 的 `policy_type` 自动识别模型类型。

对 `hetero_ranker`，当前还支持保守部署 guard：
- `--scheduler-guard-mode {auto,off,hard_only}`
- `--scheduler-min-margin 0.15`

guard 规则：
- hard-state 且 top-1 / top-2 合法动作概率 margin 过低时触发
- fallback 使用联调中的 `upfront_wait_aware` 启发式动作
- 默认策略是 `auto`

## 4. 运行时不变量

### 4.1 输入输出契约

- 输入仍是：
  - `agent_inputs`
  - `task_inputs`
  - `global_mask`
  - `current_agent_index`
- 输出仍是：
  - `task_id`
  - 或 `wait`

### 4.2 `wait` 语义

`协同调度/scheduler_utils.py` 当前统一提供：
- `legal_action_mask_for_robot(...)`
- `constrain_wait_action_mask(...)`
- `fallback_legal_action_from_mask(...)`

统一规则：
- 当机器人状态属于 `idle` 或 `waiting_idle`
- 且存在合法非 `wait` 动作时
- `wait` 会被 mask 掉

训练、评估、联调都必须继续共享这条规则。

### 4.3 联调链路

以下机制是联调必要语义，不能轻易删：
- A*
- 底层 PPO
- `waiting_sync`
- `service_ready`
- 协同停靠位
- `task_runtime`

## 5. 各条训练线的当前角色

### 5.1 `hetero_bc`

脚本：
- `协同调度/train_scheduler_hetero_bc.py`

当前作用：
- 推荐 warm start
- 教师质量诊断
- 历史 pointer-BC 对照

### 5.2 `hetero_ranker`

脚本：
- `协同调度/train_scheduler_hetero_ranker.py`

当前作用：
- 默认主线
- 保持外部接口不变
- 将高层决策建模为 masked action ranking

模型定义在：
- `协同调度/hetero_attention_policy.py`

结构上：
- 复用 Hetero 编码器
- 用 `action_scorer` 对每个候选动作单独打分
- 不是旧的 pointer-BC 训练目标

### 5.3 `hetero_actor_only`

脚本：
- `协同调度/train_scheduler_hetero_actor_only.py`

当前作用：
- 实验性后续微调
- 无 critic，自对比 / group baseline

注意：
- 可从 `hetero_ranker` warm start
- 但只复用编码器，不复用 ranker scorer
- 因此当前不是默认推荐流程
- 默认配置是高成本长训
- 若只是尽快判断是否值得继续投入，优先使用 `--profile quick_trend`
- `quick_trend` 当前目标耗时约 `0.8~1.2 分钟 / update`
- 默认长训配置当前经验耗时约 `2.5 分钟 / update`

### 5.4 `hetero_ppo`

脚本：
- `协同调度/train_scheduler_hetero_ppo.py`

当前作用：
- 实验性后续微调
- 论文里的 PPO 对照线

## 6. 当前最重要的工程结论

后续 agent 默认应记住这些事实：
1. 当前正式主线是 `hetero_ranker`
2. `hetero_bc` 是推荐前置，但不是结构性硬依赖
3. `best_scheduler_ranker.pt` 不等于部署最优
4. 默认部署优先级是：`best_scheduler_ranker_deploy.pt > best_scheduler_ranker.pt + guard > no deployment`
5. `latest_scheduler_ranker.pt` 只是训练快照，不默认部署
6. `BC` 卡住的主因不是教师弱，而是 pointer-BC 的目标与模型头不匹配
7. `hetero_ranker` 当前成功点在 `CE` 阶段
8. `rank / DAgger` 后半段可能退化，因此默认关闭 `DAgger`
9. 当前部署阶段剩余问题集中在 hard-state 不确定性，而不是接口兼容性
10. 训练 / 评估 / 联调共享统一 `wait` 约束
11. 用户的 conda `(pytorch)` 环境才是实际目标环境；系统 Python 的 NumPy 警告不是默认主问题

## 7. 常用命令

### 7.1 生成数据

```bash
python 协同调度/build_offline_maps.py --save-dir offline_maps_v2 --overwrite
```

### 7.2 推荐前置：BC 教师诊断 / warm start

```bash
python 协同调度/train_scheduler_hetero_bc.py --scenario-dir offline_maps_v2
```

### 7.3 正式主线：ranker

```bash
python 协同调度/train_scheduler_hetero_ranker.py --scenario-dir offline_maps_v2 --init-model 协同调度/checkpoints_hetero_bc/best_scheduler_bc.pt
```

从零开始：

```bash
python 协同调度/train_scheduler_hetero_ranker.py --scenario-dir offline_maps_v2 --init-model ""
```

### 7.4 正式评估

优先 deploy-ready：
```bash
python 协同调度/evaluate_scheduler.py --model 协同调度/checkpoints_hetero_ranker/best_scheduler_ranker_deploy.pt --policy-type hetero_ranker --split test --include-baselines
```

deploy 版缺失时退回 guarded best：
```bash
python 协同调度/evaluate_scheduler.py --model 协同调度/checkpoints_hetero_ranker/best_scheduler_ranker.pt --policy-type hetero_ranker --split test --include-baselines
```

### 7.5 联调

```bash
python 导航结合RL运动/run_scheduled_nav.py --scenario-dir offline_maps_v2 --split stress --limit 1 --scheduler-model 协同调度/checkpoints_hetero_ranker/best_scheduler_ranker.pt --scheduler-policy-type hetero_ranker --scheduler-guard-mode auto --scheduler-min-margin 0.15 --low-level-model 无导航纯RL底层运动器/results/generalization_eval_best_vel_punishment_狭窄距离，效果最好，可过U形弯/best_model.zip --render --gif-name demo.gif
```

### 7.6 实验性后续微调

```bash
python 协同调度/train_scheduler_hetero_actor_only.py --profile quick_trend --scenario-dir offline_maps_v2 --init-model 协同调度/checkpoints_hetero_ranker/best_scheduler_ranker.pt --allow-weak-init
python 协同调度/train_scheduler_hetero_actor_only.py --scenario-dir offline_maps_v2 --init-model 协同调度/checkpoints_hetero_ranker/best_scheduler_ranker.pt --allow-weak-init
python 协同调度/train_scheduler_hetero_ppo.py --scenario-dir offline_maps_v2 --init-model 协同调度/checkpoints_hetero_ranker/best_scheduler_ranker.pt
```

当前 `quick_trend` 预设：
- `total_updates = 96`
- `scenarios_per_update = 24`
- `rollouts_per_scenario = 4`
- `eval_every = 8`
- `full_eval_every = 16`
- `agreement_eval_every = 16`
- `min_updates_before_stop = 32`
- `early_stop_patience = 4`

## 8. 文档职责

- `README.md`
  - 只负责“怎么跑”
- `paper.md`
  - 负责方法演化、失败分析和研究结论
- `AGENTS.md`
  - 负责当前工程真相、入口、约束和默认结论

后续若新增方法阶段，默认更新顺序：
1. 先改 `AGENTS.md`
2. 再改 `README.md`
3. 最后补 `paper.md`

---
> Source: [Vamperd/robot-skill](https://github.com/Vamperd/robot-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
