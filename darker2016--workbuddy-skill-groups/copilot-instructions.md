## workbuddy-skill-groups

> - **路径**: `/Users/darker/Documents/cursor_projects/WorkBuddySkillGroups`

# CLAUDE.md — WorkBuddy 专家团 Skill 包（开源版）

## 项目核心信息

- **路径**: `/Users/darker/Documents/cursor_projects/WorkBuddySkillGroups`
- **GitHub**: https://github.com/darker2016/workbuddy-skill-groups
- **平台**: WorkBuddy — 一个 AI 协作专家 agent 市场
- **内容**: 39 个专家团队协作 skill 开源包，每个文件夹对应一个多角色协作 agent 集合

## 协作规则（必须遵守）

1. **🚫 禁止主动推送到 GitHub**
   - 除非用户明确说"push"、"推送到 GitHub"、"sync to GitHub" 等明确要求，否则永远只在本地的 `main` 分支上 commit，不执行 `git push`。
   - 每次 commit 后必须再次确认用户是否要推送。

2. **📚 所有修改必须更新相关文档**
   - 修改某个 skill 目录的结构 / 命名 / 成员 → 同步更新该目录下的 `README.md`
   - 修改根级结构（如新增 / 重命名顶级 skill 目录、修改全局策略）：同步更新以下所有文件
     - `README.md`（双语根目录）
     - `README.zh.md`（中文根目录）
     - `CONTRIBUTING.md`（如涉及贡献流程）
     - `LICENSE`（如涉及作者列表变更）
     - `.github/ISSUE_TEMPLATE.md`（如涉及 issue 类型变更）
   - 任何跨 skill 协作（如 agent 调度流程新增、改为跨文件夹引用）必须更新相关 skill 的 lead SKILL.md `description:` 字段 + 文档中的"跨 skill 协作参考"段。
   - 规则本身发生变化时，反向更新本段。

3. **📖 理解项目时不假设旧上下文**
   - 每次会话先 `git log --oneline -5` + `git status` 看当前 repo 状态
   - 再看 `README.md` / `README.zh.md` 的 Appendix A（旧名 → 新名对照表）确认当前合法路径

## 文件结构速查

### 根目录文档

| 文件 | 作用 |
|------|------|
| `README.md` | 双语根目录（前 1/2 中文 §1-7 + 附录；后 1/2 英文 §1-7 + 附录） |
| `README.zh.md` | 中文全译本（章节号与 README.md 完全对齐） |
| `LICENSE` | MIT + 第三方 attribution（12 位作者/团队） |
| `CONTRIBUTING.md` | 贡献框架（事实修复 / 翻译 / SOP 模板化 / 框架迁移 / attribution） |
| `.github/ISSUE_TEMPLATE.md` | Issue 模板（含 DMCA / 移除请求） |
| `.gitignore` | 排除 `.DS_Store` / `.workbuddy/` / `__pycache__/` |
| `CLAUDE.md` | 本文件 — 核心记忆 + 协作规则（就是你正在看的） |

### Skill 目录内部规范（各 folder 一致）

```
<xxx>-skills/
  README.md              ← 必须包含：disclaimer · 描述 · 团队成员 · 工作流阶段 · 使用
  overview.md (可选)     ← 过时的入口文件；新创建的 skill 已废弃，改为 README.md
  *team-lead.md          ← 主理人的 SKILL.md，含 frontmatter 的 name / description / 触发词
  <member1>.md           ← 成员 SKILL.md（扁平结构）
  members/ 或 skills/    ← 或放在子目录（工程类 skill 通常用 members/、skills/）
  agents/                ← 少数 skill 把 agent 放在这里
  references/            ← 参考资料
```

**差异**：
- 扁平结构（大多数）：`team-lead.md` 与 `member1.md` 在同一层级
- 子目录结构（工程类）：`members/team-lead/SKILL.md` + `members/architect/SKILL.md`
- 命名差异：部分 skill 用"主理人/lead/agent/总监"等不同称呼，但结构等价

### Frontmatter 关键键

```yaml
---
name: <team-lead-id>
description: >-
  <一句话描述，含角色名 + 调度角色数量 + 阶段数 + 触发词>
agent_created: true   # 是否实际创建了 agent
---
```

### 主理人 Lead SKILL.md 模板

