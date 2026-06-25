## tencent2026-test

> 本仓库包含一个基于 **TabM** 的深度学习基线方案，用于 [TAAC2026](https://algo.qq.com/)（腾讯广告算法大赛）PCVR（点击后转化率）预测任务。

# AGENTS.md — Tencent Test (TAAC2026 TabM Baseline)

## 项目概述

本仓库包含一个基于 **TabM** 的深度学习基线方案，用于 [TAAC2026](https://algo.qq.com/)（腾讯广告算法大赛）PCVR（点击后转化率）预测任务。

项目使用 **扁平列式 Parquet 数据集** 存储用户-商品交互记录。每条记录包含：

* ID 与标签列（`user_id`、`item_id`、`label_type`、`label_time`、`timestamp`）。
* **User int 特征** — 标量或列表形式的 `int64` 值。
* **User dense 特征** — `list<float>` 值。
* **Item int 特征** — 标量或列表形式的 `int64` 值。
* **Domain sequence 特征** — 来自 4 个域的行为序列（`domain_a_seq_*`、`domain_b_seq_*`、`domain_c_seq_*`、`domain_d_seq_*`）。

目标标签为二分类：`label_type == 2` 视为正类。

当前活跃流水线是一个**内联 TabM 实现**（运行时无需外部导入 `pytabkit`）：

1. 使用 `pyarrow.parquet` 将原始 Parquet 文件加载到预分配的 NumPy 缓冲区中。
2. 训练带有可选 IWT 特征选择的 `TabMModel`。
3. 运行推理并输出 `{user_id: probability}` 格式的 JSON 预测文件。

> **注意：** `dataset.py` 中包含一个性能调优后的 `PCVRParquetDataset`（`IterableDataset`），但**当前 TabM 流水线**（`train.py` / `infer.py`）**并未使用它**。当前流水线使用其自有的低内存数组构建器（`train.py` 中的 `build_tabm_arrays` 和 `infer.py` 中的 `build_arrays_for_infer`）。

## 技术栈

* **语言：** Python 3.13+
* **包管理器：** `uv`（通过 `pyproject.toml` 和 `uv.lock`）
* **深度学习：** PyTorch 2.11+（Linux/Windows 上的 CUDA 13.0 wheel）
* **数据：** PyArrow / Parquet、NumPy、Pandas
* **机器学习库：** scikit-learn、LightGBM、XGBoost、CatBoost、SHAP
* **表格深度学习：** `tabm`（列为依赖项）、`pytabkit`（列为依赖项，但运行时代码已内联）
* **特征选择：** `iwt-pytorch`（Iterative Weighted Thresholding，迭代加权阈值）
* **可视化 / 监控：** TensorBoard、Matplotlib、Seaborn、Streamlit、JupyterLab
* **其他：** `kditransform`

## 项目结构

```text
.
├── pyproject.toml          # UV 项目配置 + 依赖
├── uv.lock                 # 锁定依赖树
├── .python-version         # "3.13"
├── run.sh                  # train.py 的 Bash 启动器
│
├── train.py                # 训练入口（TabM）
├── infer.py                # 推理入口（TabM）
├── model.py                # TabMModel + MLP + ensemble 层（从 pytabkit 内联）
├── trainer.py              # TabMTrainer，支持 AMP、早停、检查点保存
├── dataset.py              # PCVRParquetDataset（IterableDataset，train.py 未使用）
├── utils.py                # 日志、EarlyStopping、set_seed、focal loss
├── iwt_classifier.py       # IWT 特征选择实现
│
├── ns_groups.json          # NS-token 分组参考示例（仅示例）
│
└── data/
    ├── demo_1000.parquet   # 1000 行演示数据集
    ├── schema.json         # 特征元数据（词表大小、维度、前缀）
    └── demo_data_info.md   # 人类可读的 schema 文档
```

### 模块职责

| 文件 | 职责 |
|------|------|
| `train.py` | CLI 参数解析、数据加载（`build_tabm_arrays`）、模型构建、IWT 特征选择、启动 `TabMTrainer.fit()`。 |
| `infer.py` | 加载检查点 + `train_config.json`、重建数组、应用特征选择、运行推理、写入 `predictions.json`。 |
| `model.py` | `TabMModel`、`MLP`、`LinearEfficientEnsemble`、`ScaleEnsemble`、`CatEmbeddings`、`NLinear`、参数分组辅助函数（`make_parameter_groups`）。 |
| `trainer.py` | `TabMTrainer`：训练循环、验证 AUC 评估、AMP、梯度裁剪、检查点保存（`global_step*.best_model`）、TensorBoard 日志。 |
| `dataset.py` | `PCVRParquetDataset` 和 `get_pcvr_data()`：优化的可迭代数据集，含预分配缓冲区、shuffle、时间分桶、OOB 统计。 |
| `utils.py` | `create_logger`、`EarlyStopping`、`set_seed`、`sigmoid_focal_loss`。 |
| `iwt_classifier.py` | 用于连续特征分组特征选择的 `IWT_Classifier`。 |

## 构建与运行命令

项目使用 **UV** 进行环境管理。仓库中已存在 `.venv`。

### 激活虚拟环境

```bash
# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# Linux / macOS
source .venv/bin/activate
```

### 训练

```bash
# 通过便捷启动器
bash run.sh --data_dir ./data --ckpt_dir ./checkpoints --log_dir ./logs

# 或直接运行
python train.py \
  --data_dir ./data \
  --schema_path ./data/schema.json \
  --ckpt_dir ./checkpoints \
  --log_dir ./logs \
  --arch_type tabm-mini \
  --tabm_k 32 \
  --n_blocks 3 \
  --d_block 512 \
  --dropout 0.1 \
  --lr 2e-3 \
  --batch_size 256 \
  --n_epochs 1000 \
  --patience 16 \
  --max_cat_size 10000 \
  --valid_ratio 0.1 \
  --seed 42
```

### 推理

```bash
python infer.py \
  --ckpt_dir ./checkpoints \
  --data_dir ./data \
  --schema_path ./data/schema.json \
  --output_path predictions.json \
  --max_cat_size 10000
```

### TensorBoard

```bash
tensorboard --logdir ./logs
```

## 数据格式与 Schema

数据集以 **扁平 Parquet 文件**（非嵌套结构）存储。特征元数据位于 `schema.json`，包含四个顶层键：

* `user_int`：`[[fid, vocab_size, dim], ...]`
* `item_int`：`[[fid, vocab_size, dim], ...]`
* `user_dense`：`[[fid, dim], ...]`
* `seq`：`{domain: {prefix, ts_fid, features: [[fid, vocab_size], ...]}}`

训练 / 推理代码使用这些 schema 条目来：

1. 派生列名（`user_int_feats_{fid}`、`user_dense_feats_{fid}`、`{prefix}_{fid}`）。
2. 构建预分配的 NumPy 数组。
3. 将分类 ID 裁剪 / 哈希到 `max_cat_size` 上限。
4. 使用 z-score 归一化连续特征（训练时基于训练划分计算 mean/std；推理时基于全部数据）。

### 演示数据集 (`data/demo_1000.parquet`)

* 1,000 行、120 列、约 39 MB。
* 列分布：5 个 ID/标签 + 46 个 user int + 10 个 user dense + 14 个 item int + 45 个 sequence。

## 模型架构

`TabMModel` 支持多种由 `arch_type` 控制的架构变体：

* `plain` — 标准 MLP，无集成。
* `tabm` / `tabm-normal` — 在每个线性层上使用完整的 BatchEnsemble（`LinearEfficientEnsemble`）。
* `tabm-mini` / `tabm-mini-normal` — 在 backbone 前使用单个 `ScaleEnsemble` 适配器。
* `tabm-packed` — `NLinear` 并行层。

关键超参数：

* `k` — 集成大小（默认 32）。
* `d_block` — 隐藏维度（默认 512）。
* `n_blocks` — MLP 块数（默认 3）。
* `dropout` — dropout 比率。
* `cat_embedding_dim` — 分类特征的嵌入维度（默认 8）。

模型对分类特征使用 **Embedding 层**（`CatEmbeddings`）而非 one-hot 编码，以避免高基数特征的内存爆炸。

## 训练约定

### 特征选择 (IWT)

若 `--iwt_s` > 0 且连续特征数量超过 `iwt_s`，训练器会在 CPU 上运行 `IWT_Classifier`，在模型训练前选择 `x_cont` 特征的子集。选中的索引会保存到 `train_config.json`，并在推理时自动复用。

### 检查点布局

检查点保存到以下命名目录：

```
global_step<EPOCH>.arch=<TYPE>.k=<K>.d_block=<D>.best_model/
```

每个目录包含：

* `model.pt` — 仅 `state_dict()`。
* `train_config.json` — 完整训练参数及 `selected_cont_indices`。
* `schema.json` — 训练时的副本。

仅保留最新的 `best_model` 目录；旧的会自动删除。

### 早停

`EarlyStopping` 跟踪验证 AUC（越高越好）。它将最优 `state_dict` 同时保留在内存和磁盘上。默认 patience 为 16 个 epoch。

### 环境变量

为兼容训练平台，可通过环境变量覆盖 CLI 参数：

* `TRAIN_DATA_PATH` → `--data_dir`
* `TRAIN_CKPT_PATH` → `--ckpt_dir`
* `TRAIN_LOG_PATH` → `--log_dir`
* `TRAIN_TF_EVENTS_PATH` → `--tf_events_dir`
* `MODEL_OUTPUT_PATH` → `--ckpt_dir`（推理）
* `EVAL_DATA_PATH` → `--data_dir`（推理）
* `EVAL_RESULT_PATH` → `predictions.json` 的输出目录（推理）

### 日志

`create_logger(filepath)` 配置根记录器：

* `FileHandler` 级别为 `DEBUG`（写入 `filepath`）。
* `StreamHandler` 级别为 `INFO`（控制台）。
* 自定义格式器显示挂钟时间戳和已运行时间。

### 可复现性

`utils.set_seed()` 为 `random`、`numpy`、`torch` 设置随机种子，并强制 `torch.backends.cudnn.deterministic = True`。

## 代码风格指南

* **Docstrings：** 全文使用 Google 风格文档字符串。
* **Type hints：** 大量使用 `typing` 注解（`Optional`、`Dict`、`List`、`Tuple`、`Literal`）。
* **命名：** 函数/变量使用 `snake_case`，类使用 `PascalCase`，模块级常量使用 `UPPER_CASE`。
* **注释：** 行内注释解释形状语义、填充行为和复杂索引（例如时间桶裁剪）。
* **导入：** 标准库在前，第三方库次之，本地模块最后。

## 测试

本仓库**没有自动化测试套件**。验证通过以下方式进行：

1. 在演示数据集上运行 `train.py` 并确认 loss 下降。
2. 针对生成的检查点运行 `infer.py` 并检查 `predictions.json`。
3. 在 TensorBoard 中监控 `val/auc`。

## 安全注意事项

* 项目从用户提供的路径读取 Parquet 文件和 JSON schema。在生产环境中，请勿将 `--data_dir` 或 `--schema_path` 指向不受信任的位置。
* `clip_vocab=True`（默认）会将超出词表的 ID 静默裁剪为 0 而不是抛出异常。这是为了增强鲁棒性，但可能掩盖数据质量问题。
* 并行数据加载使用 `ProcessPoolExecutor`；工作进程从主进程派生。
* 核心流水线（`train.py` / `infer.py`）不暴露网络服务。Streamlit 和 JupyterLab 虽列为依赖项，但默认脚本不会启动它们。

---
> Source: [qtycc-cc/Tencent2026_test](https://github.com/qtycc-cc/Tencent2026_test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
