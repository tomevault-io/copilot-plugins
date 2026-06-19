## trusted-skill-refinery

> Trusted Skill Refinery

# Trusted Skill Refinery

---

## English

### Name

Trusted Skill Refinery

### Purpose

This project provides a safety-first pipeline for **fetching, auditing, sanitizing, validating, and publishing** clean AI agent skills from untrusted external sources. It processes third-party skills through controlled fetch, isolation, static audit, human review, sanitization planning, non-executable candidate generation, validation, and admission.

**v0.2-dev** adds a controlled auto-rewrite engine for low-risk prompt/document skills and a high-risk routing system for scraper/API/browser skills.

### Core Principle: Dark Skill Assumption

The project is based on the **Dark Skill Assumption**: every external AI agent skill should be treated as potentially unsafe until it has passed controlled fetching, isolation, audit, risk scoring, human review, validation, and admission.

This does not mean every skill is malicious. It means no external skill should be trusted by default.

Downloading a skill does not mean trusting it. Auditing does not mean admitting it to your library. Sanitization does not mean it is safe to execute. Admission requires validation and human approval.

### Version Scope

#### v1.0 (Current Release)

v1.0 focuses on the manual sanitization workflow. It supports:

- Controlled fetching from GitHub archive URLs
- Safe archive inspection with path traversal protection
- Raw skill auditing (static, dependency, instruction)
- Risk decision (R1-R4 classification)
- Rewrite/sanitization planning
- Non-executable candidate generation
- Static validation
- Sanitized library admission
- Optional non-executable AgentFactory export packages

v1.0 does **not** provide fully automatic executable rewriting. Rewrite support in v1.0 is conservative, planning-oriented, and non-executable by default.

#### v0.2-dev (Development Branch)

v0.2-dev introduces two major new systems on top of v1.0:

**Controlled Auto-Rewrite Engine** (for low-risk prompt/document skills):
- Rule-based detection and removal of dangerous instruction patterns
- Rewrite diff — auto-generated comparison between raw and rewritten skill
- Functional preservation report — quantifies safe content retained vs removed
- Removed risk report — documents what was removed and why
- Safety regression gate — automated check that rewrite did not introduce new risks
- Streamlined human review for auto-rewritten candidates
- Non-executable sanitized library admission

**High-Risk Routing System** (for scraper/API/browser skills):
- High-risk skill type detection and classification
- Static audit (19 checks per skill covering API keys, network, browser, credentials)
- Risk routing: quarantine or manual wrapper planning
- 6 cross-cutting policies: sandbox, API key handling, rate limiting, domain allowlisting, data staging, approval gates (10 gates)
- Future wrapper roadmap (stages H1-H5 from design through controlled test)

v0.2-dev does **not** support:
- Automatic rewriting of scraper/API/browser/credential skills
- Executable wrapper generation
- Execution of raw or rewritten skills
- Dependency installation
- API key reading
- Automatic AgentFactory export

### Low-Risk Route (prompt/document skills)

```
Prompt/Document Skill
  → Controlled Intake
  → Static Audit
  → Auto-Rewrite Candidate
  → Rewrite Diff
  → Functional Preservation Report
  → Removed Risk Report
  → Safety Regression Gate
  → Human Review
  → Admission Package
  → Final Human Approval
  → Non-Executable Sanitized Library Entry
```

This route is for skills that are purely instructional — no scripts, no dependencies, no network requirements, no API keys. The auto-rewrite engine removes dangerous instruction patterns (prompt injection, hidden instructions, secret extraction, local file scan, arbitrary network access) while preserving safe content.

**Current v0.2-dev result**: 2 synthetic examples admitted (prompt_skill_001_clean_v0_2, document_skill_001_clean_v0_2). Both non-executable, all permissions false.

### High-Risk Route (scraping/API/browser/crawler skills)

```
Scraping/API/Browser/Crawler Skill
  → Controlled Intake
  → Static Audit (19 checks)
  → Risk Routing
  → Quarantine (R4: browser automation, credential management)
  → OR Manual Wrapper Planning (R3: API scrapers)
  → Sandbox / API Key / Domain Allowlist / Rate Limit / Data Staging Policy
  → No Executable Output by Default
```

