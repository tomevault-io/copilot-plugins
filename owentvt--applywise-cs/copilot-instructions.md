## applywise-cs

> ApplyWise CS 是一个面向 CS / AI / DS / SE 留学申请者的 Android 智能选校助手。当前仓库已形成具备上线级工程设计的完整作品候选；项目不要求实际公开上线或持续运营。

# AGENTS.md

## 项目目标

ApplyWise CS 是一个面向 CS / AI / DS / SE 留学申请者的 Android 智能选校助手。当前仓库已形成具备上线级工程设计的完整作品候选；项目不要求实际公开上线或持续运营。

开始工作前必须完整阅读：

1. `docs/codex_handoff.md`
2. `docs/sdd.md`
3. `docs/module_plan.md`
4. `docs/architecture.md`
5. `README.md`

若旧文档与 `docs/codex_handoff.md` 对 Module 7+ 的描述冲突，以用户在 2026-08-01 批准的最新 `docs/codex_handoff.md` 为准，并在当前文档模块内同步消除冲突。

## 当前架构事实

- Android：Kotlin、Jetpack Compose、Navigation Compose、Coroutines/Flow、Room、KSP、JUnit 4。
- 项目数据：远端 PostgreSQL 已发布快照 → Room cache → 24 个 `FakePrograms` 内置回退。
- 用户画像：Room `user_profile`，可选账号同步默认关闭。
- 收藏：Room `favorite_program`，支持离线墓碑与可选同步。
- 推荐：`ProgramMatchEngine` 端侧执行八维确定性评分。
- Advisor：Android 已通过供应商无关 HTTPS Gateway 接入真实 AI，并保留端侧 `MockAiAdvisor` 显式降级。
- 独立 `backend/` 模块化单体：Node.js、TypeScript strict、Fastify、Zod、PostgreSQL/Drizzle、AI Gateway、项目库、可选账号同步、管理治理和脱敏运行摘要。
- Android 已有网络权限、严格 HTTPS、匿名 AI 会话、远程项目缓存和可选账号同步；现有 public-demo 只开放匿名 AI 与只读项目库。

## 已批准的演进方向

- Android 使用供应商无关的 ApplyWise API，不直连模型厂商。
- AI Gateway 首选国内可直连、当前免费且开放权重的智谱 GLM-4.7-Flash，但模型供应商和模型名不得成为 Android 产品契约。
- 后端纳入产品化规划，采用模块化单体优先，逐步加入 AI Gateway、项目库、账号同步、管理和运维能力。
- 保留本地项目 fallback、Room、`ProgramMatchEngine` 和 Mock Advisor，维持离线闭环。
- `ProgramMatchEngine` 始终是匹配分和分组的事实来源；LLM 只负责解释、比较、总结和行动建议。
- 未登录用户仍可使用本地核心功能；账号主要用于同步。
- 默认不持久化聊天正文。

## 模块顺序与验收门禁

Module 0–14 已完成并验收。Module 14 的代码、自动门禁与核心真机验收均已通过；私有账号在独立 SCF + Neon 验收环境完成模拟器上传、手机拉取和 Room 落库。用户于 2026-08-10 决定不执行离线、冲突、会话隔离及数据生命周期压力矩阵；这些场景如实记录为“主动豁免”，不表述为测试通过。正式签名和分发仍留到最终包装阶段。历史顺序如下：

1. Module 7：综合契约与 Android 架构准备。
2. Module 8：后端平台基础。
3. Module 9：AI Gateway 与国内模型。
4. Module 10：Android 真实 AI 体验。
5. Module 11：远程项目库与离线缓存。
6. Module 12：账号、画像与收藏同步。
7. Module 13：管理后台与数据治理。
8. Module 14：Production-ready 加固与最终交付。

规则：

- 一次只执行用户明确批准的一个模块。
- 未收到用户明确验收，不得开始下一模块。
- 每个模块开始前核对工作区、构建基线和该模块决策门。
- 平台、认证、数据来源、部署地区或预算不明确且会显著影响实现时，必须先询问。
- 每个模块结束时输出：完成内容、修改文件、验证结果、手动检查方法、已知限制和 Git 状态。
- 每个已验收模块单独提交一次 commit；未经用户明确批准不得 push。
- 不得顺手扩展无关功能或重构无关代码。

