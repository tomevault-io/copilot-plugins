## fcstlabpro

> 本文档基于 **Institutional Crypto Alpha Research Framework** 构建，定义 FcstLabPro 的标准化研发、统计验证与部署流程。

# FcstLabPro 机构级研究与实验操作手册 (V2.0)

本文档基于 **Institutional Crypto Alpha Research Framework** 构建，定义 FcstLabPro 的标准化研发、统计验证与部署流程。

---

## 一、 研究架构设计 (Layered Architecture)

所有开发必须严格遵循分层原则，严禁跨层调用逻辑：

| 层级 | 定义 | 核心关注点 |
| --- | --- | --- |
| **Layer 0** | **数据层 (Integrity)** | 确保 `feature[t]` 仅使用 `<=t` 数据，消除未来函数。 |
| **Layer 1** | **标签层 (Labels)** | **强制非重叠 (Non-overlapping)** 收益计算。 |
| **Layer 2** | **信号层 (Raw Alpha)** | 纯粹的 `signal vs future_return` 研究，不含策略逻辑。 |
| **Layer 3** | **验证层 (Validation)** | **真正 Walk-Forward** 滚动训练与 IC 稳定性测试。 |
| **Layer 4** | **组合层 (Portfolio)** | 波动率目标 (Vol Targeting) 与信号平滑。 |
| **Layer 5** | **执行层 (Backtest)** | 考虑成本、滑点与延迟的真实回测。 |

---

## 二、 实验核心规范 (Hard Rules)

### 2.1 标签与采样 (Layer 1)

* **非重叠规则**：若预测窗口 ，实验样本必须每 21 天采样一次。
* **严禁行为**：严禁在研究阶段使用每日滑动的重叠标签进行统计。

### 2.2 滚动训练 (Layer 3)

* **标准模式**：必须使用 `Expanding` 或 `Rolling` Walk-Forward。
* **禁止行为**：严禁“一次训练，全段预测 (Train Once, Predict All)”的伪 OOS 测试。
* **锁定原则**：一旦进入 OOS 阶段，严禁调整超参数或根据结果反推信号方向。

### 2.3 统计准则 (Validation)

实验报告必须包含以下指标，且需达到机构级门槛：

| 指标 | 门槛 (Crypto 单资产) | 说明 |
| --- | --- | --- |
| **Rank IC** |  (有价值) | 超过  需启动代码审计 |
| **IC t-stat** |  (显著) | 基于 IC 时间序列计算，非样本量 |
| **Annualized Vol** | 目标  | 通过 Vol Targeting 实现 |
| **Sharpe (OOS)** |  | 成本后净收益比率 |

---

## 三、 实验执行流程

### 3.1 实验准备

1. **目标定义**: 明确验证的 Alpha 假设（如：Funding Rate 的领先性）。
2. **配置创建**: 在 `configs/experiments/` 下创建 YAML，需注明 `sampling_step`（需等于标签 ）。

### 3.2 运行训练与 IC 分析

```bash
# 1. 执行修正版 Walk-Forward 训练
python scripts/train_orion_walkforward.py --config configs/experiments/{exp_name}.yaml

# 2. 执行独立 IC 分析 (使用 ic_analysis_corrected.py)
python scripts/ic_analysis_corrected.py --bull-dir experiments/weekly/{exp_name}

```

### 3.3 结果判定

* 若 **Rank IC < 0.02**：判定为噪音，放弃该特征/模型。
* 若 **IC t-stat < 1.0**：信号不稳定，存在 Regime 依赖，需增加状态识别模块。
* 若 **Sharpe > 3.0**：极大概率存在数据泄露，需自查 Layer 0。

---

## 四、 实验经验与坑点清单

### 4.1 有效策略

* **Regime-Specific**: 分别优化 Bull/Bear 模型的  窗口（Bull 建议 21，Bear 建议 28）。
* **Vol Targeting**: 仓位与反向实现波动率挂钩，而非简单的 MA 过滤。
* **特征剪枝**: 特征过多(>100)会导致欠拟合，建议通过相关性分析保留前 30-50 个。

### 4.2 致命陷阱 (Pitfalls)

* **自欺欺人**: 反复调整  窗口直至 IC 变高（即 Over-fitting the process）。
* **信号延迟**: 忽视了  时刻收盘后，执行通常在  开盘，导致回测虚高。
* **模型容量**: `n_estimators` 过小导致 Fold 出现大量 Kappa=0，无法捕捉非线性关系。

