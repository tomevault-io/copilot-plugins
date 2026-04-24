## twu

> 使用 Claude Code + .claude/ 指令实现需求测试和测试用例生成的完整流程，帮助系统测试工程师提高工作效率。

# TWU - 测试用例生成工作流

## 项目背景

使用 Claude Code + .claude/ 指令实现需求测试和测试用例生成的完整流程，帮助系统测试工程师提高工作效率。

通过 command + skills 编排，自动化处理需求解析、需求评审、测试规划和用例生成等环节。

## 核心原则

### 文档产出规范

所有产出的文档必须遵循：

1. 清晰：结构清晰，一目了然
2. 简洁：不啰嗦，不重复
3. 结构化：格式统一，易于阅读和审核
4. 完整：覆盖所有必要信息

禁止产出冗长、复杂、难以阅读的文档。

---

## 目录结构

```
twu/                           # 项目根目录（Claude Code 运行目录）
├── .claude/                   # 配置目录（所有需求共享）
│   ├── commands/              # 斜杠命令定义
│   │   ├── req-parse.md       # /req-parse - 需求解析
│   │   ├── req-test.md        # /req-test - 需求评审
│   │   ├── req-merge.md       # /req-merge - 需求合并
│   │   ├── testcase-plan.md   # /testcase-plan - 测试规划
│   │   └── testcase-gen.md    # /testcase-gen - 用例生成
│   ├── skills/                # 技能实现
│   │   ├── req-parser/        # 需求解析 Skill
│   │   ├── req-tester/        # 需求评审 Skill
│   │   ├── req-merger/        # 需求合并 Skill
│   │   ├── testcase-planner/  # 测试规划 Skill
│   │   └── testcase-generator/# 用例生成 Skill
│   ├── hooks/                 # 会话钩子
│   └── settings.json          # 配置
├── CLAUDE.md                  # 业务背景和流程说明（本文档）
├── scripts/                   # 辅助脚本
│   └── init.py                # 项目初始化
│
├── 需求1/                     # 需求子目录（通过 init.py 创建）
│   ├── original-requirements/ # 原始需求文档（用户上传）
│   │   ├── PRD.pdf
│   │   ├── 原型图.docx
│   │   └── 接口文档.txt
│   ├── cleaned-requirements/  # 解析后需求
│   │   ├── index.md           # 合并后的完整需求文档
│   │   ├── chunks/            # 解析片段（按原文件名）
│   │   │   ├── PRD.md
│   │   │   ├── 原型图.md
│   │   │   └── 接口文档.md
│   │   ├── assets/            # 图片资源
│   │   │   ├── figure-1.png
│   │   │   └── figure-2.png
│   │   └── issues.md          # 需求问题清单（结构化格式）
│   ├── clarified-requirements/# 完备需求
│   │   ├── index.md           # 回填问题答案后的完整需求
│   │   └── assets/            # 图片资源（从 cleaned 复制）
│   └── test-case/             # 测试用例
│       ├── plan.md            # 测试规划（## ITEM + ### POINT + > 备注）
│       ├── {ITEM}/            # 按测试项分目录
│       │   ├── {POINT1}.md    # 测试用例文件（Markdown Text Protocol v0.2）
│       │   ├── {POINT2}.md
│       │   └── ...
│       ├── all_cases.md       # 所有用例的汇总文件
│       └── export.xlsx        # 导出的 Excel 格式（可选）
│
└── 需求2/                     # 其他需求项目
    └── ...
```

---

## 核心流程

### 准备阶段：初始化需求目录

```bash
uv run scripts/init.py --name "需求名称"
```

**产物**：创建需求子目录及 4 个子文件夹（original-requirements, cleaned-requirements, clarified-requirements, test-case）

**下一步**：将原始需求文档（PDF、Word、图片等）上传到 `需求名称/original-requirements/` 目录

---

### 阶段 1：需求解析 (Parse)

**命令**：`/req-parse`

**触发 Skill**：`req-parser`

**输入**：`需求名称/original-requirements/`（PDF、Word、纯文本、图片等）

