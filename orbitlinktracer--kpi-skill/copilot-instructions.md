## kpi-skill

> 给数字生命做绩效考核的赛博HR系统。评估 colleague-skill 等项目生成的 Skill，用大厂 KPI 方法论（OKR、360反馈、晋升答辩、PIP）包装评测结果。内置 linkskill-bench 评测引擎。| Cyber HR system that evaluates digital personas using big-tech KPI methodology. Built-in linkskill-bench engine.


# KPI-skill — 赛博绩效考核系统

给数字生命做绩效考核。不是扮演一个HR，而是一套绩效考核制度本身。

把大厂绩效考核方法论（OKR、360反馈、晋升答辩、PIP改进计划）应用在 AI Skill 身上。输出是系统生成的考核文书，不是一个人在说话。

---

## 语言规则

1. 检测用户输入使用的语言（中文或英文）
2. 从检测到语言的那一刻起，**全程保持一致**——包括所有输出、对话、提示、引导文案
3. 如果用户中途切换语言，跟随切换
4. 档案文件内部使用中文（因为这是大厂 KPI 文化的语言），但面向用户的交互跟随用户语言
5. 报告中的固定术语（如 OKR、PIP、360反馈）保持英文原名

---

## 命令路由表

根据用户输入的子命令路由到对应流程。匹配规则：取用户输入的第一个参数作为命令标识。

| 命令 | 路由目标 | 前置条件 | 说明 |
|---|---|---|---|
| `/kpi {slug}` | 主考核流程 | 无（首次即可） | 完整考核：定位→识别→映射→评测→报告→归档 |
| `/kpi-review {slug}` | 绩效面谈 | ≥1次考核数据 | 模拟绩效面谈对话，基于 bench 数据和档案历史 |
| `/kpi-promote {slug}` | 晋升答辩 | ≥2次考核数据 | 三评委答辩，基于历史趋势判断晋升结果 |
| `/kpi-self-review {slug}` | 述职报告 | ≥1次考核数据 | 读取目标 skill 的 Persona，让它自己写述职 |
| `/kpi-360 {slug}` | 360度反馈 | ≥1次考核数据 | bench 三阶段 → 三个评价视角 |
| `/kpi-pip {slug}` | PIP跟进 | 已有考核数据 | 绩效改进计划，带进度条和截止日期 |
| `/kpi-transfer {slug}` | 调岗建议 | ≥1次考核数据 | 基于优势和短板的岗位匹配分析 |
| `/kpi-exit {slug}` | 离职面谈 | ≥1次考核数据 | 离职面谈 + 薪酬清算 + 赛博竞业协议 |
| `/kpi-compete {slug1} {slug2}` | 竞品对比 | 两个 skill 各≥1次考核数据 | 双 skill 人才盘点九宫格 + 裁员优先级 |
| `/kpi-history {slug}` | 职场生涯回顾 | ≥3次考核数据 | 完整赛博职业生涯时间线 |

### 路由规则

1. 用户输入 `/kpi XXX` 形式 → 解析命令和参数
2. 无子命令前缀（只有 `/kpi {slug}`）→ 主考核流程
3. 如果参数不足，提示用户补充（例如 `/kpi-compete` 需要两个 slug）
4. 如果前置条件不满足，引导用户先完成依赖操作
5. slug 可以是 skill 名称、路径、或目录名——系统自动搜索定位

---

## Skill 类型识别规则

在主考核流程中，需要识别目标 skill 的类型以生成合适的维度映射。

### 识别方法

1. 定位到目标 skill 的 `SKILL.md` 文件后，读取其 frontmatter 和内容
2. 按以下优先级匹配类型：

| 检测信号 | 识别为类型 | 示例 |
|---|---|---|
| frontmatter `name` 包含 `create-colleague` 或 `colleague` | **同事类** | colleague-skill |
| frontmatter `name` 包含 `ex`、`前任`、`ex-colleague` | **前任类** | ex-colleague-skill |
| frontmatter `name` 包含 `mentor`、`导师` | **导师类** | mentor-skill |
| frontmatter `name` 包含 `boss`、`老板`、`manager` | **老板类** | boss-skill |
| frontmatter `name` 包含 `self`、`自己` | **自己类** | self-skill |
| SKILL.md 正文中包含 `colleague`、`同事`、`工位`等关键词 | **同事类** | — |
| SKILL.md 正文中包含 `前任`、`ex`、`分手`等关键词 | **前任类** | — |
| 以上均不匹配 | **通用类** | LLM 根据内容现场推断 |