---

## 五、 部署与同步流程

### 5.1 部署前强制自检

* [ ] **IC 验证**: `ic_analysis_corrected.py` 报告输出正常，t-stat > 1.5。
* [ ] **Non-overlapping**: 确认标签采样步长与预测窗口一致。
* [ ] **Docker 一致性**: 确认生产环境 Python 版本为 3.10。

### 5.2 模型晋升流程 (Promotion)

实验模型 **不能直接用于部署**。必须通过 `promote_model.py` 晋升到 `models/production/`：

```bash
# 1. 晋升 (自动执行 5.1 自检)
python scripts/promote_model.py \
    --experiment experiments/weekly/{exp_name} \
    --name {production_name} \
    --variant conservative

# 2. 提交到 git (models/production/ 是 git-tracked 的)
git add models/production/{production_name}/
git commit -m "promote: {production_name} from {exp_name}"

# 3. 部署 (MODEL_NAME 环境变量切换模型)
MODEL_NAME={production_name} ./deploy/deploy.sh
```

**晋升产物** (`models/production/{name}/`):

| 文件 | 说明 |
|------|------|
| `model.joblib` | 生产模型 |
| `config.yaml` | 训练配置 (含特征集、标签定义) |
| `manifest.json` | 模型谱系 (来源实验、git commit、哈希、指标、检查清单) |
| `meta.json` | 实验元数据 |
| `metrics.json` | 分类指标 |
| `pnl_metrics.json` | PnL 回测指标 |

**关键约束**:
- 部署脚本 (`deploy_v0305.sh`) 检查 `models/production/` 是否存在，不存在则拒绝部署
- `live_signal.py` 默认从 `models/production/{name}/` 加载模型
- `.gcloudignore` 确保 `models/production/` 进入 Docker 镜像
- 回滚 = `git revert` 晋升 commit

### 5.3 复现性验证 (Reproducibility)

**任何重构/代码修改后**，必须重跑生产模型验证数值一致：

```bash
# 重跑 E1
.venv/bin/python scripts/run_experiment.py \
    --config models/production/e1-conservative/config.yaml

# 重跑 E8
.venv/bin/python scripts/run_experiment.py \
    --config models/production/e8-touch/config.yaml

# 对比 metrics（必须 bit-exact）
diff models/production/e1-conservative/metrics.json \
     experiments/weekly/{新生成的E1目录}/metrics.json

# 对比 predictions（必须逐行一致）
diff experiments/weekly/weekly_bear_v0305_E1_decontam/predictions.csv \
     experiments/weekly/{新生成的E1目录}/predictions.csv
```

**已验证基线** (2026-03-08)：

| 模型 | Accuracy | Kappa | F1 | 状态 |
|------|----------|-------|----|------|
| E1 | 0.8733 | 0.3433 | 0.4142 | ✅ bit-exact |
| E8 | 0.9226 | 0.7512 | 0.7991 | ✅ bit-exact |

复现条件：`seed=42` + LightGBM 确定性训练 + 固定数据 (`data/raw/btc_binance_BTCUSDT_1d.csv`)。

> ⚠️ **历史教训**：v0301 Bear 模型 Kappa 从 0.05 崩到 -0.17 的事故
> 就是因为没有及时做复现验证。详见 `experiments/weekly/MODEL_REPRODUCE_SUMMARY.md`。

### 5.4 信号生成

```bash
# 信号生成时必须带上 --download 确保使用最新 Layer 0 数据
python scripts/weekly_signal.py --download --save

```

---

## 六、 本地 vs GPU 训练

### 6.1 训练环境选择

| 模型类型 | 训练环境 | 说明 |
|----------|----------|------|
| **LightGBM** | 本地 CPU | 快速训练，几分钟内完成 |
| **Orion-BiX** | GPU (vast.ai) | 需要 GPU 显存 8GB+，约 10 分钟 |
| **其他深度学习** | GPU (vast.ai) | 如 LSTM, Transformer 等 |

### 6.2 实验命名规范

**Config name 格式**: `{version}_E{N}_{description}`
目录路径已包含分类信息 (`experiments/weekly/`)，name 不再重复 `weekly_bear_` 前缀。

**示例**:
- `v0305_E1_decontam` → 目录: `experiments/weekly/v0305_E1_decontam_run1`
- `v0305_E8_touch_label` → 目录: `experiments/weekly/v0305_E8_touch_label_run1`
- `v0308_E15_savgol` → 目录: `experiments/weekly/v0308_E15_savgol_run1`