**处理流程**：
1. 扫描并分类原始文件（PDF/Word/图片/文本）
2. 使用 Docling 解析 PDF/Word 文档，提取文本、表格和图片
3. 使用多模态能力分析图片内容（原型图、界面截图等），生成详细描述
4. 修复 Docling 解析产生的格式错误（特殊字符、列表序号、标题标记等）
5. 合并所有解析片段到 `cleaned-requirements/index.md`

**产物**：
- `cleaned-requirements/index.md` - 完整的需求文档
- `cleaned-requirements/chunks/` - 各文件的解析片段
- `cleaned-requirements/assets/` - 提取的图片资源

**人工审核重点**：
- [ ] 图片描述是否准确（特别是原型图和界面截图）
- [ ] 表格是否完整（无错行、错列）
- [ ] 格式是否清晰（标题、列表、段落）
- [ ] 有无乱码或缺失内容

**下一步**：人工修正 `index.md`，确认无误后进入评审阶段

---

### 阶段 2：需求评审 (Test)

**命令**：`/req-test`

**触发 Skill**：`req-tester`

**输入**：`cleaned-requirements/index.md`

**处理流程**：
1. 站在测试人员角度分析需求
2. 识别影响测试用例设计的模糊点、缺失点和矛盾点
3. 生成结构化的问题清单（issues.md）

**产物**：`cleaned-requirements/issues.md`（结构化格式，详见下方）

**下一步**：测试工程师在 `issues.md` 中填写 `[产品回答]` 字段

---

### issues.md 格式说明

```markdown
# 需求评审问题清单

> **需求名称**: [需求标题]
> **生成时间**: 2026-01-21 10:30
> **问题总数**: 15 个
> **优先级分布**: P0: 5 | P1: 7 | P2: 3

---

## 问题 1 [前因后果 · critical]

**[位置]** 第3页 - 订单支付功能

**[来源]**
需求文档第3页提到"超时未支付订单自动关闭"，但未说明：
- 超时时长是多少？
- 关闭后库存是否恢复？

**[测试视角]**
作为测试人员，需要理解超时逻辑，才能：
1. 设计边界条件用例（临界时间点）
2. 验证库存回滚的正确性

**[需澄清]**
1. 超时时长是多少分钟？
2. 订单关闭后，库存是立即释放还是延迟释放？

**[回答建议]**
- 超时时长：15分钟 / 30分钟 / 其他
- 库存释放：立即 / 延迟5分钟

**[产品回答]**
<!-- 在此填写答案 -->

---

## 问题 2 [边界定义 · warning]
...
```

**分类枚举**：前因后果、边界定义、异常场景、交互细节、数据状态、性能兼容

**优先级枚举**：critical（不明确无法设计用例）、warning（影响场景识别）、info（补充说明）

**填写规则**：
- 在 `[产品回答]` 下方直接填写答案
- 可以是纯文本、列表或表格
- 必须针对 `[需澄清]` 中的问题逐一回答

---

### 阶段 3：需求完善 (Merge)

**命令**：`/req-merge`

**触发 Skill**：`req-merger`

**输入**：
- `cleaned-requirements/index.md`
- `cleaned-requirements/issues.md`（已填写答案）

**处理流程**：
1. 读取 `issues.md` 中的 `[产品回答]`
2. 将答案回填到需求文档的对应位置
3. 整合为完备需求文档

**产物**：`clarified-requirements/index.md`（完备需求文档）

**人工审核重点**：
- [ ] 所有问题的答案是否已回填
- [ ] 回填内容是否与原文档融合自然
- [ ] 有无遗漏或冲突的信息

**下一步**：确认需求完备后，进入测试规划阶段

**注意**：如需大幅调整，可直接编辑 `clarified-requirements/index.md`，或返回上一步重新执行

---

### 阶段 4：测试规划 (Plan)

**命令**：`/testcase-plan`

**触发 Skill**：`testcase-planner`

**输入**：`clarified-requirements/index.md`