```markdown
# <团队名> · 主理人（<姓名> · <职级一句话>）

## 概述
<一句话职责。**不直接做专业产出**，而是调度 N 位团队成员 + 阶段化 SOP。>

## 团队成员
| 成员 | ID | 角色 | 名称 |
|------|----|------|------|
| <member1> | `<agent-id>` | <角色> | <名> |

## 工作流阶段
- Phase 1: <描述>
- Phase 2: <描述>
...

## 调度规则（CRITICAL）
- ❌ 禁止跳过"建立团队"的正式流程
- ❌ 禁止代写成员的专业产出
- ❌ 禁止让成员直连通信（所有信息流必须经主理人中转）
- ✅ 成员结论为准 — 只编排和汇编

## 使用
把 `<team-lead.md>` 作为 system prompt。
```

## Skill 团队 分类速查（按用户场景）

| 主类别 | Skill 目录 |
|--------|-----------|
| 投资分析 | `investment-masters-skills`、`a-share-skills`、`stock-partner-skills`、`trading-agent-skills`、`citongshuopro-skills`、`super-partner-skills` |
| 内容创作/营销 | `ai-content-creator-skills`、`video-gen-skills`、`promo-creator-skills`、`content-distribution-skills`、`marketing-campaign-skills`、`seo-content-skills`、`humanize-ppt-skills` |
| 开发/工程 | `aicoding-architecture-expert-skills`、`mvp-dev-skills`、`makers-skills`、`software-company-skills`、`software-workshop`、`engineering-assurance-skills`、`ai-data-copilot-skills` |
| 设计 | `design-engine-skills`、`humanize-ppt-skills`（复用） |
| 法律/合规 | `chatlaw-skills`、`enterprise-legal-skills`、`smb-team-skills` |
| 财税 | `tax-compliance-skills`、`invoice-verify-skills` |
| 数据分析 | `huashu-data-pro-skills`、`ai-data-copilot-skills`（复用） |
| 销售 | `sales-battle-skills` |
| 运营/HR | `hr-operations-skills` |
| 研究/教育 | `academic-journal-selector-skills`、`gpt-researcher-skills`、`ket-prep-skills`、`ncre-exam-skills` |
| 社媒 | `social-engagement-skills` |
| 市场/产品 | `product-strategy-skills` |
| 变现 | `content-monetization-skills` |
| 文档 | `open-spec-doc-skills` |

## ⚠️ 不要开源的 5 个 skill（已从平台列表移除）

- CloudOpsTeam（腾讯云技术支持）
- IndustrySreTeam（腾讯云行业 SRE）
- RumFullstackTeam（腾讯云 RUM 全链路）
- MigraqTeam（腾讯云上云迁移）
- skill-docs（重复目录，已被 open-spec-doc-skills 替代）

## ⚠️ 版本与归因

- 各 skill 内容可能随 WorkBuddy 平台更新而变化 → 不保证与原版 100% 一致
- 第三方版权作者见 `LICENSE` 顶部 attribution 列表
- 标签: 来自 WorkBuddy 官方团队 / 特邀作者（Easychen / 卡尔的AI沃茨 / 花叔 / 苍何 / 杜哥学量化 / 百望 / vincentlli / AI科学局 / Xu Qingchu 等）

## CLI 工作流：skillscreator（构想待实现）

后续计划实现 `skillscreator` CLI：
- 把"一个简单想法"→ 转化为完整的 skill 专家团 skeleton
- 会被放到根目录 `skillscreator/` 子目录
- 作用：
  1. 解析需求 → 识别"类别 / 阶段 / 主理人 + 成员结构"
  2. 基于现有 skill 的常见模式，生成 `team-lead.md` frontmatter + 成员列表 + 工作流阶段
  3. 同时生成该 skill 目录的 README.md（带 disclaimer + 文档关联）
  4. 提示是否同步更新根级 README 文件

## 发版工作流（每当有用户任务完成时）

1. `git status` — review 所有增删改
2. `git add -A`
3. `git commit -m "<类型>: <简述>"` （只 commit，不 push）
4. 询问用户: "已本地 commit，需要现在推送到 GitHub 吗？"
5. 仅当用户确认 push 时执行 `git push origin main`

---
> Source: [darker2016/workbuddy-skill-groups](https://github.com/darker2016/workbuddy-skill-groups) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