**后缀规则**:
- 默认: 自动追加 `_runN` 序号 (N = 1, 2, 3...)，同一配置多次运行不冲突
- `--overwrite`: 不加后缀，直接用 name 作为目录名（清空重跑）

```bash
# 默认: 自动追加 _runN
python scripts/run_experiment.py --config configs/experiments/weekly/{config}.yaml
# → experiments/weekly/v0305_E1_decontam_run1/

# 覆盖模式: 直接用 name
python scripts/run_experiment.py --config configs/experiments/weekly/{config}.yaml --overwrite
# → experiments/weekly/v0305_E1_decontam/
```

### 6.3 训练命令

**本地 (LightGBM)**:
```bash
python scripts/run_experiment.py --config configs/experiments/weekly/{config}.yaml --overwrite
```

**GPU (vast.ai)**:
```bash
# 1. 先提交代码到 GitHub
git add . && git commit -m "feat: 添加实验配置"
git push origin main

# 2. 在 vast.ai 上拉取代码并训练
nohup python scripts/run_experiment.py --config configs/experiments/weekly/{config}.yaml > experiments/weekly/{exp_name}/train.log 2>&1 &
```

### 6.4 实验提交规范

**每个实验完成后提交一次**（保持commit历史清晰）:

```bash
# 方式1: 每个实验单独提交
git add experiments/weekly/{exp_name}/
git commit -m "feat: 实验 {exp_name} 完成，Kappa=0.xx"

# 方式2: 多个相关实验一起提交
git add experiments/weekly/label_comparison_*/
git commit -m "feat: 完成标签策略对比实验"
```

**提交时机**:
- ✅ 实验有实质性进展/明确结论
- ✅ 文档/报告更新
- ❌ 快速迭代调参中（暂不提交）
- ❌ 结果不理想需要重做时

---

## 七、 GPU 远程训练 (vast.ai)

### 7.1 运行命令模式

在 vast.ai 等 GPU 服务器上运行实验的标准命令:

```bash
# 1. 创建实验目录
mkdir -p experiments/weekly/{exp_name}

# 2. 后台运行并保存日志
nohup python scripts/run_experiment.py --config configs/experiments/weekly/{config_name}.yaml > experiments/weekly/{exp_name}/train.log 2>&1 &

# 3. 查看日志
tail -f experiments/weekly/{exp_name}/train.log
```

### 7.2 实验完成后的操作

1. 检查 `meta.json` 中的 `status` 字段确认是否成功
2. 查看 `metrics.json` 获取汇总指标
3. 查看 `fold_metrics.csv` 分析各 fold 表现

---

## 八、 代码可复用性注意事项

### 8.1 可选依赖 (Optional Imports)

`src/models/__init__.py` 和 `src/experiment/runner.py` 中的 torch 模型（LSTM, GRU, TFT, PatchTST 等）使用 `try/except` 导入。**不安装 torch 也能正常运行 LightGBM 实验。**

### 8.2 已知 DRY 违规（待修复）

| 优先级 | 问题 | 位置 |
|--------|------|------|
| 🔴 高 | `_calculate_rsi/sma` 复制 3 份 | `labels/directional_filtered.py`, `labels/touch_filtered.py`, `features/technical.py` |
| 🔴 高 | `backtest.py` 串行/并行 fold 逻辑重复 ~80 行 | `src/evaluation/backtest.py` |
| 🟡 中 | `get_git_info()` 复制 2 份 | `src/experiment/tracker.py`, `scripts/promote_model.py` |
| 🟡 中 | `BaseModel.fit()` 缺少 `sample_weight` 参数 | `src/models/base.py` |
| 🟡 低 | `src/utils/io.py` 已写好但 0 处 import | 全项目 28 处裸 json/yaml 操作 |

**修复这些问题前，必须先通过 5.3 节的复现性验证。**

详见 `docs/cr_0308_reusability.md`。

---

## 九、 维护记录

* **2026-02-18**: 初始版本。
* **2026-02-19**: 引入 Institutional Framework，修正 IC 分析逻辑，增加 Layer 0-5 架构约束。
* **2026-02-27**: 添加 vast.ai GPU 远程训练命令规范。
* **2026-03-08**: 添加复现性验证流程 (E1/E8 bit-exact 确认)；torch 模型改为可选导入；记录 DRY 违规清单。

---

*本文档由 FcstLabPro 核心架构组维护*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joebling)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joebling)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