**处理流程**：
1. 识别业务模块（ITEM）：按需求章节或业务实体划分
2. 识别测试场景（POINT）：基于场景法识别独立操作路径
3. 评估风险等级（Critical / High / Medium / Low）
4. 提炼测试关注点（边界值、异常情况、业务规则等）

**产物**：`test-case/plan.md`（结构化测试规划）

**测试规划格式**：

```markdown
# 测试规划

## 登录功能

### 用户名密码登录
> 风险等级: Critical
> 测试关注点: 密码错误、账户锁定、密码长度边界（8-20位）

### 手机验证码登录
> 风险等级: High
> 测试关注点: 验证码过期（60s）、错误次数限制（5次）

### 第三方登录（微信/支付宝）
> 风险等级: Medium
> 测试关注点: 授权失败、token过期处理

## 订单管理

### 创建订单
> 风险等级: Critical
> 测试关注点: 库存扣减、价格计算、并发下单

### 取消订单
> 风险等级: High
> 测试关注点: 库存回滚、退款流程、取消时间限制（30分钟内）
```

**层级说明**：
- **ITEM（测试项）**：业务模块或需求章节，如"登录功能"、"订单管理"
- **POINT（测试点）**：独立的操作路径，如"用户名密码登录"、"手机验证码登录"
- **备注（> 风险等级 / 测试关注点）**：规划阶段的思考，用于指导用例生成

**人工审核重点**：
- [ ] 测试项划分是否合理（无遗漏、无重复）
- [ ] 测试点是否是独立场景（而非同一场景的不同数据）
- [ ] 风险等级评估是否准确
- [ ] 测试关注点是否抓住了关键逻辑

**下一步**：确认规划合理后，进入用例生成阶段

**注意**：如果测试点划分不合理，直接编辑 `plan.md` 调整

---

### 阶段 5：用例生成 (Generate)

**命令**：`/testcase-gen`

**触发 Skill**：`testcase-generator`

**输入**：
- `test-case/plan.md`
- `clarified-requirements/index.md`
- `CLAUDE.md`（业务背景）

**处理流程**：
1. 读取测试规划（plan.md）
2. 为每个 POINT 创建目录结构（`test-case/{ITEM}/`）
3. 逐个 POINT 生成测试用例：
   - 识别输入项
   - 划分等价类和边界值
   - 设计正向和反向用例
   - 即时校验格式和质量
4. 合并所有用例到 `all_cases.md`
5. 执行全局校验和重复检测
6. 生成质量报告

**产物**：
- `test-case/{ITEM}/{POINT}.md` - 各测试点的用例文件
- `test-case/all_cases.md` - 所有用例汇总
- 质量报告（等价类覆盖率、边界值覆盖率、用例数量统计）

**测试用例格式（Markdown Text Protocol v0.2）**：

```markdown
## [P1] 验证有效用户名和密码登录成功
[测试类型] 功能
[前置条件] 已注册用户test_user，密码Test1234
[测试步骤] 1. 输入用户名test_user，输入密码Test1234，点击登录。2. 验证跳转到首页并显示用户名
[预期结果] 1. 登录请求成功。2. 跳转到/home页面，顶部显示'欢迎，test_user'

## [P3][反向] 验证用户名为空时登录失败
[测试类型] 功能
[前置条件] 进入登录页
[测试步骤] 1. 不输入用户名，输入密码Test1234，点击登录。2. 验证提示'用户名不能为空'
[预期结果] 1. 登录请求被拒绝。2. 停留在登录页，用户名输入框下方显示红色提示'用户名不能为空'
```

**优先级枚举**：P1（核心正向）、P2（基本正向）、P3（核心异常）、P4（边界条件）、P5（低频场景）

**测试类型枚举**：功能、接口、性能、安全、兼容、易用、安装部署、可靠、本地化、可维护、可扩展、配置

**人工审核重点**：
- [ ] 用例是否覆盖测试点的关键逻辑
- [ ] 测试数据是否具体（非占位符）
- [ ] 预期结果是否可验证（非模糊描述）
- [ ] 用例数量是否合理（基于实际场景复杂度）
- [ ] 有无重复用例

