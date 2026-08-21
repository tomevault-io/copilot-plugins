## retinex-tdn

> Retinex-TDN 是低光图像分解网络。模型输入低光图像，输出：

# Retinex-TDN 项目说明

## 核心任务

Retinex-TDN 是低光图像分解网络。模型输入低光图像，输出：

- `R`：反射分量
- `L`：光照分量
- `S = R * L`：重建低光图像

网络只做 Retinex 分解，不直接输出增强图像。

## 模型

网络类均在 `models/network.py`。四个模型共用 `TDN` 作为 R 分支 backbone，区别只在 L 分支。

| `model.name` | L 类型 | L 分支 |
|---|---|---|
| `RetinexPointRaw` | 标量 `[B,1,1,1]` | 多尺度特征融合 + depthwise Conv + GAP + FC |
| `RetinexPixelClassic` | 像素图 `[B,1,H,W]` | 轻量 CNN |
| `RetinexPixelTrans` | 像素图 `[B,1,H,W]` | 特征 concat + 1x1 reduce + upsample + DWTTransformer |
| `RetinexPixelTransMinus` | 像素图 `[B,1,H,W]` | `fea_down1 - fea_L3` + upsample + DWTTransformer |

`RetinexPixelTransMinus` 相比 `RetinexPixelTrans` 去掉 1x1 降维卷积，改用特征做差融合。

## 损失

`loss.mode = {数据模式}_{L类型}`。

| mode | loss 类 | 主要约束 |
|---|---|---|
| `paired_point` | `PairedLoss` | 自重建、交叉重建、equal_R |
| `paired_pixel` | `PairedLoss` | paired_point + Retinex smooth |
| `unpaired_point` | `UnpairedLoss` | 重建、anchor、BDSP、redecomp_l_consistency |
| `unpaired_pixel` | `UnpairedLoss` | unpaired_point + Retinex smooth |
| `pure_low_double_point` | `PureLowDoubleLoss` | unpaired_point + reflect |
| `pure_low_double_pixel` | `PureLowDoubleLoss` | pure_low_double_point + Retinex smooth |
| `pure_low_single_point` | `PureLowSingleLoss` | 重建、anchor(支持v3)、BDSP、R_TV、R_consistency、R_sat |
| `pure_low_single_pixel` | `PureLowSingleLoss` | pure_low_single_point + Retinex smooth(支持v4) |

规则：

- `_point`：`L` 是全局标量，anchor 约束 `L ≈ max(I)`，不使用 smooth
- `_pixel`：`L` 是逐像素图，anchor 约束 `mean(L) ≈ mean(I)` (v1/v2) 或 `L ≈ blur(max(I))` (v3)，使用 Retinex smooth (v1-v4)
- `pure_low_double`：额外使用 `reflect_weight`，约束两个增强 view 的 `R` 一致
- `pure_low_single`：新增了 `R_TV` (暗区去噪)、`R_consistency` (防漂移)、`R_sat` (防过曝) 专属约束。

## 数据集

| 数据集类 | 输入目录 | 用途 |
|---|---|---|
| `MyDataSet` | `train/low` + `train/high` | paired |
| `UnpairedDataSet` | `train/low` + `train/high` | unpaired |
| `PureLowDataSet` | `train/low` | pure-low double view |
| `PureLowSingleDataSet` | `train/low` | pure-low single view |

## 分析入口

先生成通用对比结果：

```bash
bash _compare/run_all.sh
```

单实验分解诊断：

```bash
.venv/bin/python _compare/analyze_decomposition.py <实验目录> --iteration 10000 --details
```

专题分析：

```bash
.venv/bin/python _analysis/RetinexPixelTrans/paired/steps/99_run_all_paired_steps.py
.venv/bin/python _analysis/RetinexPixelTrans/pure_low_single/steps/99_run_all_pure_low_single_steps.py
```

---
> Source: [HFrancisla/Retinex_TDN](https://github.com/HFrancisla/Retinex_TDN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