This route applies when a skill requires network access, API keys, browser automation, or credential management. These skills are **never** auto-rewritten. They are either quarantined permanently (browser, credential) or deferred for manual wrapper planning (API scrapers).

**Current v0.2-dev result**: 5 real Firecrawl skills stress-tested. 0/5 allowed auto-rewrite. 2 quarantined (browser automation, credential management). 3 deferred for manual wrapper planning.

### What Must Never Be Automatic

The following must **never** be automated, in any version:

- Execution of raw or rewritten skills
- Dependency installation
- API key reading or storage
- Network access during audit or rewrite
- Browser session access
- Credential file (.env) writes
- AgentFactory export without final human approval
- Sanitized library admission without validation + human approval
- Trust in auto-rewrite output without human review

### Current Sanitized Library Entries

| # | Skill ID | Version | Type | Executable | Permissions |
|---|----------|---------|------|------------|-------------|
| 1 | firecrawl_build_scrape_sanitized_v0_1 | v1.0 | Manual, real Firecrawl skill | false | all false |
| 2 | prompt_skill_001_clean_v0_2 | v0.2 | Auto-rewritten, synthetic prompt skill | false | all false |
| 3 | document_skill_001_clean_v0_2 | v0.2 | Auto-rewritten, synthetic document skill | false | all false |

All entries are non-executable. All permission flags (`execution_allowed`, `network_access_allowed`, `api_key_read_allowed`) are `false`.

### Human Approval Gates

- **v1.0**: 3 human review gates (HR-0001 through HR-0004) — raw audit review, rewrite planning review, final admission approval
- **v0.2-dev low-risk**: Streamlined to 2 gates — human review after safety regression, final admission approval
- **v0.2-dev high-risk**: 10 approval gates for any future wrapper implementation (design → sandbox → API key → mock → dry-run → network test → data staging → log redaction → legal → final human)

### Safety Rules

These rules are **mandatory and never overridden**:

- **Default no execution** — Raw skills, candidates, and wrapper plans are never executed.
- **Default no network** — The pipeline makes no outbound network calls (except controlled fetch in Phase 8).
- **Default no dependency installation** — No package manager is ever invoked.
- **Default no secret access** — No .env, token, cookie, ssh key, or Keychain access.
- **Raw skill stays in raw inbox** — `05_raw_skill_inbox/` is isolated; no execution, no library write.
- **Downloaded content stays isolated** — `15_skill_fetcher/02_download_cache/` and `03_extracted_raw/` are quarantined.
- **Sanitized library admission requires validation + human approval**.
- **AgentFactory export requires separate approval**.
- **High-risk skills are never auto-admitted**.

### Safety Warning

Do not execute raw skills. Do not install dependencies from raw skills. Do not read or expose API keys. Do not trust rewritten skills without validation and human review. Do not treat non-executable sanitized templates as runnable tools. Do not assume auto-rewrite output is safe without human review.

### Future Roadmap

#### v0.2-dev (current)

- Complete auto-rewrite for prompt/document skills ✓
- High-risk scraper/API/browser routing ✓
- Awaiting real prompt/document skills for live intake test

#### v2.0 (planned)

- Controlled auto-rewrite with rewrite diff reports
- Functional preservation reports
- Safety regression checks
- Prompt/document skill rewrite
- API wrapper planning
- Sandbox dry-run validation
- Data staging pipeline

Even in v2.0, sanitization may reduce or constrain some original functionality in order to remove unsafe behavior, reduce permissions, or prevent uncontrolled execution.

### Risk Levels

| Level | Description | Recommended Action |
|-------|-------------|-------------------|
| R0/R1 | Low risk — safe instructional content | Approve as reference |
| R2 | Medium — requires controlled review | Human review required |
| R3 | Elevated — API key, network, or dependency risk | Human review + rewrite planning required |
| R4 | High risk — execution, installation, or credential risk | Quarantine — never admit |

No risk level permits automatic admission.

### Required Inputs

- **Source URL** or local zip path
- **Declared skill name** — identifier for tracking
- **Intended use** — what you plan to do with the sanitized skill
- **Whether network/API/key access is expected** — affects risk classification
- **Target library/export preference** — sanitized library only, or also AgentFactory export

