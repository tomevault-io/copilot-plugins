## 26spring-mip-surgicalteachingsys

> 本项目是一个结合了高真实性医学模拟与AI智能评估的“视网膜光凝手术”教育类游戏。旨在解决传统眼科规培中训练题目少、缺乏即时反馈、带教成本高及标准不统一的痛点。游戏受众为眼部外科手术医师及对眼科知识感兴趣的玩家。

# 视网膜光凝手术智能教学游戏系统 - 项目需求与系统设计文档 (claude.md)

## 1. 项目概述
本项目是一个结合了高真实性医学模拟与AI智能评估的“视网膜光凝手术”教育类游戏。旨在解决传统眼科规培中训练题目少、缺乏即时反馈、带教成本高及标准不统一的痛点。游戏受众为眼部外科手术医师及对眼科知识感兴趣的玩家。

## 2. 系统整体架构
系统整体划分为四层架构：
* **题库层（基础数据层）**：包含真实/合成病例数据、标准答案、错误标签库及五级难度关卡样本。
* **关卡生成层（模型1）**：离线大模型，负责根据难度体系结构化生成病例蓝图和关卡。
* **手术仿真层（游戏引擎层）**：在线阶段，负责视网膜物理仿真与实时渲染，接收玩家参数控制并产出操作日志。
* **教学复盘层（模型2）**：在线大模型，比对标准答案与玩家日志，输出结构化评分与教学反馈报告。

## 3. 核心业务模块规范

### 3.1 核心模拟机制 (Simulation Engine)
* **渲染逻辑**：基于物理学计算与患者特性进行图形学着色。核心包括高斯光束空间扩散模拟、非线性视觉着色、以及波长吸收与组织热损伤的渲染。
* **外设与交互映射**：
    * **摇杆/WASD键**：移动射击点与调焦/视角移动。
    * **控制面板/方向+数字键**：选择形状、波长及调节各类激光参数。关键参数包括：功率 (Power)、曝光时间 (Exposure)、光斑直径 (Spot Diameter)、光斑间距 (Spacing) 和内半径 (Inner Radius)。
    * **脚踏板/空格键**：发射激光。

### 3.2 离线题库生成智能体 (Model 1)
负责构建题库，基于**五级难度体系**生成训练关卡：
* **Level 1 (典型入门题)**：单一病灶，高容错率，侧重基础决策。
* **Level 2 (基础操作题)**：核心考察“试打与调参”。
* **Level 3 (范围与布局题)**：空间操作，要求精准控制光斑分布与覆盖。
* **Level 4 (危险区谨慎题)**：极度强调安全边界（避开黄斑/视盘等）。
* **Level 5 (综合复杂题)**：多病灶/合并症，全面考验复杂局势下的综合能力。

**要求的数据结构样本 (JSON)**：
```json
{
  "case_id": "L2_001",
  "level": 2,
  "source_type": "hybrid",
  "disease_type": "peripheral_retinal_tear",
  "disease_type_cn": "周边视网膜裂孔",
  "teaching_goal": ["意识到该病例需光凝治疗", "完成试打并调整功率", "安全区内完成包绕式光凝"],
  "case_summary_text": "患者男，46岁...",
  "eye_laterality": "right",
  "preop_image_path": "cases/L2_001/preop_fundus.png",
  "preop_image_meta": { "width": 1536, "height": 1536, "fov": "wide", "color_profile": "standard" },
  "lesion_annotations": [
    {
      "lesion_id": "lesion_01",
      "lesion_type": "retinal_tear",
      "mask_path": "cases/L2_001/masks/lesion_01_mask.png",
      "bbox": [1020, 320, 180, 140],
      "risk_level": "medium"
    }
  ]
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EricFeng9) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
