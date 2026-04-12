## yolo-fusion-ua-lite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

**YOLO-Fusion-UA-Lite** — 基于 YOLOv8n 的双模态（RGB+IR）轻量化 OBB 车辆检测系统，数据集为 DroneVehicle（5类：car/truck/bus/van/freight_car）。

**框架关键约定：**
- 框架源码在 `ultralytics-8.2/`，以 editable install 方式安装（`pip install -e`）。**禁止** `pip install ultralytics` 覆盖，会产生静默错误
- 输入为 6 通道：ch1-3=RGB，ch4-6=IR。**模态顺序不可更改**，否则静默错误
- 只用 `data_croped/`（去白边 640×512），不要用 `data/`

## 目录结构

```
src/
├── cfg/
│   ├── datasets/          # 数据集 YAML（双模态/IR单模态）
│   └── model/             # 模型架构 YAML
├── dataset_preprocess/    # 数据预处理脚本（标签转换、裁切、清洗）
├── testing/               # 评测与推理可视化
├── tools/                 # 辅助工具（混淆矩阵、曲线绘制、权重迁移）
└── trainning/             # 训练脚本与训练管理器

ultralytics-8.2/ultralytics/   # 框架源码（含自定义修改）
├── nn/modules/fusion.py        # 融合模块（FeatureAttentionConcat、CrossModalFusionAttention 等）
├── nn/modules/block.py         # ModalitySelector、IdentityInput
├── nn/tasks.py                 # OBBModel、DualBackboneOBBModel
├── data/dataset.py             # YOLODualDataset（双路数据加载，自动配对 img/imgr）
└── models/yolo/obb/train.py    # OBBTrainer（含 modality dropout）

models/         # 训练输出与权重
result/         # 评测结果
data_croped/    # 实际使用的数据集（去白边 640×512）
docs/research/  # 研究文档（实验记录、方案、复盘）
```

## 架构数据流

```
输入 6ch(RGB+IR) → ModalitySelector(切分)
  ├─ RGB主干(ch1-3) → P3/P4/P5
  └─ IR主干(ch4-6)  → P3/P4/P5
        ↓
  融合模块(Inception+注意力+Concat) → 融合P3/P4/P5
        ↓
  FPN + PANet → Neck输出
        ↓
  三尺度 OBB 检测头
```

模型架构由 YAML 定义，位于 `src/cfg/model/`。数据集配置位于 `src/cfg/datasets/`。

## 环境

```bash
conda activate .\.conda\ultra82-py312
# Python 3.12 + PyTorch 2.6.0+cu124 + ultralytics 8.2.103 (editable)
```

## 常用命令

```bash
# 训练（通过 train_manager 统一入口，自动断点续训与异常重试）
python src/trainning/train_manager.py src/trainning/<训练脚本>.py

# 断点续训
python src/trainning/resume_train.py --resume <run_dir>

# 评测
python src/testing/test_general.py --device 0 --model-name <名称> --weights <权重路径>
# 完整参数见 test_general.py --help

# 模态缺失稳定性测试
python src/testing/test_stability.py
```

训练超参定义在各训练脚本的 `get_train_manager_spec()` 中，评测参数通过命令行传入。

## 关键工程约束

| 约束 | 说明 |
|------|------|
| **Ultralytics 版本** | 必须使用 `ultralytics-8.2/` 的 editable install，禁止 pip install 覆盖 |
| **模态顺序** | ch1-3=RGB，ch4-6=IR，改动导致静默错误 |
| **数据集** | 只用 `data_croped/`（去白边 640×512） |
| **新架构初始化** | 必须从 `yolov8n.pt`（COCO预训练）初始化 |
| **Inception 输入** | 输入通道数 `c1 % 4 == 0`，否则断言失败 |
| **AMP 兼容** | 融合模块中 SiLU/Sigmoid 必须 `inplace=False` |
| **融合模块接口** | 输入 `list[Tensor, Tensor]`，输出 `[B, 2C, H, W]` |

## 代码规范

