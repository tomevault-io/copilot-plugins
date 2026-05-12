## agent-knowledge-framework

> 每个 agent 在开始任何任务前，**必须先通读本文件和角色 `AGENTS.md`**，再开始动手。

# Agent Notes

每个 agent 在开始任何任务前，**必须先通读本文件和角色 `AGENTS.md`**，再开始动手。

## 仓库结构

```
agent-knowledge-framework/
├── base/                       # 通用知识（所有角色共享）
│   ├── experience/             # 不属于特定角色的经验
│   ├── principles/             # 通用原则和规范
│   ├── skills/                 # 通用技能（每个技能目录主入口为 SKILL.md）
│   └── insights/               # 通用洞察（跨角色的规律性认知）
└── roles/                      # 各角色目录
    ├── cli-tool-dev/          # CLI/TUI 工具开发
    ├── maintainer/            # 知识仓库维护
    └── <role>/
        ├── AGENTS.md          # 角色描述 + 知识索引（始终加载）
        ├── principles/        # 角色专有原则
        ├── skills/            # 角色专有技能
        ├── insights/          # 角色专有洞察
        └── experience/        # 角色经验复盘
```

## 知识加载与沉淀

- **常驻 context**：`AGENTS.md`（本文件）+ `roles/<role>/AGENTS.md`
- **按需加载（分层）**：角色/目录 `AGENTS.md` 索引 → skill/principle/insight → experience（按 `source` 或 "Escalate to experience if" 升级）
- **症状/高风险直达**：出现特定报错或做迁移/发布/权限操作时，可直接用关键词检索 experience
- **Context 压缩后**：如果你不确定索引内容或协作规则细节，说明 context 已被压缩，立即重新 `Read` 本文件 + 角色 `AGENTS.md`

详细的加载策略（常驻 vs 按需判断标准、experience 唤醒、摘要写法）见 **`base/principles/knowledge-loading.md`**。

完成重要工作后（PR review 修复、跨层 bug 修复、首次跑通流程、踩坑），agent 应主动沉淀经验。知识分类（experience/skill/principle/insight）、存放位置判断、提炼流程和反模式，见 **`base/knowledge-sedimentation.md`**。

## 多 Agent 协作规则

本仓库假定**多个 coding agent 可能同时修改**，所有写操作必须遵守以下规则。

Agent 工作中涉及两类仓库，规则同样适用：

- **Agent 目录**（本仓库）：存放角色知识、经验、技能文档
- **工作目录**（实际代码仓库）

### 1. 所有写操作必须在 worktree 中进行，禁止直接 push main

**始终创建 worktree**，不要在主工作目录中 `git checkout` 或 `git checkout -b` 切分支。只读操作可以在 main 上进行。

**常见违规模式（禁止）**：
- `git checkout -b <branch>` 然后直接在主工作目录编辑——这不是 worktree，只是切了分支
- "先写完再搬到 worktree"——一旦开始编辑就很难搬，必须一开始就在 worktree 里

**正确流程**：
```bash
git fetch origin
git worktree add .claude/worktrees/<topic> -b <agent>/<topic> origin/main
cd .claude/worktrees/<topic>
# ... 编辑、commit ...
git push -u origin <agent>/<topic>
```

**关键**：永远基于 `origin/main` 创建 worktree，不要基于本地 `main`（可能过时）。详见 `base/principles/git-worktree.md`。

> **检查点**：执行任何 `git checkout -b` 或文件编辑前，先问自己"我现在在 worktree 里吗？"。如果 `pwd` 不包含 `.claude/worktrees/`，停下来，先创建 worktree。

### 2. 通过 PR 合并，agent 不要自行 merge

- push 分支后创建 PR，交给**人工 review 后合并**
- **agent 不得自行 merge PR**，即使有权限也不要执行
- push 时发现冲突：`git fetch origin && git rebase origin/main` 解决后再 push

### 3. push 后同步更新 PR title/description

向已有 PR 的分支 push 新 commit 后，**必须更新 PR title 和 description**，使其反映分支的完整变更。

### 4. 完成后清理 worktree

```bash
git worktree remove .claude/worktrees/<topic>
git branch -d <agent>/<topic>
```

## 开工 Preflight（每个任务都要做）

> 把"开工检查"从思考题变成流程题，避免 context compaction / 口述路径 / 记忆偏差造成的漂移。

### 1）进入角色（硬性）

每个任务的**第一条回复**必须明确：`当前角色` + `已读取 roles/<role>/AGENTS.md`。续接长 session 或不确定流程细节时，按 compaction 处理：重新 `Read` 本文件 + 角色 `AGENTS.md` 后再执行。

### 2）路径必须先验证，禁止猜

**先用 `ls/rg` 找到真实路径，再打开/引用**。找不到就明确说明"仓库内不存在该路径"并给出最接近候选项。常见笔误：`skill` vs `skills`、`AGENTS` vs `SKILL`。

### 3）触发型 skill：要做事先读（硬性）

- 接触凭证 / 权限 / token → 先读 `base/principles/credential-safety.md`
- 长任务（>30min / 需归档）→ 相关原则

### 4）确认"本地=最新"

**触发时机**：开工前、push/提 PR 前、遇到奇怪冲突/缺文件时。对 Agent 目录和工作目录都要检查。

```bash
git fetch origin
git rev-list --left-right --count main...origin/main   # 0 0 = 一致; 0 N = 落后需同步
git pull --rebase origin main                             # 工作区干净时执行
```

> 写操作仍以 `origin/main` 创建 worktree 为准，不要依赖本地 main。

---
> Source: [st1page/agent-knowledge-framework](https://github.com/st1page/agent-knowledge-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