**下一步**：确认用例质量后，导出为 Excel 或直接使用 Markdown 格式

---

### 阶段 6：导出和交付

**Excel 导出（可选）**：

```bash
uv run .claude/skills/testcase-generator/scripts/to_excel.py "test-case/" -o "test-case/export.xlsx"
```

**产物**：`test-case/export.xlsx`（包含所有测试用例的 Excel 文件）

**交付物**：
- `test-case/plan.md` - 测试规划
- `test-case/all_cases.md` - Markdown 格式用例
- `test-case/export.xlsx` - Excel 格式用例（可选）

**流程结束**

---

## 常见问题处理

### 1. 文档解析失败

**症状**：Docling 报错、乱码、格式错乱

**处理方式**：
- **PDF 加密**：提示用户提供解密密码或使用解密工具
- **文件损坏**：跳过该文件，记录错误，继续处理其他文件
- **编码问题**：自动尝试 UTF-8、GBK、GB2312 编码
- **Docling 失败**：自动降级使用 PyPDF2 或 python-docx

**人工介入**：如果自动处理失败，可手动转换文档为 `.txt` 或 `.md` 格式后重新执行

---

### 2. 需求问题过多

**症状**：生成的问题数量超过预期（例如 40+ 个问题）

**可能原因**：
- 需求文档确实不完备（缺失关键信息）
- 需求文档是初稿或草稿
- 需求文档包含大量技术细节

**处理方式**：
- 人工筛选 critical 和 warning 优先级的问题
- info 级别问题可以选择性忽略
- 如果问题质量不高，提供反馈以便优化 req-tester Skill

---

### 3. 测试点数量异常

**症状**：生成的测试点过多（例如 100+ 个）或过少（例如 5 个）

**可能原因**：
- **过多**：POINT 粒度过细，将不同数据识别为不同场景
- **过少**：POINT 粒度过粗，遗漏了关键场景

**处理方式**：
- 直接编辑 `test-case/plan.md` 调整
- 检查是否混淆了 POINT（场景）和 CASE（数据）
- 参考"层级说明"重新划分

---

### 4. 用例数量膨胀

**症状**：单个 POINT 生成 20+ 个用例

**可能原因**：
- 输入项过多且组合复杂
- 每个等价类都生成了独立用例（不必要）

**处理方式**：
- 人工删除冗余用例（相似异常可保留代表性用例）
- 检查是否存在过度细分的等价类
- 保留核心流程和主要异常即可

---

### 5. 格式校验失败

**症状**：validate.py 报错

**常见错误**：
- 优先级不是 P1-P5
- 测试类型不在 12 种枚举中
- 步骤编号不连续
- 步骤与预期结果数量不一致

**处理方式**：
- 根据错误提示修正对应的 `.md` 文件
- 重新运行校验脚本确认修复

---

### 6. 迭代和回退

**场景**：人工审核发现严重问题，需要重新执行

**处理方式**：
- **轻微问题**：直接编辑产物文件（`index.md`、`plan.md`、用例文件等）
- **严重问题**：删除或备份当前产物，重新执行对应命令
- **AI 会自动读取最新输入**：无需担心缓存问题

**建议**：每个阶段完成后，备份产物文件，便于回退

---

## 核心设计理念

### 1. 关注测试必要性，而非文档完美性

**需求评审（req-test）** 的目标是：
- ✅ 识别影响测试用例设计的模糊点
- ✅ 识别导致测试结果不确定的缺失点
- ❌ 不纠结文档的形式和格式
- ❌ 不追求业务背景的完整性

### 2. 基于场景法识别测试点，而非等价类

**测试规划（testcase-plan）** 的逻辑是：
- **ITEM**：业务模块或需求章节
- **POINT**：独立的操作路径（场景法）
- **CASE**：具体的测试数据（等价类和边界值）

**错误示例**：将"密码长度 <8"、"密码长度 8-20"、"密码长度 >20"拆分为 3 个 POINT
**正确示例**：合并为 1 个 POINT "密码长度验证"，在 CASE 中覆盖 3 种等价类