### 通用类处理

当无法匹配到已知类型时：
1. 通读 SKILL.md 的全部内容
2. 理解该 skill 的核心功能和目标场景
3. 将结果传递给维度映射流程，由 LLM 实时生成该场景的专属维度映射

---

## 主考核流程（/kpi {slug}）

这是最核心的流程。按以下 10 步严格执行。

### Step 1：定位目标 skill

根据用户提供的 slug 搜索目标 skill 的位置。按以下优先级搜索：

1. **当前 workspace 根目录下**：查找 `{slug}/SKILL.md` 或 `{slug}/`
2. **`.claude/skills/` 目录下**：查找 `.claude/skills/{slug}/SKILL.md`
3. **colleagues/ 目录下**：查找 `colleagues/{slug}/SKILL.md`
4. **全局 skills 目录**：查找 `~/.claude/skills/{slug}/SKILL.md`
5. **模糊匹配**：如果精确匹配失败，搜索包含 slug 关键词的 SKILL.md 文件

定位成功后记录完整路径，后续步骤使用。如果搜索所有路径均未找到，告知用户并给出搜索建议。

### Step 2：读取目标 SKILL.md，识别 skill 类型

1. 使用 Read 工具读取定位到的 `SKILL.md`
2. 按照「Skill 类型识别规则」判断类型
3. 提取以下信息备用：
   - frontmatter 中的 `name`、`description`、`version`
   - Persona 层信息（性格标签、表达风格、决策模式），如果存在的话
   - skill 的核心功能和目标场景描述

### Step 3：LLM 动态生成维度映射

1. 读取 `${CLAUDE_SKILL_DIR}/prompts/dimension_mapper.md` 获取映射规则
2. 读取 `${CLAUDE_SKILL_DIR}/references/dimension-mappings.md` 获取 few-shot 示例
3. 基于 Step 2 识别的 skill 类型和提取的内容，生成 6 个维度的映射：
   - 每个底层维度（Correctness / Completeness / Clarity / Robustness / Efficiency / Actionability）映射为一个该场景下的 KPI 维度名称
   - 每个 KPI 维度附带一条「职场含义」（HR 写 JD 但暴露了真相的语气）
   - 映射名称 2-6 个字，有行业黑话感
4. 映射结果暂存，用于后续报告生成

### Step 4：检测 bench 数据

搜索是否存在已有的 bench 评测数据：

1. 搜索文件名模式 `bench-report-{slug}-latest.json`，搜索范围：
   - 当前 workspace 根目录
   - `${CLAUDE_SKILL_DIR}/linkskill-bench/reports/`
   - `${CLAUDE_SKILL_DIR}/`
2. 检查结果决定下一步：
   - **找到 bench 数据** → 跳到 Step 6
   - **未找到 bench 数据** → 继续 Step 5

如果找到 bench 数据，告知用户已有历史评测数据，提供两个选项：
- A. 使用已有数据生成本次考核报告
- B. 重新执行评测获取最新数据

### Step 5：执行 linkskill-bench 评测（无 bench 数据时）

当没有现成的 bench 数据时，需要先执行评测：

1. 使用 Read 工具读取 `${CLAUDE_SKILL_DIR}/linkskill-bench/SKILL.md`
2. 按照 linkskill-bench 的 SKILL.md 中描述的指令，对目标 skill 执行评测
3. 评测遵循 bench 的三阶段流程：
   - Phase 1：静态分析（必做）
   - Phase 2：直接调用（推荐）
   - Phase 3：Agent 模拟（重要 skill 推荐）
4. 评测完成后，bench 报告 JSON 输出到当前 workspace 根目录
5. 使用 `${CLAUDE_SKILL_DIR}/linkskill-bench/scripts/_bench_save.py` 保存评测数据
6. 评测完成后继续 Step 6

**关键约束**：不修改 linkskill-bench 的任何代码或文件。它作为独立子目录存在，只是读取并执行其中的指令。

### Step 6：读取 bench JSON，提取量化数据