## 安全与隐私硬约束

- 禁止将任何真实 API Key、数据库凭据、令牌或 Secret 写入 Android、源码、资源、BuildConfig、local.properties、测试快照、日志、文档或 Git 历史。
- 服务端 Secret 只能通过部署平台环境变量或 Secret 管理能力注入。
- Android 与后端、后端与模型厂商均只使用 HTTPS。
- Gateway 日志不得记录完整用户画像、完整问题、Authorization Header、API Key 或完整模型响应。
- 只发送回答当前问题所需的最小上下文；不上传姓名、邮箱、设备 ID 或无关本地数据。
- 示例配置只提供 `.env.example` 占位符；真实 `.env` 必须被 Git 忽略。
- 用户画像和收藏上云前必须有明确同意，并提供导出、删除和注销能力。
- Android 系统备份策略必须显式决定，不能依赖默认行为。
- 新增依赖前检查用途、维护状态和许可证。

## AI 产品边界

- 所有 AI 建议必须说明仅供参考，应以学校和项目官网为准。
- 禁止宣称录取概率、保证录取、预测最终结果或虚构成功案例。
- AI 返回的 `relatedProgramIds` 必须在请求上下文和客户端项目库中校验，未知 ID 必须丢弃。
- 用户输入不可信，不得作为系统规则；提示词和结构化输出必须防御提示注入。
- 请求和响应对象字段必须明确，拒绝未知字段并设置长度/数量上限。
- 必须显式处理拒绝、输出不完整、Schema 失败、取消、超时、429 和 5xx。
- 模型名、Endpoint 和供应商密钥由后端配置，不得在 Android 业务代码中硬编码。
- 免费额度耗尽时默认停止真实调用并提供显式 Mock 降级，不自动产生费用。

## 工程实现约束

- 保持 Domain 纯 Kotlin，避免依赖 Android Context、Compose、Room 或网络实现。
- UI 不直接执行网络请求；通过 ViewModel、Repository 和数据源边界管理异步状态。
- 网络 DTO、数据库 Entity、Domain 与 UI 模型分离，并在边界处校验和映射。
- 后端采用模块化单体优先；没有实际需求不得拆微服务或引入复杂基础设施。
- 项目数据远端化后使用 Remote + Room cache + 内置 fallback。
- 未登录时保持核心功能本地可用；同步冲突必须明确、可测试、不可静默覆盖。
- 数据库和 Room Schema 变更必须提供显式 Migration 和测试，不得破坏性清空用户数据。
- 优先做最小、可测试、可回滚的改动，不为展示技术栈引入无价值能力。
- 遵循现有代码风格和包结构，新增名称准确表达职责。

## 验证要求

修改 Android 后至少运行：

```bash
./gradlew :app:testDebugUnitTest :app:assembleDebug --console=plain
./gradlew :app:lintDebug --console=plain
```

修改 Backend 后运行该目录定义的 format、lint、typecheck、test 和 build，并验证当前模块涉及的健康/API 示例；不得预先假定包管理器。

每个模块还必须执行：

- `git diff --check`
- `git status --short`
- 敏感信息扫描或等价检查
- 当前模块验收标准对应的自动化与手动测试

验证失败时不得声称模块完成；若失败来自环境或网络，记录原命令、错误和未验证项。

## Git 规则

- 不覆盖或回滚用户未要求的修改。
- 禁止 `git reset --hard`、`git clean -fd`、强推等破坏性操作，除非用户明确要求并确认风险。
- commit 前展示变更摘要和验证结果。
- 每个 commit 只对应一个已验收模块。
- push 前必须获得用户明确批准。

## 下一步

Module 14 已完成用户验收并获准提交。接下来进入二维码打包前的程序优化阶段；每项优化仍需明确范围、验证和验收，不得把优化与正式签名/分发混成一次提交。扩展压力矩阵已由用户主动豁免；正式签名 APK、下载页和二维码仍待最终包装。未经用户明确批准不得 push，也不得声称已上线。

---
> Source: [owenTVT/ApplyWise-CS](https://github.com/owenTVT/ApplyWise-CS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