### 3. 策略指导，而非指令控制

**用例生成（testcase-gen）** 遵循：
- ✅ 提供判断标准（测试价值评估、覆盖判断）
- ✅ 提供思考框架（5 步思维链）
- ❌ 不强制用例数量（"一个 POINT 必须生成 5-8 个 CASE"）
- ❌ 不机械执行（"每个等价类必须生成 1 个用例"）

**核心原则**：AI 根据场景复杂度自适应调整，而非死板遵循规则

### 4. 上下文传递，而非一对一映射

**备注（> 风险等级 / 测试关注点）** 的作用是：
- ✅ 传递规划阶段的思考（"这个场景有什么容易遗漏的逻辑"）
- ✅ 提醒生成阶段注意关键点（"需要覆盖库存扣减"）
- ❌ 不强制一对一映射（"测试关注点: 库存不足 → 必须生成：库存不足用例"）

**比喻**：备注是"接力棒"，传递信息但不限制发挥空间

---

## 产物格式规范汇总

| 阶段 | 产物文件 | 格式规范 | 关键字段 |
|------|----------|----------|----------|
| 需求解析 | `cleaned-requirements/index.md` | Markdown | 标题、段落、表格、图片引用 |
| 需求评审 | `cleaned-requirements/issues.md` | 结构化 Markdown | `## 问题 N [分类 · 优先级]`<br>`[位置] [来源] [测试视角] [需澄清] [回答建议] [产品回答]` |
| 需求完善 | `clarified-requirements/index.md` | Markdown | 标题、段落、表格、图片引用 |
| 测试规划 | `test-case/plan.md` | 结构化 Markdown | `# 测试规划`<br>`## ITEM`<br>`### POINT`<br>`> 风险等级:`<br>`> 测试关注点:` |
| 用例生成 | `test-case/{ITEM}/{POINT}.md` | Markdown Text Protocol v0.2 | `## [P1-P5] 用例标题`<br>`[测试类型]`<br>`[前置条件]`<br>`[测试步骤]`<br>`[预期结果]` |
| 汇总导出 | `test-case/all_cases.md` | Markdown Text Protocol v0.2 | 所有用例合并 |
| Excel 导出 | `test-case/export.xlsx` | Excel | 表格（用例ID、标题、类型、优先级、步骤、预期结果等） |

---

## 辅助脚本说明

### 1. 初始化脚本

```bash
uv run scripts/init.py --name "需求名称"
```

创建需求目录结构。

### 2. 测试规划解析器

```bash
uv run .claude/skills/testcase-planner/scripts/parse_plan.py test-case/plan.md
```

验证 `plan.md` 格式是否符合规范。

### 3. 测试用例校验器

```bash
# 全局校验
uv run .claude/skills/testcase-generator/scripts/validate.py "test-case/"

# 全局校验 + 重复检测
uv run .claude/skills/testcase-generator/scripts/validate.py "test-case/" --check-duplicates
```

校验用例格式、重复性、覆盖率。

### 4. Excel 导出器

```bash
uv run .claude/skills/testcase-generator/scripts/to_excel.py "test-case/" -o "test-case/export.xlsx"
```

将 Markdown 用例导出为 Excel。

### 5. 需求问题校验器

```bash
uv run .claude/skills/req-tester/scripts/validate_issues.py cleaned-requirements/issues.md
```

校验 `issues.md` 格式是否符合规范。

---

## 总结

TWU 提供了一套完整的测试用例生成工作流，核心特点是：

1. **AI + 人工协作**：每个阶段 AI 自动处理，人工审核关键决策
2. **结构化产物**：所有产物都有明确的格式规范，便于解析和校验
3. **策略指导**：不死板控制数量，而是提供判断标准和思考框架
4. **上下文传递**：通过备注和分阶段设计，保持信息流动
5. **质量保障**：内置校验脚本和检查清单，确保产物质量

**适用场景**：
- 系统测试用例设计
- 需求评审和完善
- 测试规划和覆盖率分析

---
> Source: [chyax98/twu](https://github.com/chyax98/twu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