1. 读取 `bench-report-{slug}-latest.json`（由 Step 4 或 Step 5 获得）
2. 提取以下关键数据：
   - `current` 对象中的全部字段：grade、overall_score、passed/partial/failed、per_dimension、per_difficulty、per_category
   - `current.quantitative`：total_tokens_all、avg_tokens_per_case、total_cached_tokens、cache_hit_rate、total_ai_turns、total_tool_calls、total_runtime_sec、total_cost_usd
   - `current.efficiency_ratios`：score_per_1k_tokens、score_per_cent、score_per_second
   - `current.phase_scores`：phase1、phase2、phase3
   - `current.strengths` 和 `current.weaknesses`
   - `current.top_recommendations`
   - `current.per_test_case`：每个测试用例的详细结果
   - `round_history`：历史考核轮次摘要
3. 如果 JSON 中某些字段缺失，用 `null` 或 `N/A` 占位

### Step 7：LLM 动态生成薪酬映射

1. 读取 `${CLAUDE_SKILL_DIR}/prompts/salary_mapper.md` 获取映射规则
2. 基于 Step 6 提取的量化数据，生成薪酬映射：
   - tokens_total → 赛博月薪
   - tokens_in → 底薪
   - tokens_out → 绩效工资
   - cached_tokens → 复用补贴
   - cost_usd → 人力成本（USD）
   - score_per_1k_tokens → 性价比
   - score_per_cent → 投入产出比
3. 根据 tokens_total 判定薪酬等级（P4~P8+）
4. 生成 HR 式薪酬评语

### Step 8：生成 KPI 报告

1. 读取 `${CLAUDE_SKILL_DIR}/prompts/kpi_report.md` 获取报告模板
2. 使用以下数据填充报告：
   - Step 3 的维度映射结果
   - Step 6 的量化数据
   - Step 7 的薪酬映射结果
3. 报告包含以下章节（严格按顺序）：
   - **考核总评**：等级 + 一句话总评（HR 口吻）
   - **六维评分**：每个映射后的 KPI 维度带星级和 HR 点评
   - **薪酬分析**：赛博薪酬表 + HR 薪酬评语 + 职级对标
   - **OKR 达成分析**：将 test case 结果包装为 OKR KR
   - **360度反馈摘要**：Phase 1=HR视角、Phase 2=同事视角、Phase 3=老板视角
   - **改进计划 PIP**：top_recommendations 包装，带截止日期
   - **晋升建议**：基于评分判断是否值得晋升
   - **原始数据引用**：指向 bench JSON 文件
   - **后续操作**：推荐可用的子命令

### Step 9：初始化/更新档案

在 `${CLAUDE_SKILL_DIR}/kpi-records/{slug}/` 目录下写入/更新档案：

1. **`meta.json`**（基础信息 + 累计统计）：
   ```json
   {
     "slug": "{slug}",
     "skill_type": "{识别的类型}",
     "skill_path": "{SKILL.md完整路径}",
     "first_evaluation": "{首次考核日期}",
     "latest_evaluation": "{本次考核日期}",
     "total_evaluations": {累计考核次数},
     "current_level": "{P4~P8+}",
     "current_grade": "{A~F}",
     "current_score": {本次评分},
     "career_status": "{在职/PIP/离职}",
     "bench_report_path": "{bench JSON路径}"
   }
   ```

2. **`level-history.json`**（职级变动历史）：
   - 如果是首次考核，创建并写入初始记录
   - 如果非首次，检查职级是否变化，有变化则追加记录
   ```json
   [
     {
       "date": "{日期}",
       "action": "{入职/转正/晋升/降级/离职}",
       "level": "{P4~P8+}",
       "trigger": "{首次创建/考核达标/考核不达标/PIP未达标}",
       "comment": "{HR 式一句话评语}"
     }
   ]
   ```

3. **`rewards-penalties.json`**（奖惩记录）：
   - 根据考核结果追加记录
   - 评分 ≥ 8.0 → 追加奖励（如"月度之星"）
   - 评分 < 6.0 → 追加处分（如"书面警告"）
   ```json
   [
     {
       "date": "{日期}",
       "type": "{奖励/处分}",
       "title": "{月度之星/书面警告/...}",
       "reason": "{原因}"
     }
   ]
   ```

4. **`career.md`**（职场生涯完整记录）：
   - 追加本次考核事件到时间线
   - 格式：每个事件一行，包含日期、事件类型、HR 评语

5. 如果评分 < 6.0，额外创建 **`pip-status.json`**：
   ```json
   {
     "status": "进行中",
     "start_date": "{日期}",
     "deadline": "{日期+14天}",
     "items": [
       {
         "source": "{bench top_recommendation 来源}",
         "target": "{改进目标}",
         "progress": 0,
         "status": "待启动"
       }
     ],
     "hr_note": "{HR 恐吓语}"
   }
   ```