### Output Artifacts

- Source manifest (download record with SHA256 and URL)
- Inventory (candidate skill list)
- Audit findings (static, dependency, instruction)
- Permission manifest (aggregated constraints)
- Risk decision (R1-R4 with rationale)
- Human review record (CSV queue with decision)
- Rewrite planning package (functional spec, risk plan, wrapper design)
- Validation report (static validation results)
- Sanitized library entry (admitted skill in `10_sanitized_skill_library/`)
- Export package (AgentFactory export design, optional)

### Example Case: Firecrawl Build/Scrape

1. **Source**: Firecrawl Skills GitHub archive (public)
2. **Selected skill**: `firecrawl-build-scrape` — single SKILL.md, no scripts, no dependencies
3. **Raw audit**: R3 — API key dependency, external network service required. No secret theft detected.
4. **Rewrite planning**: Designed non-executable wrapper with constraint inheritance.
5. **Candidate package**: 9 files, 0 executables, all permission flags `false`.
6. **Validation**: 55/55 checks PASS.
7. **Admission**: ADMIT-0001 into sanitized library.
8. **Export**: Non-executable reference/template export to AgentFactory.

### Strict Boundaries

- Never execute untrusted skill content.
- Never install raw skill dependencies.
- Never read secrets (.env, token, key, cookie).
- Never auto-approve trusted admission.
- Never treat non-executable sanitized templates as runnable tools.
- Never bypass human review gates.
- Never write to AgentFactory without explicit approval.
- Never auto-admit high-risk skills to the sanitized library.

---

## 中文

### 名称

Trusted Skill Refinery（受信任 Skill 提炼系统）

### 用途

本项目提供一个安全优先的管线，用于从不受信任的外部来源**获取、审查、清洗、验证和发布**干净的 AI Agent Skill。它对第三方 Skill 进行受控获取、隔离、静态审查、人工复核、清洗规划、非执行候选包生成、验证和入库。

**v0.2-dev** 新增了面向低风险 prompt/document Skill 的受控自动改写引擎，以及面向 scraper/API/browser Skill 的高风险路由系统。

### 核心原则：黑暗 Skill 假设

本项目基于“黑暗 Skill 假设”：所有外部 AI Agent Skill 在经过受控获取、隔离、审查、风险分级、人工复核、验证和入库之前，都应默认视为存在潜在安全风险。

这并不代表每个 Skill 都是恶意的，而是强调：任何外部 Skill 都不应该默认被信任。

下载一个 Skill 不代表信任它。审查不代表将其收入你的库。清洗不代表它可以安全执行。入库需要验证和人工批准。

### 版本范围

#### v1.0（当前发布版）

v1.0 重点是手动清洗流程。它支持：

- 从 GitHub archive URL 受控获取
- 带路径遍历保护的安全 archive 检查
- 原始 Skill 审查（静态、依赖、指令）
- 风险决策（R1-R4 分级）
- 清洗/改写规划
- 非执行候选包生成
- 静态验证
- Sanitized library 入库
- 可选的非执行 AgentFactory 导出包

v1.0 不提供完整的自动安全可执行改写能力。v1.0 的改写能力是保守的、偏规划型的，并且默认生成非执行候选包。

#### v0.2-dev（开发分支）

v0.2-dev 在 v1.0 基础上引入了两个主要新系统：

**受控自动改写引擎**（针对低风险 prompt/document Skill）：
- 基于规则的危险指令模式检测和移除
- 改写差异报告 — 自动生成原始与改写后 Skill 的对比
- 功能保留报告 — 量化保留的安全内容与移除内容
- 风险移除报告 — 记录移除了什么以及为什么
- 安全回归门 — 自动检查改写是否引入新风险
- 简化自动改写候选的人工复核流程
- 非执行 sanitized library 入库

**高风险路由系统**（针对 scraper/API/browser Skill）：
- 高风险 Skill 类型检测和分类
- 静态审查（每 Skill 19 项检查，覆盖 API key、网络、浏览器、凭证）
- 风险路由：隔离或手动 wrapper 规划
- 6 项跨领域策略：沙盒、API key 处理、速率限制、域名白名单、数据暂存、审批门控（10 道门）
- 未来 wrapper 路线图（从设计到受控测试的 H1-H5 阶段）

