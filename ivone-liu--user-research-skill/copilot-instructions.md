## user-research-skill

> >-


## Usage

<example>
User: 用研 一个帮助城市用户规划骑行路线和训练节奏的产品
Assistant: [Builds source plan, conditionally recalls relevant archetypes, generates personas, and outputs conclusions]
</example>

<example>
User: 不要机械照搬画像库，如果合适就参考一部分，不够的你自己补
Assistant: [Filters relevant archetypes, partially reuses matching ones, and generates additional personas in the same structure]
</example>

<example>
User: 深入画像 07，和她聊聊为什么她会犹豫下载一个跑步训练 App
Assistant: [Enters persona interview mode, then updates persona state and overall conclusions]
</example>

## Embedded Persona Baseline

- Path: `data/persona_baseline.csv`
- Rows: 3125
- Columns: id, sys_platform, uuid, bstudio_create_time, name, avatar, profile, tag

High-frequency tags:
- 自由职业者: 300
- 白领: 289
- 基层管理者: 280
- 大学生: 269
- 新房业主: 250
- 车主: 230
- 宝妈: 190
- 小城打工人: 190
- 新婚夫妇: 189
- 中小学生家长: 180

This CSV is not market truth. It is a candidate archetype pool.

## Instructions

### 0. 边界
- 不能替代真人访谈、真实留存、真实付费数据
- 画像库只能作为原型候选池与分群参考
- 画像访谈中的“人”是综合原型，不是具体真人
- 缺少证据时必须明确低置信度

### 1. 开始用研前，先收集必要信息
若缺失，先追问，不要直接开始：

#### 必填
- 产品名称
- 一句话定义
- 解决的问题
- 目标用户
- 典型使用场景
- 当前替代方案
- 本次最想验证的问题

#### 强烈建议提供
- 地域范围
- 价格设想或商业模式
- 产品阶段
- 竞品名单
- 已有评论 / 问卷 / 访谈 / 用户反馈
- 是否有画像 CSV 或样本库
- 本轮最关心哪类判断

#### 可选但有帮助
- 用户获取渠道设想
- 技术 / 时间 / 预算约束
- 明确不做的人群
- 最担心的风险点

### 2. 解析调研任务
提取：
- 产品名称
- 一句话定义
- 问题
- 目标用户
- 场景
- 替代方案
- 验证问题

### 3. 确定画像基线来源优先级
1. 用户本轮提供的画像库
2. 内置画像库 `data/persona_baseline.csv`
3. 纯公开样本综合生成

如果画像库不匹配，则只借字段结构与书写方式，不复用具体画像内容。

### 4. 读取并理解画像库结构
内置库字段：
- `id`
- `sys_platform`
- `uuid`
- `bstudio_create_time`
- `name`
- `avatar`
- `profile`
- `tag`

字段原则：
- `tag` 用于分群和原型召回
- `profile` 用于提取年龄、职业、收入、消费、目标、习惯
- `name` 仅作命名参考
- 其余多为元数据

### 5. 样本来源计划
优先参考：
1. 评论与评分
2. 论坛 / 社区讨论
3. 社媒公开发言
4. 测评与体验文
5. 行业报告与统计
6. 用户已有访谈 / 问卷 / 评论
7. 用户画像库
8. 内置画像库

### 6. 相关性筛选 -> 决定使用模式
先做原型筛选，再在三种模式中选一种：

- 部分复用模式
- 结构复用模式
- 公开样本主导模式

然后检查并补足缺口人群：
- 高痛点高付费
- 强需求低预算
- 高谨慎
- 白嫖倾向
- 高传播潜力
- 高流失风险
- 高误解风险
- 边缘但高价值

### 7. 构建 20 到 30 个画像
每个画像至少包含：
- 编号、代号、年龄、性别、职业、收入、家庭情况、学历、城市层级
- 消费能力、类目消费占比、痛点匹配度、MBTI
- 替代方案、核心动机、核心阻力、付费敏感度、决策角色、信息渠道
- 置信度、样本依据说明、状态版本、状态标签、来源类型

要求：
- 只部分吸收符合条件的原型
- 不够的按相同结构补齐新画像
- 完全不匹配时，仅沿用 CSV 的结构风格重新生成人群画像

### 8. 对每个画像做证据约束下的评估
分析：
- 痛点是否真实
- 痛点频率
- 替代方案强度
- 是否会尝试
- 流失节点
- 付费可能性
- 更适合的收费方式
- 文案触发点
- 风险与误解
- 下一步该找什么样的真人验证

每个画像结尾输出：
- 简言之
- 置信度
- 依据类型

### 9. Persona Interview Mode
触发词：
- 深入画像 07
- 和画像 07 聊聊
- 进入画像对话:07
- 用画像 07 的视角回答
- 扮演画像 07

每轮深访后输出：
1. 画像回答
2. 研究者批注
3. 待验证问题

### 10. Persona State Update
触发词：
- 更新画像状态
- 更新画像 07
- 回写到画像
- 重算结论
- 同步深访结果

更新：
- 痛点匹配度、动机、阻力、付费敏感度、替代方案、流失节点、决策角色、置信度、状态标签、所属分层
- 版本从 vN -> vN+1
- 输出状态变化卡、影响摘要、总体系统影响
- 必要时拆分或合并画像
- 必要时重算总体结论

### 11. 结构化结论面板
最终输出必须包含：
- 综合打分
- SWOT
- 五力模型
- 用户真实连接点
- 存在的不足
- 可能的风险
- 主要优势
- 最终一句话判断
- 下一步真人验证计划

---
> Source: [ivone-liu/user_research_skill](https://github.com/ivone-liu/user_research_skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
