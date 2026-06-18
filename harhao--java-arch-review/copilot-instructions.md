## java-arch-review

> Java 服务端设计架构审查工具，基于资深全栈工程师的视角，审查 Java/Spring Boot 项目的架构设计合理性与工程实践规范性。覆盖数据库索引、SQL 注入防范、配置管理、日志规范、错误处理、代码分层、DRY 原则、认证权限、RESTful 接口设计、参数校验、限流防护、文档沉淀、测试保障、数据存储设计、缓存使用、事务管理、数据库迁移管理、数据库迭代规则等 19 个核心维度。当用户需要审查 Java 服务端代码的架构设计是否合理、工程质量是否达标时使用此 skill。触发关键词：架构审查、设计审查、Java 后端审查、服务端架构、后端 CR、工程规范检查、server arch review、架构合理性。即使用户只是说"帮我看看这个 Java 项目"、"review 一下后端代码"、"检查一下工程质量"也应该触发此 skill。


# Java 服务端设计架构审查工具

> 基于资深全栈工程师的视角，从 19 个核心维度审查 Java 服务端架构设计合理性与工程实践规范性。

---

## 审查维度

共 19 个审查维度，覆盖从编码规范到部署运维的完整链路：

| 序号 | 维度 | 权重 | 参考文件 |
|------|------|------|---------|
| 0 | 需求与编码规范 | 4% | `references/coding-standards.md` |
| 1 | 数据库索引 | 7% | `references/database-index.md` |
| 2 | SQL 注入防范 | 7% | `references/sql-injection.md` |
| 3 | 配置管理 | 5% | `references/config-logging.md` |
| 4 | 可观测性日志 | 5% | `references/config-logging.md` |
| 5 | 错误处理 | 7% | `references/error-handling.md` |
| 6 | 代码分层 | 5% | `references/code-layering.md` |
| 7 | DRY 原则 | 3% | `references/code-layering.md` |
| 8 | 认证与权限 | 7% | `references/security-auth.md` |
| 9 | RESTful 接口 | 5% | `references/api-design.md` |
| 10 | 参数校验与 XSS | 5% | `references/api-design.md` |
| 11 | 限流防护 | 6% | `references/rate-limiting.md` |
| 12 | 文档沉淀 | 3% | `references/quality-testing.md` |
| 13 | 测试保障 | 4% | `references/quality-testing.md` |
| 14 | 数据存储设计 | 6% | `references/data-storage.md` |
| 15 | 缓存使用 | 5% | `references/data-storage.md` |
| 16 | 事务管理 | 5% | `references/data-storage.md` |
| 17 | 数据库迁移管理 | 6% | `references/db-migration.md` |
| 18 | 数据库迭代规则 | 5% | `references/db-migration.md` |

## 违规等级

| 等级 | 标识 | 含义 |
|------|------|------|
| BLOCKER | `[BLOCKER]` | 必须修复，阻塞合入。涉及安全漏洞、数据损坏风险 |
| MAJOR | `[MAJOR]` | 强烈建议修复。涉及性能问题、维护性问题 |
| MINOR | `[MINOR]` | 建议优化。涉及最佳实践、可读性 |

## 参数解析

根据用户输入确定审查模式：
- 无参数 或 "pr" → `--mode pr`（默认，仅扫描 git 变更文件）
- "全量" 或 "full" → `--mode full`（19 维度全量扫描）
- "只看安全" 或 "security" → `--mode focus --dimensions "sql-injection,secrets,security"`
- "quick" 或 "快速" → `--mode quick`（仅 BLOCKER 级别）
- 其他指定维度 → `--mode focus --dimensions "<用户指定>"`

## 审查流程

审查采用**脚本扫描 + AI 深度分析**的混合模式。

#### Phase 1：脚本扫描（确定性检测）

```bash
bash {SKILL_DIR}/scripts/arch-review.sh --project {PROJECT_PATH} --mode {MODE}
```

参数说明：
- `--project`: 目标 Java 项目路径
- `--mode`: 审查模式 (full/pr/focus/quick)，默认 full
- `--dimensions`: focus 模式下指定维度，如 `"sql-injection,security"`

脚本输出 JSON 格式结果，包含：
- `findings[].type = "confirmed"` — 确定性问题，直接采信
- `findings[].type = "needs_ai_review"` — 可疑点，需要 AI 深度判断
- `uncoveredDimensions` — 脚本无法覆盖的维度编号，需要 AI 自主审查

#### Phase 2：AI 深度审查

1. **`needs_ai_review` 项**：读取相关源代码，结合 context 信息判断
2. **`uncoveredDimensions` 维度**：读取 `references/*.md` 规则文件，按规则逐项审查
3. **综合分析**：架构合理性、业务逻辑覆盖度等

