## cn-kis-v2-0

> 任务生命周期纪律：AI 每次任务前确认上下文，每次提交前确认测试与风险


# 任务生命周期纪律（强制）

## 任务开始时（必须确认的 5 件事）

每次用户描述新的开发需求时，AI 在动代码之前必须确认：

| 问题 | 确认方式 |
|------|----------|
| 关联的是哪个 Issue？ | 用户告知 Issue 编号，或 AI 建议先创建 Issue |
| 归属哪个工作台？ | 从分支名提取，或询问用户 |
| 任务目标是什么？ | 用户描述，AI 复述确认 |
| 当前环境是哪个？ | 本地开发 / 阿里云测试 / 火山云生产 |
| 是否触及红线域？ | 认证/权限域、知识资产域、部署配置 |

## 新 Cursor 窗口恢复上下文

当用户切换到新的 Cursor 窗口或对话时，AI 应主动执行：

```bash
git branch --show-current   # 当前任务分支
git log --oneline -5        # 最近提交记录
git status                  # 工作区状态
git diff --stat origin/main # 相对 main 的改动摘要
```

并根据以上信息告知用户：
- 当前任务是什么（从分支名中解析）
- 已完成哪些改动
- 下一步应该做什么

## 提交代码前（必须过的 6 项检查）

在 AI 生成 `git commit` 命令或建议用户提交之前，必须逐项确认：

```
□ 改动只涉及当前分支所属工作台（无跨台污染）
□ 未包含 .env、密钥、*.pem 等敏感文件
□ 若涉及 backend，已完成自测（见下方定义）
□ 若涉及 models.py，迁移文件已生成并包含在提交中
□ 若涉及认证域，refresh_token 防覆盖逻辑未被破坏
□ 提交信息符合 Conventional Commits 格式
```

## 业务用户"自测"定义

> 对于不具备专业测试背景的业务同事，"自测"的最低标准是：
> **能够截图/描述目标功能可见且正常运行，不存在明显报错。**

| 改动类型 | 最低自测标准 |
|---------|------------|
| 前端 UI 改动 | 启动本地前端，截图证明页面可打开、无报错红屏 |
| 后端 API 新增/修改 | 能描述接口返回的是什么（截图 / curl 输出），或告诉 AI「帮我用 curl 验证这个接口」 |
| 数据库迁移 | `python manage.py migrate` 运行成功，无 ERROR |
| 文案/样式微调 | 页面截图显示改动正确 |
| 配置文件改动 | 运行 `python manage.py check` 无错误 |

**AI 的职责**：当用户不确定如何自测时，AI 必须主动提供具体的自测命令或步骤，而不是直接跳过自测阶段提交代码。

**自测无法完成时的处理**：
- 用户本地环境未配置齐全 → AI 帮助完成环境配置
- 涉及外部服务（飞书 API 等）→ 降级为验证代码逻辑正确（静态分析），并在 PR 描述中注明「需要集成测试验证」
- 时间紧迫 → 创建 Draft PR，注明「待完成自测」，不得直接创建正式 PR

## 提交信息格式（Conventional Commits）

```
<type>(<workstation>): <简要描述>

type: feat | fix | chore | docs | refactor | test | hotfix | wave
workstation: 与分支所属工作台一致

示例：
feat(quality): 新增样品规则编辑器
fix(secretary): 修复登录循环问题
chore(common): 更新 PR 模板分支命名校验
wave(identity): RBAC token 加固（Wave 1）
```

## PR 准备前（必须生成的 PR 内容）

在建议用户创建 PR 时，AI 必须输出以下内容的草稿：

1. **PR 标题**：`<type>(<workstation>): <简要描述>` 格式
2. **业务目标**：一句话说清楚为什么做这个
3. **影响范围**：涉及哪些工作台、接口、是否有迁移
4. **测试步骤**：至少 3 条可执行的测试步骤
5. **风险点**：最高 2 条主要风险
6. **回滚方案**：`git revert <commit>` 或具体命令

## PR 提交后的自动化处理路径

PR 创建并推送后，系统会自动运行 CI、CodeRabbit、PR Quality Gate。
**业务人员只需关注飞书通知，根据通知内容告诉 Cursor AI 执行对应技能。**

AI 收到以下情况时，必须主动引导并执行对应技能，不等用户询问：

| 飞书/GitHub 通知 | AI 必须立即执行 |
|---|---|
| 飞书：「CI 失败，失败步骤是 XXX」 | 主动说「我来帮你修复」，执行 `fix-ci-failure` 技能 |
| 飞书：「AI Review 需要修改」（CodeRabbit） | 主动说「我来逐条处理」，执行 `handle-review` 技能 |
| GitHub Review 评论有 changes_requested | 执行 `handle-review` 技能 |
| 飞书：「PR 已合并」 | 提醒用户在飞书找「发布到生产」按钮并点击 |
| 飞书：「生产部署失败」 | 执行 `fix-ci-failure` 技能，用 `gh run view` 读取部署日志排查 |

### 飞书 2 人批准后流程（全自动，无需操作 GitHub）

```
飞书 2 人点击「批准合并」
  → GitHub feishu-approval/required = success
  → PR auto-merge 已启用
  → 所有 required checks 全部通过后 GitHub 自动合并
  → 飞书出现「发布到生产」按钮
  → 负责人点击按钮 → 生产自动部署
```

**业务人员不需要打开 GitHub，不需要运行任何 git 命令来完成合并。**

### 技能索引（AI 必须知道）

| 技能 | 触发词 |
|---|---|
| `start-task` | 开始新任务、创建分支、认领 Issue |
| `sync-main` | 同步最新代码、合并 main 到当前分支 |
| `prepare-pr` | 帮我提交 PR、准备 PR |
| `fix-ci-failure` | CI 失败了、检查没通过、帮我修 CI |
| `handle-review` | 帮我处理评审意见、CodeRabbit 有意见 |
| `resolve-conflict` | 有冲突、合并冲突 |
| `check-system-pulse` | 系统状态、今天进展、脉搏报告 |

## AI 的禁止行为

- ❌ 用户说「先随便提交」就直接 `git add . && git commit`
- ❌ 不检查迁移文件就提交 models.py 变更
- ❌ 提交时使用无意义的 message（如"fix", "update", "修改"）
- ❌ 不问 Issue 编号就帮用户建分支
- ❌ 新窗口打开后不恢复上下文就直接修改代码
- ❌ CI 失败时只把 GitHub Actions 链接甩给用户，不主动帮忙修复
- ❌ CodeRabbit 有意见时只说「去看看」，不主动执行 handle-review 技能

## 上下文持久化（不依赖聊天历史）

上下文的真相源是仓库，不是 Cursor 对话。  
每次窗口切换后，上下文恢复依赖：

```
git branch --show-current   → 任务名称与工作台
git log --oneline -3        → 进展与已完成的工作
git status                  → 当前未提交的改动
docs/TEAM_WORKFLOW.md       → 协作规范
.cursor/rules/              → AI 行为约束
```

**不要**依赖对话历史来恢复状态。对话历史不可靠，仓库状态才是真相。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/china-norm-company)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/china-norm-company)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