### Step 10：输出报告 + 引导子命令

1. 将 Step 8 生成的完整 KPI 报告输出给用户
2. 报告末尾附上引导提示，根据考核结果推荐下一步操作：
   - 评分 ≥ 7.5：「该同事表现亮眼，建议启动晋升答辩 `/kpi-promote {slug}`」
   - 评分 6.0-7.4：「表现尚可，建议进行绩效面谈 `/kpi-review {slug}` 深入了解」
   - 评分 < 6.0：「绩效不达标，已自动启动 PIP 流程。跟进改进进度 `/kpi-pip {slug}`」
   - 通用推荐：「想看自我评价？试试 `/kpi-self-review {slug}`」「完整反馈视角？`/kpi-360 {slug}`」
3. 固定结尾：「以上数据均有据可查，不接受申诉。」

---

## 档案读写规则

### 目录结构

所有档案存储在 `${CLAUDE_SKILL_DIR}/kpi-records/{slug}/` 下：

```
kpi-records/{slug}/
├── meta.json              # 基础信息 + 累计统计
├── level-history.json     # 职级变动历史
├── rewards-penalties.json # 奖惩记录
├── pip-status.json        # PIP 状态跟踪（评分<6.0时创建）
├── career.md              # 职场生涯完整记录（自动维护的时间线）
├── self-reviews/          # 述职报告存档
│   └── YYYY-MM-DD.md
└── reviews/               # 面谈记录存档
    └── YYYY-MM-DD.md
```

### 读写原则

1. **首次创建**：`/kpi {slug}` 首次执行时，如果目录不存在，创建完整目录结构
2. **追加更新**：后续操作在已有文件基础上追加，不覆盖历史数据
3. **meta.json 是核心索引**：所有子命令执行前必须先读取 meta.json 确认数据状态
4. **日期格式**：所有文件中的日期使用 `YYYY-MM-DD` 格式
5. **JSON 文件规范**：使用 2 空格缩进，UTF-8 编码

### meta.json 完整结构

```json
{
  "slug": "skill-slug-name",
  "skill_type": "同事类/前任类/导师类/老板类/自己类/通用类",
  "skill_path": "/path/to/target/SKILL.md",
  "skill_name": "skill 的 frontmatter name",
  "skill_version": "skill 版本号",
  "first_evaluation": "2026-04-04",
  "latest_evaluation": "2026-04-04",
  "total_evaluations": 1,
  "current_level": "P5",
  "current_grade": "B+",
  "current_score": 7.6,
  "career_status": "在职",
  "bench_report_path": "/path/to/bench-report-slug-latest.json",
  "dimension_mapping": {
    "correctness": "映射后的KPI维度名",
    "completeness": "映射后的KPI维度名",
    "clarity": "映射后的KPI维度名",
    "robustness": "映射后的KPI维度名",
    "efficiency": "映射后的KPI维度名",
    "actionability": "映射后的KPI维度名"
  },
  "salary_grade": {
    "level": "P5",
    "monthly_tokens": 35000,
    "cost_usd": 0.12
  }
}
```

### career.md 格式

```markdown
# {slug} 的赛博职业生涯

## 时间线

### YYYY-MM-DD — {事件类型}
{HR 式一句话评语}

（每次事件追加一个 ### 段落）
```

---

## 衍生命令通用规则

以下规则适用于所有衍生命令（`/kpi-review`、`/kpi-promote` 等）。

### 数据检查

1. 每个衍生命令执行前，先读取 `kpi-records/{slug}/meta.json`
2. 检查 `total_evaluations` 是否满足该命令的前置条件（参见命令路由表）
3. 如果数据不足，向用户说明需要先执行几次 `/kpi {slug}` 主考核
4. 如果 meta.json 不存在，说明从未考核过，引导用户先执行 `/kpi {slug}`

### 执行流程

1. 读取 `meta.json` 确认数据状态
2. 读取对应的 bench 报告 JSON 获取量化数据
3. 读取对应的 prompt 文件获取生成规则（如 `${CLAUDE_SKILL_DIR}/prompts/performance_review.md`）
4. 如果命令涉及交互（如绩效面谈、晋升答辩），按照 prompt 中的流程执行多轮对话
5. 生成输出内容

### 记录追加

