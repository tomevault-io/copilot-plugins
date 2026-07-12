## nous

> 认知层 AI 建模——不只模仿你怎么说话，理解你怎么想。

# Nous

认知层 AI 建模——不只模仿你怎么说话，理解你怎么想。

## Quick Facts

- **Stack**: Python (core/) + Next.js/TypeScript (web/) + Anthropic Claude API
- **Port**: localhost:3999 (web dev server)
- **Build**: `cd web && npm run build`
- **Test**: `python core/interview.py` (CLI), `cd web && npm run dev` (Web)
- **GitHub**: private repo, `git push` 推送

## Key Directories

```
core/          → Python 核心模块（interview, predictor, signal_extractor）
web/           → Next.js 前端（5 个 tab: 认知访谈/模型验证/偏差检测/认知预测/研究数据）
ml/            → ML 课程项目（DistilBERT 认知信号分类器）
data/          → 认知模型、信号历史、预测数据
research/      → 研究文档（bias-taxonomy, cognitive-signal-taxonomy）
scripts/       → 工具脚本（passive_collector, batch_analyze）
.claude/       → handoff.md (跨 session 协作) + settings.json
```

## 核心概念

- **9 认知维度**: Decision Architecture, Attention Allocation, Reasoning Style, Emotional Processing, Social Cognition, Blind Spots, Value Hierarchy, Response to Uncertainty, Execution-Layer Flexibility
- **15 类认知信号**: 7 行为 + 4 认知过程 + 4 认知偏差（详见 research/cognitive-signal-taxonomy.md）
- **矛盾检出**: stated（用户说的）vs behavioral（用户做的）不一致 = 客观盲区证据
- **双系统架构**: 系统1 在乎驱动能量分配 + 系统2 独立质量门槛

## 产品定位

不是"更准的认知模型"，是"知道行为层什么时候会出错的检测器"。
行为层的质检员，不是替代品。

## 当前数据

- 模型理解准确度：99%（直接判断模式）
- 行为预测：T1/T2 各 71%（确定性评分，随机基线 25%）
- 累积信号：101 个认知信号，20 条矛盾
- ML 标注：1K 条已完成，需标满 10K

## 跨 Session 协作

**必读**：`.claude/handoff.md` 是跨 session 共享交接文件。

**角色分工**：
- **主管 session**：规划、研究、review、写任务
- **CLI session**：执行编码任务、写完成报告

**强制规则（违反 = 任务未完成）**：
1. 开始工作前读 handoff.md
2. **每完成一个任务，在做任何其他事之前，立即更新 handoff.md**：
   - 待完成列表中标 ✅ + 完成日期
   - CLI 完成报告区写一行摘要（改了哪些文件、关键结果）
3. 不更新 handoff = 任务未完成。主管 session 只通过 handoff 来 review，看不到报告就当没做
4. **向用户汇报之前必须先检查**：handoff 更新了吗？没更新就先更新再汇报

## Code Style

- **不可变性**：创建新对象，不修改已有对象
- **文件 < 400 行**，函数 < 50 行
- **错误处理**：所有 API 调用 try/catch，给用户中文错误提示
- **Commit**: `<type>: <description>`（feat/fix/refactor/docs）

## Critical Rules

1. **不随便清 localStorage**——上次清掉了几天的对话修正数据
2. **这个 session 没有 ANTHROPIC_API_KEY**——需要调 API 的任务写到 handoff 让 CLI 做
3. **大文件用采样**——>100K chars 取开头+中间+结尾，不全量跑 API
4. **T1/T2 用确定性评分**——选对 1.0 选错 0.0，不走 LLM
5. **Vercel 60s 超时**——API 调用必须拆分并行
6. **handoff.md < 100 行**——超过时必须 compact：CLI 完成报告归档到已完成摘要（一天一行），删除详细报告正文，只保留关键数据和待完成任务。每次读 handoff 时检查行数，超过 100 行主动 compact。

---
> Source: [jessieyiqichen/nous](https://github.com/jessieyiqichen/nous) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
