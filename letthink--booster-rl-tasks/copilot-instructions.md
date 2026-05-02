## booster-rl-tasks

> 本仓库默认面向 Isaac Lab / RSL-RL 的训练诊断与调参工作。

# Booster RL Tasks Agent Workflow

本仓库默认面向 Isaac Lab / RSL-RL 的训练诊断与调参工作。

## 当任务涉及训练效果分析时

必须先读取结构化训练数据，再给调参建议：

1. 优先读取 `logs/**/events.out.tfevents.*`
2. 若 event 不可读，再回退到训练文本日志
3. TensorBoard 网页截图只能作为辅助，不应作为唯一依据

## 必看指标

- `Train/mean_reward`
- `Train/mean_episode_length`
- `Loss/entropy`
- `Loss/learning_rate`
- `Policy/mean_noise_std`
- `Episode_Termination/*`
- `Episode_Reward/track_lin_vel_xy_exp`
- `Episode_Reward/track_ang_vel_z_exp`
- `Episode_Reward/action_rate_l2`
- `Episode_Reward/joint_pos_limits`
- `Episode_Reward/joint_vel_limits`

## 默认调参顺序

1. 先判断是否是“任务定义/课程学习”问题
2. 再判断是否是 PPO 更新过激或探索不足
3. 最后才做小范围 PPO 标量调参

如果 `joint_pos_out_of_limit`、`joint_vel_out_of_limit`、`bad_orientation` 很高：
- 优先收紧动作尺度、命令范围、课程学习速度
- 其次加强动作平滑与 limit penalty
- 不要先提高学习率

如果 `episode_length` 很高但速度跟踪差：
- 优先提高速度跟踪奖励或补 gait/phase 观测
- 其次小幅增加探索

## 改动约束

- PPO 自动调参默认只允许修改：
  - `learning_rate`
  - `entropy_coef`
  - `desired_kl`
- 结构性调参应单独记录在 `docs/tuning_history/`
- 每轮调参都要写出：
  - 当前失败模式
  - 主要证据指标
  - 本轮改动
  - 下一轮观察阈值

## 推荐命令

先 dry-run：

```bash
python3 scripts/rl_ops/tune_and_record.py \
  --config-file source/booster_rl_tasks/booster_rl_tasks/tasks/manager_based/locomotion/agents/rsl_rl_ppo_cfg.py \
  --target-class T1WalkPPORunnerCfgV20 \
  --events-dir logs/rsl_rl/t1_walk_v20/<run_dir> \
  --dry-run --print-json
```

再决定是否实际修改。

---
> Source: [letthink/booster_rl_tasks](https://github.com/letthink/booster_rl_tasks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
