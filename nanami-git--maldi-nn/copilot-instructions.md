## maldi-nn

> 如果子目录中存在更近一级的 `AGENTS.md`，则子目录文件优先生效。

# AGENTS.md

## 作用范围

本文件适用于当前仓库根目录下的全部内容。

如果子目录中存在更近一级的 `AGENTS.md`，则子目录文件优先生效。

## 项目定位

这是一个围绕 MALDI-TOF 质谱分析构建的 Python 深度学习项目，当前仓库主要用于：

1. 读取和预处理 MALDI-TOF 质谱数据。
2. 训练和使用用于菌种识别与 AMR 预测的模型。
3. 复现 Maldi Transformer、AMR recommender 及相关 baseline。

仓库既是可安装的 Python 包，也是一个实验代码和结果记录仓库。

## 顶层目录说明

- `maldi_nn`：主包代码，包含数据读取、预处理、模型、CLI 脚本和复现实验代码。
- `baselines`：当前仓库实际在用的本地 backbone、binned 训练和 peak shuffling 相关实现。
- `experiments`：自定义实验目录，包含本地训练脚本、数据处理脚本和 Hyper-V + DDA 流水线。
- `assets`：示例谱图、示例输入和预训练权重。
- `data`：本地数据目录，当前已有 `data/processed/*.h5` 缓存。
- `result`：训练输出、checkpoint、tensorboard 日志、指标和可视化结果。
- `.github`：GitHub 工作流配置目录。

## 关键源码结构

### `maldi_nn/spectrum.py`

质谱预处理核心模块，包含：

- `SpectrumObject.from_bruker`
- `SpectrumObject.torch`
- `SequentialPreprocessor`
- `VarStabilizer`
- `Smoother`
- `BaselineCorrecter`
- `Trimmer`
- `PersistenceTransformer`
- `Binner`
- `PeakFilter`

修改这里会直接影响数据分布、HDF5 结构和下游模型表现。

### `maldi_nn/nn.py`

底层网络定义，如 `MLP`、`Transformer`。  
`Transformer` 依赖 `bio_attention`，checkpoint 加载异常优先排查依赖版本兼容。

### `maldi_nn/models.py`

高层模型定义，如 `AMRModel`、`SpeciesClassifier`、`MaldiTransformer`。  
修改时必须考虑 checkpoint 兼容性。

### `maldi_nn/utils/data.py`

DataModule 核心文件，包含：

- `DRIAMSAMRDataModule`
- `DRIAMSSpectrumDataModule`
- `SpeciesClfDataModule`
- `batch_collater`

对 HDF5 schema 假设强，尤其是：

- `0/intensity`
- `0/mz`
- `central`
- `unstructured/split`
- `unstructured/species_labels`

不要随意更改字段名和 split 规则。

### `maldi_nn/utils/drug.py`

药物编码模块，包含 one-hot、ECFP、selfies、deepsmiles 等编码实现。  
依赖 `rdkit`、`selfies`、`deepsmiles`、`bio_attention`。

### `maldi_nn/scripts`

正式 CLI 入口目录，对应 `setup.cfg` 中的 console scripts，主要包括：

- `process_driams.py`
- `process_rki.py`
- `train_malditransformer.py`
- `train_amr.py`
- `train_clf.py`
- `domain_adaptation.py`
- `smoke_test_lstm_mz.py`

### `maldi_nn/reproduce`

论文复现实验目录，包含 baseline、finetune、峰分布估计及辅助训练组件。

## 本地实现说明

### `baselines/`

当前仓库的本地 backbone 主要在这里维护：

- `baselines/transformer_baseline`
- `baselines/lstm_baseline`
- `baselines/cnn1d_baseline`
- `baselines/fcn_baseline`
- `baselines/bin_training`
- `baselines/peak_shuffling.py`

`baselines/README.md` 已说明其与 `experiments/train_driams_main.py`、`experiments/train_bin_supervised.py` 的对应关系。  
如果要改 backbone、binned 训练或 peak shuffling，优先在这里联动检查。

### `experiments/`

`experiments` 是当前仓库的本地实验区，不属于上游主包逻辑，常见脚本包括：

- `train_driams_main.py`
- `train_bin_supervised.py`
- `process_driams.py`
- `estimate_peak_distr.py`
- `plot_random_spectrum_before_after.py`
- `hyperv_dda/`

其中 `hyperv_dda/` 是一套用于 Hyper-V + DDA + 双 GPU 的本地流水线。

## 数据与格式约束

项目高度依赖 `h5torch`。常见约束：

- DRIAMS AMR 默认 split：`A_train` / `A_val` / `A_test`
- 部分实验会并入 B/C/D 医院数据
- peak-based 模型通常要求峰数不低于 `n_peaks`
- `negpeaksampler` 依赖：
  - `unstructured/p_x_200`
  - `unstructured/p_y_x_200`

当前仓库已有的处理后数据文件：

- `data/processed/spectraraw.h5`
- `data/processed/spectrapks.h5`
- `data/processed/spectrabin.h5`

## 修改原则

1. 优先最小改动，不轻易重写核心预处理。
2. 不随意改 HDF5 字段名、split 名称、标签编码规则。
3. 改 `spectrum.py` 要假设所有模型都会受影响。
4. 改 `models.py` / `nn.py` 必须考虑 checkpoint 兼容。
5. 改 `utils/data.py` 必须联动考虑 collate、padding、脚本入口。
6. 为旧 checkpoint 做兼容时，优先在调用端兼容。
7. `experiments` 与 `maldi_nn` 主包逻辑要区分清楚。
8. 不要删除 `result/` 里现有实验产物，除非用户明确要求清理。

## 验证要求

按改动类型至少完成如下验证：

- 文档改动：检查 Markdown 结构和编码。
- 预处理或数据模块改动：做样本读取和 datamodule 初始化 smoke test。
- 模型改动：至少模型可初始化，必要时做一次前向。
- checkpoint 相关改动：至少验证一个已有 checkpoint 可加载。
- 脚本改动：至少跑 `--help` 或参数解析级验证。
- `experiments/` 脚本改动：优先做本地最小流程验证，不要假设与主包入口完全一致。

## 已知风险

- 对外部数据目录结构依赖强，尤其是 DRIAMS 和 RKI。
- 训练脚本偏向 GPU 环境。
- 旧 checkpoint 与新依赖版本可能不兼容。
- `result/` 里存在大量历史实验输出，做维护时要注意不要误删。

## 建议阅读顺序

1. `README.md`
2. `baselines/README.md`
3. `maldi_nn/spectrum.py`
4. `maldi_nn/utils/data.py`
5. `maldi_nn/models.py`
6. `maldi_nn/scripts`
7. `maldi_nn/reproduce/README.md`
8. `experiments/hyperv_dda/README.md`

---
> Source: [Nanami-git/maldi-nn](https://github.com/Nanami-git/maldi-nn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
