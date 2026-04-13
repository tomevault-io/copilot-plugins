## project-ap-ig

> ├── AGENTS.md                         # Codex 协作规则与项目总览

# AGENTS

## 目录结构

```text
project/
├── AGENTS.md                         # Codex 协作规则与项目总览
├── task.md                           # 历史任务顺序与旧推进计划
├── history.md                        # 数据、方法、结果、机制假说历史
├── survey/
│   ├── references/                   # 外部论文与参考仓库入口
│   └── notes/                        # 文献笔记
├── experiment/
│   ├── code/
│   │   ├── src/                      # 核心源码
│   │   ├── scripts/                  # 主实验脚本（split_ 前缀为数据分离后的 wrapper）
│   │   └── attentionhead/            # 注意力头专项实验代码
│   ├── datasets/
│   │   ├── clean/                    # 全量 1722 对 clean 样本
│   │   ├── corrupt/                  # 全量 1722 对 corrupt 样本
│   │   ├── train/                    # 70% 训练集（1223 对），用于电路发现
│   │   ├── test/                     # 30% 测试集（499 对），用于泛化验证
│   │   ├── merge_summary.json        # 全量元数据
│   │   └── split_summary.json        # 分层抽样记录（seed=42, lang×clean_candidate）
│   └── results/
│       ├── split/                    # ★ 当前主结果（数据分离后）
│       │   ├── pipeline/             #   train 集上的主流水线（电路发现）
│       │   ├── attentionhead/        #   train 集上的注意力头全量实验
│       │   ├── instruction_integration/  # 模块 1
│       │   ├── output_route_decision/    # 模块 2
│       │   ├── tool_call_construction/   # 模块 3
│       │   ├── tool_call_suppression/    # 模块 4
│       │   ├── test_validation/      #   test 集上的泛化验证
│       │   └── split_comparison_summary.md  # 全量 vs train vs test 对比表
│       ├── attentionhead/            # [旧] 全量注意力头结果（未分离）
│       ├── instruction_integration/  # [旧] 模块 1（未分离）
│       ├── output_route_decision/    # [旧] 模块 2（未分离）
│       ├── tool_call_construction/   # [旧] 模块 3（未分离）
│       ├── tool_call_suppression/    # [旧] 模块 4（未分离）
│       └── legacy/                   # [旧] 最终 signed circuit 与旧主结果包
├── paper/                            # 论文主文、图表、草稿
```

## 行为规则

- 没有完成目标，不要自行停下；先把当前回合内能做完的事情做完。
- 除绘图、论文写作、代码标识外，默认全部用中文，不用难懂缩写和空洞说法。
- 科研不是作秀；表达要简洁、清楚、有力，不写做作的话。
- 任何判断都要有依据；不确定就先查代码、数据、结果，不要硬猜。
- 不要编造实验结论，不要替数据说话。
- 非必要不写新文档；必须写文档时，优先图、表、短段落结合，保证 Markdown 预览可直接阅读。
- 需要翻墙时先执行 `source /etc/network_turbo`。
- 默认使用 `base` 环境，优先本地 `4090D` GPU。
- 不要中断、覆盖或抢占别人的进程；先检查，再行动。
- 临时文件，放置在/root/autodl-tmp/tmp文件夹下，而不是/tmp文件夹下

## 实验范式：Discovery / Validation 分离

- **所有新实验必须遵循 train/test 分离范式**：在 train 集（70%，1223 对）上发现电路、计算方向、确定节点，然后在 test 集（30%，499 对）上验证泛化性。
- **train 集上跑的实验**结果放在 `experiment/results/split/` 对应子目录下。
- **test 集上的验证**结果放在 `experiment/results/split/test_validation/` 下。
- 论文中引用的所有指标，必须同时报告 train 和 test 的数字；如果 test 上出现显著下降（AUC 下降 > 0.01 或 top1 下降 > 5%），需要在论文中讨论原因。
- 旧的全量结果（`results/` 下不在 `split/` 内的目录）仅作为历史参考，不再作为论文的主引用来源。

## 核心想法与现有结果

- 本项目研究的是：为什么只改用户提示开头的一个动词或短词组，就会让模型在首个输出词上从 `<tool_call>` 翻到非 `<tool_call>`。
- 当前统一使用的模型是 `/root/autodl-tmp/Qwen/Qwen3-1.7B`。
- 当前数据集共 `1722` 对 clean/corrupt 样本（Python 555 / Java 584 / C++ 583），关闭 thinking，采用 Qwen 原生工具调用格式；每对样本除首个用户要求词外尽量保持一致。
- 数据已按 `lang × clean_candidate` 分层随机划分为 **train 1223 对（70%）** 和 **test 499 对（30%）**，seed=42。
- 项目的核心方法是模块级电路定位，并加入双向反转：既把 `<tool_call>` 一侧当 clean，也把它当 corrupt，从而同时找到促进、抑制和共享部分，让电路更 faithful。
- 在 train 集上重新发现的 signed circuit 仍为 `24` 个节点、`64` 条边，与全量结果完全一致。
- 当前新的总机制假说分为 4 个模块：`Instruction Integration`、`Output-Route Decision`、`Tool-Call Construction`、`Tool-Call Suppression`。
- 注意力头全量聚合实验已在 train 集上完成：覆盖 `1223` 个样本、`448` 个注意力头。
- 当前 4 个模块的规范结果入口（数据分离后）：
  - `experiment/results/split/instruction_integration/`
  - `experiment/results/split/output_route_decision/`
  - `experiment/results/split/tool_call_construction/`
  - `experiment/results/split/tool_call_suppression/`
- test 集泛化验证结果在 `experiment/results/split/test_validation/`，关键指标：
  - Route Score R_module AUC：train 0.9946 → test **0.9943**（Δ = 0.0003）
  - Construction +MLP27 top1：train 97.9% → test **97.2%**（Δ = 0.7%）
  - Suppression +L23H6 no-tool top1：train 79.0% → test **78.2%**（Δ = 0.8%）
- 全量 vs train vs test 的完整对比表见 `experiment/results/split/split_comparison_summary.md`。
- 四个模块的当前最稳结论是：
  - `Instruction Integration`: `L2H14 + L11H5 -> MLP11`
  - `Output-Route Decision`: `MLP11 -> MLP16 -> MLP19`
  - `Tool-Call Construction`: `MLP19 -> L20H5 -> (L21H1 / L21H12) -> L24H6 -> MLP27`
  - `Tool-Call Suppression`: `MLP16 -> MLP17` 分叉进入 `L16H4 -> MLP17 -> L23H6`
- 当前最重要的工作已经从”继续扩模块实验”转为”统一总图、统一叙事、统一论文主文写法”。

## 当前机制猜想（英文）

> Before generating the first output token, the model first performs Instruction Integration, combining the opening request with the function-body phrase, the filename, and the remaining task description into a unified representation of what kind of answer is being asked for. It then enters an Output-Route Decision stage, where this integrated instruction state is converted into a stable internal choice between a tool-mediated output route and a direct-response route. If the decision favors the tool route, a Tool-Call Construction mechanism organizes filename cues, function-body content, and call-format structure to push the first token toward `<tool_call>`. If the decision favors the direct-response route, a competing Tool-Call Suppression mechanism strengthens the no-tool state while actively inhibiting the tool-calling pathway. In this view, tool use is not treated as an isolated yes-or-no switch, but as the outcome of a broader decision about how the answer should be produced and delivered.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Geuan666)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Geuan666)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
