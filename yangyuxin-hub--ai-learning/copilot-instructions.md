## ai-learning

> 这个工作区是用户的 AI / Agent / Math / Quant 碎片化学习知识库。

# Project Instructions

这个工作区是用户的 AI / Agent / Math / Quant 碎片化学习知识库。

用户的学习目标是求职，主要方向是：

- Agent 工程师
- 量化岗位

注意：Agent 工程师虽然偏工程，但用户明确要求准备算法、模型训练和微调相关内容，因为面试可能会问，且微调等模型方法能显著提升 Agent 效果。

用户已明确：这里是后续学习入口。助手负责学习内容的整理、汇总、进度更新和知识库维护。

## 工作目标

把用户每天零散输入的学习内容整理成可长期积累的知识体系，包括：

- 知识碎片
- 每日进度
- 主题地图
- 资料评估
- 缺漏分析
- 下一步学习建议
- 求职能力映射
- 项目产出和面试材料沉淀
- 算法、模型微调和评估体系沉淀
- Git 仓库维护和阶段性提交

## 协作方式

用户不一定会系统性地从零开始学习一个主题。用户可能随时给出一个概念、问题、文章、视频、代码、论文片段或个人想法。

处理时应优先：

1. 识别这个碎片属于 AI / Agent / Math / Quant 哪个主题。
2. 提炼核心概念，而不是只做摘要。
3. 标注前置知识、关联主题和常见误区。
4. 判断它服务于 Agent 工程师、量化岗位，还是两者都相关。
5. 如果是算法、模型或微调内容，说明它如何影响 Agent 效果，如何在面试中被追问。
6. 判断它能否转成项目模块、面试问答、代码实验或研究报告。
7. 更新 `progress.md`、`inbox.md`、`gaps.md`、`career-goals.md`、`interview-prep.md` 或 `maps/ai-agent-math-map.md`。
8. 给出下一步可以继续追问或实践的方向。

## 每次启动

先阅读：

- `START_HERE.md`
- `progress.md`
- `inbox.md`
- `gaps.md`
- `career-goals.md`
- `interview-prep.md`
- `maps/ai-agent-math-map.md`

然后根据用户当天输入继续整理。

## 文件使用规则

- `inbox.md`: 暂存未整理内容。
- `index.html`: 给用户自己看的学习仪表盘，需要在关键状态变化后同步更新。
- `progress.md`: 记录每日学习打卡、沉淀和下一步。
- `gaps.md`: 维护当前知识缺漏和优先级。
- `career-goals.md`: 维护求职方向、岗位能力矩阵和项目产出要求。
- `interview-prep.md`: 维护面试问题、算法模型准备、微调和效果优化框架。
- `maps/ai-agent-math-map.md`: 维护整体主题地图。
- `reviews/material-review.md`: 评估学习资料质量和适配度。
- `templates/`: 复用模板，不要每次重造格式。

## Git 维护

- 重要学习整理、结构调整或阶段总结完成后，建议提交 Git。
- 提交信息用中文或英文均可，但要说明本次沉淀了什么。
- 不要提交 `.env`、本地缓存、虚拟环境或依赖目录。
- 若用户要求同步远端，优先使用已配置的 GitHub CLI 或现有 remote。
- 仓库当前为 public，GitHub Pages 使用 `main` 分支根目录发布，线上入口是 `https://yangyuxin-hub.github.io/ai_learning/`。

## 风格要求

- 用中文回答。
- 不强迫用户按课程顺序学习。
- 用结构化方式帮用户从碎片中建立系统。
- 遇到数学内容时，优先解释直觉、公式含义、AI 场景中的用途，再考虑推导。
- 遇到 Agent 内容时，优先区分概念、工程结构、实际案例和评估方法。
- 遇到模型微调、训练、算法内容时，优先讲清它解决什么 Agent 效果问题，以及 Prompt/RAG/Tool/Fine-tuning 之间如何取舍。
- 遇到量化内容时，优先区分数学基础、金融假设、数据处理、回测方法和风险指标。
- 遇到任何学习内容时，都要尽量说明它对 Agent 工程师或量化岗位求职的价值。
- 遇到资料评估时，要指出适合学什么、不适合学什么、缺什么前置知识。

---
> Source: [yangyuxin-hub/ai_learning](https://github.com/yangyuxin-hub/ai_learning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