v0.2-dev **不支持**：
- 自动改写 scraper/API/browser/credential Skill
- 生成可执行 wrapper
- 执行原始或改写后的 Skill
- 安装依赖
- 读取 API key
- 自动导出到 AgentFactory

### 低风险路线（prompt/document Skill）

```
Prompt/Document Skill
  → 受控接收
  → 静态审查
  → 自动改写候选
  → 改写差异报告
  → 功能保留报告
  → 风险移除报告
  → 安全回归门
  → 人工复核
  → 入库包
  → 最终人工批准
  → 非执行 Sanitized Library 条目
```

此路线适用于纯指令型 Skill — 无脚本、无依赖、无网络需求、无 API key。自动改写引擎移除危险指令模式（prompt 注入、隐藏指令、密钥窃取、本地文件扫描、任意网络访问），同时保留安全内容。

**当前 v0.2-dev 结果**：2 个合成示例已入库（prompt_skill_001_clean_v0_2、document_skill_001_clean_v0_2）。均为非执行，所有权限标志为 false。

### 高风险路线（scraping/API/browser/crawler Skill）

```
Scraping/API/Browser/Crawler Skill
  → 受控接收
  → 静态审查（19 项检查）
  → 风险路由
  → 隔离（R4：浏览器自动化、凭证管理）
  → 或手动 Wrapper 规划（R3：API scraper）
  → 沙盒 / API Key / 域名白名单 / 速率限制 / 数据暂存策略
  → 默认无可执行输出
```

此路线适用于需要网络访问、API key、浏览器自动化或凭证管理的 Skill。这些 Skill **永不**被自动改写。它们要么被永久隔离（浏览器、凭证），要么被延迟至手动 wrapper 规划（API scraper）。

**当前 v0.2-dev 结果**：5 个真实 Firecrawl Skill 完成压力测试。0/5 允许自动改写。2 个隔离（浏览器自动化、凭证管理）。3 个延迟至手动 wrapper 规划。

### 绝不能自动化的操作

以下操作在任何版本中都**绝不能**自动化：

- 执行原始或改写后的 Skill
- 安装依赖
- 读取或存储 API key
- 审查或改写期间的网络访问
- 浏览器会话访问
- 凭证文件（.env）写入
- 未经最终人工批准的 AgentFactory 导出
- 未经验证 + 人工批准的 sanitized library 入库
- 未经人工复核就信任自动改写输出

### 当前 Sanitized Library 条目

| # | Skill ID | 版本 | 类型 | 可执行 | 权限 |
|---|----------|---------|------|------------|-------------|
| 1 | firecrawl_build_scrape_sanitized_v0_1 | v1.0 | 手动，真实 Firecrawl Skill | false | 全部 false |
| 2 | prompt_skill_001_clean_v0_2 | v0.2 | 自动改写，合成 prompt skill | false | 全部 false |
| 3 | document_skill_001_clean_v0_2 | v0.2 | 自动改写，合成 document skill | false | 全部 false |

所有条目均不可执行。所有权限标志（`execution_allowed`、`network_access_allowed`、`api_key_read_allowed`）均为 `false`。

### 人工审批门

- **v1.0**：3 道人工复核门（HR-0001 至 HR-0004）— 原始审查复核、改写规划复核、最终入库批准
- **v0.2-dev 低风险**：简化为 2 道门 — 安全回归后的人工复核、最终入库批准
- **v0.2-dev 高风险**：10 道审批门用于任何未来的 wrapper 实施（设计 → 沙盒 → API key → 模拟 → dry-run → 网络测试 → 数据暂存 → 日志脱敏 → 法律 → 最终人工）

### 安全规则

以下规则是**强制性的，永不推翻**：

