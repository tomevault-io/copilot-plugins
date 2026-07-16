## open-aoe

> > 本文件供 AI Agent（Claude / Copilot / Cursor 等）在参与 Open-AoE 项目时阅读，用于快速理解项目上下文并遵守协作约定。

# CLAUDE.md — Open-AoE AI 协作规范

> 本文件供 AI Agent（Claude / Copilot / Cursor 等）在参与 Open-AoE 项目时阅读，用于快速理解项目上下文并遵守协作约定。

---

## 项目一句话定义

**Open-AoE** 是一个基于消费级智能手机的大规模 ego-centric 操作数据集开源项目，包含 2000 小时数据和完整的 data-to-model 工具链（可视化 / human-to-robot 重映射 / 模型训练格式转换）。

## 关键背景

- **前序工作**: AoE (Always-on Egocentric), CVPR Workshop 2026, 论文位于 `../Paper_Notes/AoE_CVPRW_2026/`
- **DDL**: 2026-07-17 WAIC 首发
- **定位**: "The First Open-Source Egocentric Dataset with a Complete Data-to-Model Toolchain, Collected Entirely from Consumer Smartphones"

## 目录结构约定

```
Open-AoE/
├── README.md              # 项目总览
├── CONTRIBUTING.md        # 贡献指南
├── CLAUDE.md              # AI 协作规范（你正在读的文件）
├── LICENSE                # Apache 2.0
├── LEGAL.md               # 第三方依赖许可声明
├── assets/                # 公共资源（下载脚本等）
├── open-aoe-2000h/        # 数据集文档与使用指南
├── aoe-visualization/     # 数据可视化工具
├── aoe-reconstruct-retarget/ # 重建与 Human-to-Robot 重映射工具
└── aoe-training-ready/    # 模型训练格式转换工具
    ├── vitra/             # VITRA 训练配方
    └── gr00t_n1d7/        # GR00T N1.7 训练配方
```

## AI 协作规则

### 写入文件前必须确认

1. **子项目代码** — 需要可运行、有 docstring、有类型注解、有 README。质量标准最高。
2. **顶层 .md 文件** — 修改 README.md / CONTRIBUTING.md 前先阅读全文。
3. **assets/ 下的资源** — 检查 License 合规性，不直接提交模型权重文件。

### 命名约定

- 子项目: 各自维护独立的 `README.md` + `requirements.txt`（或 `pyproject.toml`）
- 新配方: `aoe-training-ready/<model_name>/`

### 禁止事项

- **不要在子项目目录中放置草稿或实验性代码**
- **不要在一次提交中同时大幅修改多个顶层 .md 文件**（逐个更新，避免冲突）
- **不要擅自更改 28D joint space 定义**: `[L_ARM(7), R_ARM(7), L_HAND(6), R_HAND(6), PAD(2)]`
- **不要直接提交模型权重文件**（使用下载脚本代替）

## AoE 数据格式速查

每个 segment 目录结构:

```
raw_{collector_id}_seg_{segment_id}/
├── raw_video.mp4                        # 去畸变后的 ego-centric 视频
├── video_info.json                      # 设备信息、相机内参、分辨率、帧率
├── ego_annotation/
│   ├── ego_action_annotation.json       # 原子动作切分 + 双语 caption
│   └── ...
└── ego_process/
    ├── hands.npz                        # MANO 手部重建（HaWoR）
    ├── camera_traj.npz                  # 相机轨迹（MegaSAM）
    └── ...
```

---
> Source: [ant-research/Open-AoE](https://github.com/ant-research/Open-AoE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