- 所有代码包含详细中文注释
- 新融合模块写在 `fusion.py`，注册到 `nn/modules/__init__.py`，确保 `tasks.py` 可解析
- 新模型架构通过 YAML 配置定义在 `src/cfg/model/`
- 训练脚本需实现 `get_train_manager_spec()` 以兼容 train_manager

## 开发测试闭环 SOP

1. **修改融合模块** → `ultralytics-8.2/ultralytics/nn/modules/fusion.py`
2. **注册模块** → `__init__.py` 导出 + `tasks.py` 可解析
3. **编写模型 YAML** → `src/cfg/model/`
4. **快速验证** → 用 quick 脚本跑小比例数据冒烟
5. **正式训练** → `train_manager.py` 运行完整训练
6. **评测** → `test_general.py` 输出到 `result/`
7. **记录** → 更新 `docs/research/研究备忘录_v7.md`

### 注意力实验默认执行规范（Exp-0/A/B/C）

以下规则默认强制执行，不需要用户在每次会话重复说明。

#### 1) 预训练权重加载规范（双路主干）

- 初始化权重固定为 `yolov8n.pt`（COCO 预训练）。
- 双路主干实验禁止依赖 `pretrained=<pt路径>` 的隐式整网匹配加载。
- 必须使用**显式主干迁移**：把单路 backbone（`model.0~9`）参数复制到双路 backbone（RGB/IR 对应层）。
- 训练 overrides 中显式设置 `pretrained=False`，避免整网自动匹配污染融合层与检测头初始化。
- 迁移后必须打印统计日志（至少 `copied`、`skipped`），并在启动日志中可见。

#### 2) 统一超参检查清单（开训前逐项核对）

- `epochs=160`
- `freeze=10`
- 本仓库自定义语义：单主干与双主干均使用 `freeze=<N>` 表示前 `N` 轮冻结全部 backbone，并在第 `N+1` 轮开始前自动解冻。
- `patience=0`
- `close_mosaic=16`
- `close_dropout=16`
- `drop_prob_rgb=0.10`
- `drop_prob_ir=0.10`
- `erasing=0.0`（所有训练脚本必须显式设置）
- `batch=16`
- `conf=0.25`
- `iou=0.75`
- `max_det=500`
- `use_test_as_val=True`
- `imgsz=640`
- 验证阶段已禁用模态 dropout，无需额外配置。

#### 3) 训练输出目录约定

- 注意力实验统一输出到：`models/attention_exp/<RUN_NAME>/`
- 训练脚本中 `get_run_dir()`、`project`、`name` 必须保持一致，避免 Train Manager 与实际保存路径不一致。

#### 4) 开训前自动自检项

- YAML 文件存在且可解析。
- 数据集配置指向 `data_croped/`。
- 预训练文件 `yolov8n.pt` 存在。
- Train Manager 规范字段完整：`train_cmd`、`resume_cmd`、`resume_ready`、`run_dir`、`total_epochs`。

#### 5) 动态门禁前的单轮验证跑执行方式（强制）

- 单轮验证跑必须通过 Train Manager 拉起：`python src/trainning/train_manager.py --script src/trainning/<训练脚本>.py`。
- 监控训练日志，确认第 1 轮训练与验证结束后手动停止进程，用于保留 `weights/last.pt`。
- 禁止通过临时修改训练脚本（如把 `epochs` 改为 1、运行时覆写脚本常量）来替代该流程。
- 完成上述步骤后再执行动态门禁：`python src/trainning/regression_gate.py --script src/trainning/<训练脚本>.py --dynamic`。

## 重要文档索引

| 文档 | 路径 | 说明 |
|------|------|------|
| 研究备忘录 v7 | `docs/research/研究备忘录_v7.1.md` | 最高层级决策文档：实验记录、结论、工程规范 |
| 点子库与文献引用 v4 | `docs/research/点子库与文献引用_v4.md` | 研究方案索引与文献引用 |
| 阶段复盘与注意力分析 v3 | `docs/research/阶段复盘与注意力机制分析_v4_2026-03-22.md` | 模块对比与实验设计 |