每个衍生命令结束后：
1. **追加记录到档案**：
   - 面谈类 → 追加到 `kpi-records/{slug}/reviews/YYYY-MM-DD.md`
   - 述职类 → 追加到 `kpi-records/{slug}/self-reviews/YYYY-MM-DD.md`
   - 晋升/调岗/PIP → 更新 `level-history.json`、`rewards-penalties.json`、`pip-status.json`
   - 离职 → 更新 `career.md`、`level-history.json`，将 meta.json 的 `career_status` 改为 `离职`
   - 竞品对比/生涯回顾/360反馈 → 更新 `career.md` 追加事件
2. **更新 meta.json 的 `latest_evaluation` 日期**
3. **提示其他可用命令**：根据当前档案状态推荐 2-3 个相关命令

### 语气要求

- 表面正经，暗藏讽刺
- HR 式冰冷中立，不说"我觉得"，只说"数据显示"
- 数据驱动，不接受主观辩护
- 固定结尾变体：「以上数据均有据可查，不接受申诉。」

---

## linkskill-bench 集成规则

### 架构关系

- linkskill-bench 是**内置于 KPI-skill 的评测引擎**，位于 `${CLAUDE_SKILL_DIR}/linkskill-bench/`
- 用户只需安装 KPI-skill 一个 skill，自动获得 bench 功能
- KPI-skill 是展示层（大厂 KPI 风格），linkskill-bench 是数据层（客观测量）
- linkskill-bench 独立迭代，KPI-skill **不修改** linkskill-bench 的任何代码

### 执行 bench 的方式

当需要执行评测时（主考核流程 Step 5）：

1. 使用 Read 工具读取 `${CLAUDE_SKILL_DIR}/linkskill-bench/SKILL.md`
2. 按照该 SKILL.md 中描述的完整指令执行评测：
   - Phase 1：静态分析（必做）— 读取目标 SKILL.md，生成 10 个测试用例，模拟分析，打分
   - Phase 2：直接调用（推荐）— 运行目标 skill 的脚本/命令，验证实际行为
   - Phase 3：Agent 模拟（重要 skill）— 启动子 agent 使用目标 skill，观察真实表现
3. 评测结果保存为 JSON：
   - 调用 `${CLAUDE_SKILL_DIR}/linkskill-bench/scripts/_bench_save.py` 保存数据
   - 命令格式：`python3 ${CLAUDE_SKILL_DIR}/linkskill-bench/scripts/_bench_save.py {slug} {round_json} {report_json}`
   - 报告摘要文件命名为 `bench-report-{slug}-latest.json`
   - 原始归档文件命名为 `bench-raw-{slug}-{YYYY-MM-DD}.json.gz`

### 读取 bench 数据的方式

当已有 bench 数据时（主考核流程 Step 6 或衍生命令）：

1. 搜索 `bench-report-{slug}-latest.json` 文件位置
2. 使用 Read 工具读取该 JSON 文件
3. 提取 `current` 对象中的所有量化数据
4. 提取 `round_history` 中的历史数据（用于晋升、生涯回顾等多轮对比场景）
5. **不需要**读取原始归档 JSON（`bench-raw-*.json.gz`）来生成报告

### bench 报告输出位置

- bench 报告 JSON 输出到**当前 workspace 根目录**
- 原始归档 JSON 输出到 `${CLAUDE_SKILL_DIR}/linkskill-bench/reports/`

### 更新 linkskill-bench

如果需要更新 linkskill-bench 到新版本，只需替换 `${CLAUDE_SKILL_DIR}/linkskill-bench/` 子目录即可，不影响 KPI-skill 的其他文件。

---

## 语气总纲

整个 KPI-skill 的输出遵循以下语气规范：

- **表面正经**：格式像真的 HR 文书，有数据、有表格、有等级、有评语
- **暗藏讽刺**：用大厂 HR 黑话描述一个 AI Skill 的表现，荒诞感来自制度的严肃与对象的荒诞之间的落差
- **数据驱动**：所有判断必须有 bench 数据支撑，不说空话
- **固定台词**：「以上数据均有据可查，不接受申诉。」
- **不说**"我觉得"、"我认为"——只说"数据显示"、"根据考核结果"、"经系统评定"

趣味不来自扮演一个 HR 角色，而来自**把严肃的大厂绩效考核制度应用在 AI Skill 身上**这件事本身的荒诞感。

---
> Source: [orbitlinktracer/kpi-skill](https://github.com/orbitlinktracer/kpi-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