#### Phase 3：报告生成

1. 合并 Phase 1 confirmed 结果 + Phase 2 AI 审查结果
2. 计算健康度评分：100 - (BLOCKER × 5) - (MAJOR × 2) - (MINOR × 1)
3. 按 BLOCKER → MAJOR → MINOR 排序输出

## 输出纪律

**必须严格遵守以下规则，控制输出量：**

1. **不要打印脚本的原始 JSON 输出** — 解析后直接用于报告，不展示给用户
2. **不要逐项叙述分析过程** — 不要输出"我正在检查 XXX..."、"发现 XXX 可能有问题..."等中间过程
3. **默认使用摘要报告** — 除非用户明确要求"详细"、"完整报告"、"detail"
4. **MINOR 问题默认折叠** — 摘要模式下仅列出 MINOR 的数量，不展开每一项

## 报告模板

### 默认：摘要报告

```markdown
# 架构审查报告

**健康度: XX/100** | BLOCKER: X | MAJOR: X | MINOR: X

## BLOCKER（必须修复）

1. **[检查项名称]** `文件路径:行号`
   问题: 一句话说明
   建议: 一句话修复方案

2. ...

## MAJOR（建议修复）

1. **[检查项名称]** `文件路径:行号`
   问题: 一句话说明

2. ...

> MINOR 问题共 X 项，输入"展开 MINOR"查看详情。

## 建议
1. 最关键的改进方向（1-3 条）
```

### 用户要求"详细"时：完整报告

```markdown
# Java 服务端设计架构审查报告

## 项目信息
- 项目名称: xxx
- 技术栈: Spring Boot x.x / MyBatis-Plus / MySQL / Redis
- 审查范围: [全量扫描 / 增量扫描 / 指定模块]

## 健康度评分: XX/100

## 审查结果统计
| 等级 | 数量 |
|------|------|
| BLOCKER | X |
| MAJOR | X |
| MINOR | X |

## BLOCKER 问题（必须修复）
[等级] 维度编号 | 检查项名称
位置: 文件路径:行号
问题: 具体违规说明
建议: 修复方案及代码示例

## MAJOR 问题（强烈建议修复）
（同上格式逐项列出）

## MINOR 问题（建议优化）
（同上格式逐项列出）

## 总结与建议
[1-3 条最关键的改进方向]
```

## 审查模式

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **Full** | 19 项维度全量扫描 | 新项目初始化、大版本发布前 |
| **PR** | 仅扫描变更文件涉及的维度 | Pull Request Code Review |
| **Focus** | 指定维度检查 | 专项治理（如安全专项） |
| **Quick** | 仅 BLOCKER 级别 | 快速门禁检查 |

## 脚本覆盖矩阵

| 维度 | 脚本确定性扫描 | AI 深度分析 |
|------|--------------|------------|
| D0 编码规范 | — | CHECK-0001, 0002 |
| D1 数据库索引 | — | CHECK-0101~0105 |
| D2 SQL注入 | CHECK-0201, 0202 | CHECK-0203 |
| D3 配置管理 | CHECK-0301, 0302, 0303 | — |
| D4 日志规范 | CHECK-0401, 0402 | CHECK-0403, 0404 |
| D5 错误处理 | CHECK-0501, 0502 | CHECK-0503, 0504 |
| D6 代码分层 | CHECK-0601, 0602 | CHECK-0603 |
| D7 DRY 原则 | CHECK-0701, 0702 | CHECK-0703 |
| D8 认证权限 | — | CHECK-0801~0803 |
| D9 RESTful 接口 | — | CHECK-0901~0904 |
| D10 参数校验 | CHECK-1001, 1002, 1003 | CHECK-1004 |
| D11 限流防护 | — | CHECK-1101~1108 |
| D12 文档沉淀 | CHECK-1201, 1202 | CHECK-1203 |
| D13 测试保障 | CHECK-1301, 1302, 1303 | CHECK-1304 |
| D14 迭代规范 | — | CHECK-1401, 1402 |
| D15 数据存储 | CHECK-2001~2004 | — |
| D16 缓存使用 | — | CHECK-2101, 2102 |
| D17 事务管理 | — | CHECK-2201, 2202 |
| D18 DB迁移 | CHECK-2301, 2305 | CHECK-2302~2306 |
| D19 DB迭代 | — | CHECK-2401~2408 |

详细检查项清单参见 `references/checklist-index.md`。

---
> Source: [Harhao/java-arch-review](https://github.com/Harhao/java-arch-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