科学问题、假设、实验设计动机、论文叙事等**研究上下文**在上述文档中维护，CLAUDE.md 不重复。
CLAUDE.md 只跟踪**工程上下文**：当前处于什么阶段、具体到第几个实验、什么做完了什么待做。

## 协作规范

### 双端协作模式

- **网页端 Claude**：研究讨论（实验设计、方案推演、论文叙事、科学分析）
- **本地 Coding Agent**：工程执行（编码、训练、评测、调试）

网页端产出的研究决策写入 `docs/research/`，本地 Coding Agent 根据决策执行工程任务。

### **重要规范，必须遵守**
- 每次进入新会话，先通读一次 `README.md` 再开始工程操作。
- 后续凡是涉及 `ultralytics-8.2/` 框架代码优化，必须在 `README.md` 增补变更记录与当前行为说明。
- 每次新建训练脚本后，在正式训练前必须先通过门禁：`python src/trainning/regression_gate.py --script <训练脚本路径>`（必要时追加 `--dynamic`）。

### 信息分层

| 层级 | 内容 | 维护位置 |
|------|------|----------|
| 研究上下文 | 科学问题、假设、实验设计动机、论文叙事、结论分析 | `docs/research/` |
| 工程上下文 | 当前阶段、实验进度、待办任务、已完成清单 | `CLAUDE.md`（本文件） |
| 工程规范 | 框架约束、代码规范、开发 SOP | `CLAUDE.md`（本文件） |

### CLAUDE.md 更新职责

本地 Claude Code 在以下时机更新 CLAUDE.md 的"项目当前进展"：
- 完成一个实验（训练+评测）后，标记为已完成
- 开始新实验前，添加到待办
- 用户给出新的阶段规划时，更新阶段描述

## 项目当前进展

**当前阶段：** 冻结错误修复后的前置对照重建阶段（先完成三组前置对照，再进入 Exp-B/C）

### 已完成

- [x] IR-only 从零训练（M0）
- [x] IR-only COCO 预训练基准（历史结果已标记受 freeze 错误影响，待重跑更新）
- [x] FA-Concat + Neck（M3）
- [x] FA-Concat + COCO 微调（M5，当前最优）
- [x] 正则化实验（M6）

### 本轮工程状态（新会话接续必读）

- [x] 框架 `freeze` 语义已统一：`freeze=N` 表示前 N 轮冻结 backbone，并在第 N+1 轮前自动解冻。
- [x] 断点续训已修复：当起始 epoch 超过冻结窗口时，自动 `skip backbone freezing`（仅保留 `.dfl` 常驻冻结）。
- [x] 框架参数防呆已加固：`freeze` 负值、非法索引、越界索引均直接报错；训练前打印 `Freeze plan`。
- [x] 固定回归单入口已上线：`src/trainning/regression_gate.py`（静态/动态门禁，失败即非零退出）。
- [x] Exp-0 训练脚本已就绪并通过门禁：`src/trainning/attention_exp0_train.py`。
- [x] Exp-A 训练脚本已就绪并通过门禁：`src/trainning/attention_expa_train.py`。
- [x] Exp-0 / Exp-A 输出目录统一为 `models/attention_exp/<RUN_NAME>/`。
- [x] IR 预训练脚本已切换为 `yolov8n.pt` 主干迁移范式（`pretrained=False` + 显式迁移 + 硬校验）：
  - `src/trainning/ir_only_pretrained_baseline.py`
  - `src/trainning/ir_only_pretrained_baseline_quick.py`

### 进行中 / 待办

- [ ] 第一组前置对照：IR-only COCO 重跑 vs M5（进行中）
- [ ] 第二组前置对照：Exp-0 vs Exp-A（待启动，IR-only COCO 重跑完成后）
- [ ] 第三组前置对照：CM-FA 重跑 vs M5（待启动，Exp-A 完成后）
- [ ] 结论审计检查点（待启动，三组前置对照完成后）
- [ ] 第二阶段注意力选型：Exp-B（待启动，结论审计通过后）
- [ ] 第二阶段注意力选型：Exp-C（待启动，结论审计通过后）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EdgarZhong)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EdgarZhong)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