- **默认不执行** — 原始 Skill、候选包和 wrapper 计划均不执行。
- **默认不联网** — 管线不发起出站网络调用（Phase 8 受控获取除外）。
- **默认不安装依赖** — 永不调用包管理器。
- **默认不访问密钥** — 不访问 .env、token、cookie、ssh key 或 Keychain。
- **原始 Skill 保留在 raw inbox** — `05_raw_skill_inbox/` 是隔离的；不执行、不写入库。
- **下载内容保持隔离** — `15_skill_fetcher/02_download_cache/` 和 `03_extracted_raw/` 是隔离的。
- **Sanitized library 入库需要验证 + 人工批准**。
- **AgentFactory 导出需要单独批准**。
- **高风险 Skill 永不自动入库**。

### 安全警告

不要执行原始 Skill。不要安装原始 Skill 的依赖。不要读取或暴露 API key。不要在未经验证和人工复核前信任任何改写后的 Skill。不要把非执行型 sanitized template 当成可运行工具使用。不要假设自动改写输出在未经人工复核前是安全的。

### 未来路线图

#### v0.2-dev（当前）

- 完成 prompt/document Skill 自动改写 ✓
- 高风险 scraper/API/browser 路由 ✓
- 等待真实 prompt/document Skill 进行实战接收测试

#### v2.0（计划）

- 受控自动改写，含改写差异报告
- 功能保留报告
- 安全回归检查
- Prompt/文档型 Skill 改写
- API wrapper 规划
- 沙盒 dry-run 验证
- 数据暂存管线

即使在 v2.0，清洗和改写也可能减少或限制原 Skill 的部分功能，因为系统会优先删除危险行为、降低权限，或防止不受控执行。

### 风险等级

| 等级 | 描述 | 建议操作 |
|-------|-------------|-------------------|
| R0/R1 | 低风险 — 安全的教学内容 | 批准作为参考 |
| R2 | 中等 — 需要受控审查 | 需要人工复核 |
| R3 | 升高 — API key、网络或依赖风险 | 需要人工复核 + 改写规划 |
| R4 | 高风险 — 执行、安装或凭证风险 | 隔离 — 永不入库 |

任何风险等级都不允许自动入库。

### 必要输入

- **来源 URL** 或本地 zip 路径
- **声明的 Skill 名称** — 用于追踪的标识符
- **预期用途** — 你计划用清洗后的 Skill 做什么
- **是否预期需要网络/API/key 访问** — 影响风险分类
- **目标 library/导出偏好** — 仅 sanitized library，还是也包括 AgentFactory 导出

### 输出产物

- 来源清单（含 SHA256 和 URL 的下载记录）
- 清单（候选 Skill 列表）
- 审查发现（静态、依赖、指令）
- 权限清单（聚合的约束）
- 风险决策（R1-R4 含理由）
- 人工复核记录（含决策的 CSV 队列）
- 改写规划包（功能规格、风险计划、wrapper 设计）
- 验证报告（静态验证结果）
- Sanitized library 条目（`10_sanitized_skill_library/` 中的入库 Skill）
- 导出包（AgentFactory 导出设计，可选）

### 示例案例：Firecrawl Build/Scrape

1. **来源**：Firecrawl Skills GitHub archive（公开）
2. **选择的 Skill**：`firecrawl-build-scrape` — 单个 SKILL.md，无脚本，无依赖
3. **原始审查**：R3 — 依赖 API key，需要外部网络服务。未检测到密钥窃取。
4. **改写规划**：设计带约束继承的非执行 wrapper。
5. **候选包**：9 个文件，0 个可执行文件，所有权限标志为 `false`。
6. **验证**：55/55 项检查通过。
7. **入库**：ADMIT-0001 收入 sanitized library。
8. **导出**：非执行参考/模板导出到 AgentFactory。

### 严格边界

- 永远不要执行不受信任的 Skill 内容。
- 永远不要安装原始 Skill 的依赖。
- 永远不要读取密钥（.env、token、key、cookie）。
- 永远不要自动批准受信任入库。
- 永远不要把非执行型 sanitized template 当成可运行工具。
- 永远不要绕过人工复核门。
- 永远不要在未经明确批准的情况下写入 AgentFactory。
- 永远不要将高风险 Skill 自动收入 sanitized library。

---
> Source: [rocq8811910/trusted-skill-refinery](https://github.com/rocq8811910/trusted-skill-refinery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
