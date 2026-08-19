## obsidian-task-center

> 一个 Obsidian 插件：周/月视图 + 父子任务渲染 + 自然语言 Quick Add + 移动端手势。

# obsidian-task-center

一个 Obsidian 插件：周/月视图 + 父子任务渲染 + 自然语言 Quick Add + 移动端手势。

项目的三份核心文档：

- `USER_STORIES.md`：唯一产品需求来源
- `UX.md`：界面与交互规范
- `ARCHITECTURE.md`：技术架构与实现边界

## 元规则

1. 永远中文回复。
2. 先文档，后代码。没有完成需求与设计收敛前，不进入实现。
3. 不写一次性脚本（`mjs` / `sh` / 临时 `md`）。一次性的活直接执行。
4. 所有实现都要能回溯到用户故事，不做“感觉有用”的游离开发。
5. commit message 使用中文 conventional：`<type>(<scope>): <description>`，其中 `type ∈ feat/fix/chore/upgrade/docs`。

## 多 Agent 并行（重要）

这个仓库经常有多个 agent **同时**在 `main` 上工作。分支历史会在你工作期间被别的 agent 推进（新提交、甚至 rebase/amend），你看到的 HEAD 随时可能已经不是你上次看到的那个。据此：

1. **只在 `main` 上并行**：不开分支、不切分支。每个 agent 直接在 `main` 上提交自己的小步改动；并行靠"各自频繁提交"，不靠分支隔离。
2. **禁用命令清单（最重要）**：下面这些命令在本仓库**一律禁止**，因为它们会动到共享历史、或一次性冲掉整个工作树里别的 agent **未提交**的在途改动（未提交改动不进 reflog，冲掉就**永久丢失、无法找回**）。即使你只想撤销自己那一处改动，也不准用它们——它们的作用域往往是整棵树或整个文件，会顺手抹掉别人的活：
   - **改写历史类**：`git reset`（尤其 `--hard`、`git reset HEAD~`）、`git rebase`、`git commit --amend`、`git push --force` / `--force-with-lease`。
   - **还原 / 清理工作树类**：`git checkout -- <任何路径>`（含 `git checkout .`）、`git restore`（含 `--worktree` / `--staged` / `git restore .`）、`git stash`（`push` 会把工作树改动抽走）、`git clean`（`-f` / `-fd`）、`git switch` / `git checkout <branch>`（切分支）。
3. **撤销自己的改动只能手动改文件**：要回退你自己写的东西，用编辑器把那几处改回去，再正常提交一个新 commit。**永远不要**用上面任何 git 命令去"还原 / 抹掉"——哪怕你确信只动了自己的文件。
4. **不碰不是你写的东西**：工作树里出现你没动过的未提交改动（`git status` 里的 `M`/`??`），那是别的 agent 的在途工作——不要 `git add -A` 一把全提交、不要用第 2 条里的命令覆盖。提交时只 `git add` 你自己明确改过的文件。别人改坏了 build 也别替它"还原"，那是它自己会提交修复的在途状态。
5. **提交前先看清范围**：用 `git status` + `git diff` 确认你只提交了自己的改动；`git add -A` 在多 agent 环境里很危险，会卷入别人的在途文件。
6. 如果确实需要动历史或清理工作树，先停下来问人，不要自行决定。

> 教训 1：曾经为了撤销自己的一个提交用了 `git reset --hard`，连带把另一个 agent 刚加在其上的提交一起摘掉。正确做法是手动改回自己的文件 + 新提交，永远不动 `main` 历史。
>
> 教训 2：曾经为了清掉另一个 agent 越界的烂改动，跑了 `git checkout -- .` / `git restore .`，把同在 `main` 工作树上另一个 agent **还没提交**的 `view.ts` 改动整片冲掉——未提交改动不进 reflog，无法恢复，只能让对方重写。要丢弃某个文件的改动只能手动编辑那个文件，绝不用 checkout/restore/stash/clean 还原工作树。

## 项目命令

这些命令描述项目本身的检查入口。完成开发后的默认验证不要直接在本地开发机跑，而是通过 Crabbox/provider 在远端测试环境执行。

- 开发：`pnpm dev`
- 构建：`pnpm build`
- 单测：`pnpm test`
- e2e：`pnpm e2e`

## 远端测试

完成开发后，只用 Crabbox/provider 在远端测试环境跑正式 gate；本地开发机只负责编辑代码和发起远端命令。

本仓库的正式远端 gate 是 Bring Your Own Infrastructure 形态：内网 k3s/containerd 宿主作为 devbox 宿主，通过 Crabbox `external` lifecycle 创建/释放独立 Pod lease。每个 gate 拿到唯一 lease id → 新建一个带 SSH 的唯一 Pod，Crabbox 通过 SSH ProxyCommand 同步 checkout、执行命令、回传日志和结果，结束后释放对应 Pod；多个 agent 并行时各自一个 Pod，互不踩踏。不要把它当成 Blacksmith Testbox，也不要退回到单工作目录 Static SSH gate。

box 生命周期（`external.command` 指向本机 adapter、`idempotentLeaseId`、`workRoot`）声明在 committed 的 `.crabbox.yaml` 里，所以 `crabbox run` 不再带任何 per-run lifecycle flag。真正内网的值——宿主主机名、k3s namespace、镜像——不写入 git：由 gitignored 的 `.crabbox/external-lease-provider.mjs` 从 `.crabbox/remote-test.env` 的环境变量读取。需要远端运行时，只从 gitignore 内的本地配置读取 gate 命令并执行。

推荐把私有命令放在 `.crabbox/remote-test.env`，该文件只保存本机可用的最小环境变量：

```bash
CRABBOX_TEST_PREPARE='...' # 没有前置连接需求时可为空
CRABBOX_TEST_GATE='crabbox run --shell '\''...gate 命令...'\'''  # provider/target/lifecycle 都来自 .crabbox.yaml
```

在远端跑正式 gate 只有一条路径：

```bash
pnpm run test:remote
```

`CRABBOX_TEST_GATE` 走 Crabbox `external` BYOI 路径，形态为 `crabbox run --shell '...'`——provider / target / external lifecycle 都来自 `.crabbox.yaml`，gate 里不重复这些 flag。`.crabbox.yaml` 声明公开的 provider / target 和指向本机 adapter 的 external lifecycle（中性路径、workRoot、idempotency），不保存内网地址（主机名 / namespace / 镜像留在 gitignored adapter + env）。

正式 gate 至少包含：

- `pnpm install --frozen-lockfile`
- `pnpm run typecheck`
- `pnpm run lint`
- `pnpm run test:unit`
- `pnpm run build`
- `CI=true GITHUB_ACTIONS=true OBSIDIAN_VERSIONS=latest/latest WDIO_MAX_INSTANCES=1 pnpm run test:e2e:ci`

`pnpm e2e` 是全量历史 e2e，不等同于正式 gate；其中包含不稳定或暂未纳入 CI 白名单的 specs。判断发布/PR 是否可过时，以 `docs/ci-test-matrix.md` 和上面的正式 gate 为准。

## 工作流

这个项目的默认顺序是：

1. 先写或修改 `USER_STORIES.md`
2. 再从用户故事推出 `UX.md`
3. 再从用户故事和 UX 收敛 `ARCHITECTURE.md`
4. 最后才写代码、补测试、验证行为

禁止跳过前面的文档阶段，直接讨论实现细节或直接改代码，除非任务本身就是纯修复已有实现中的明确 bug。

## 1. 用户故事优先

任何功能开发先落到 `USER_STORIES.md`，并以用户视角描述：

- 谁在什么场景下要完成什么事
- 为什么这件事有价值
- 成功结果是什么
- 明确边界和非目标

只有当用户故事足够清晰、可验收、无明显冲突时，才进入 UI 设计。

## 2. 由用户故事推出 UI

`UX.md` 不是独立发挥，而是用户故事的界面表达。写 UI 方案时要回答：

- 每个用户故事在界面上的入口是什么
- 用户的主路径和关键分支是什么
- 哪些状态需要可视化：空状态、加载态、错误态、选中态、拖拽态、手势反馈等
- 桌面端与移动端的交互是否一致，若不一致，差异是什么

如果一个 UI 决策无法映射回具体用户故事，就说明它还不该进入规范。

## 3. 由故事和 UI 推出架构

`ARCHITECTURE.md` 只解决“如何支撑需求和交互”，不抢在前面主导产品。

架构设计至少要说明：

- 模块边界与职责
- 数据流与状态来源
- 与 Obsidian API 的交互方式
- 哪些能力要做成可测试的纯逻辑，哪些是视图层适配
- 哪些约束来自移动端、性能、兼容性或插件生命周期

如果某个技术方案不能明显支撑用户故事或 UX，就不应进入架构正文。

## 4. 最后才是代码

只有在用户故事、UX、架构三者已经对齐后，才进入实现。

实现阶段要求：

- 代码必须对应已有用户故事
- 新功能必须补测试，优先覆盖关键用户路径
- UI 相关改动除了行为验证，还要验证视觉结果
- 改完代码先 `pnpm build`，再跑相关测试；涉及真实交互的改动要跑 `pnpm e2e`

## 任务分类

### 新功能

先改文档，再改代码：

1. 更新 `USER_STORIES.md`
2. 更新 `UX.md`
3. 更新 `ARCHITECTURE.md`
4. 实现代码与测试

测试里应能看出对应的用户故事编号或场景描述。

### Bug 修复

如果是“现有行为偏离既有用户故事 / 既有 UI / 既有架构”的 bug，可以直接进入修复，但仍要先确认依据来自哪份文档。

要求：

1. 先写一个能复现问题的失败测试
2. 再修复实现让测试转绿
3. 修复过程中如果发现其实是需求不清，不要硬修，先回到文档

### 重构

重构不改变行为，只整理实现。

要求：

1. 不得偷偷改变用户可见行为
2. 依赖已有测试兜底
3. 如果触及的区域没有测试，先补覆盖，再重构

## 测试要求

- 单测用于保护纯逻辑、边界条件、数据转换
- e2e 用于保护真实用户路径、Obsidian 集成、交互回归
- UI 改动不能只看测试通过，还要确认界面结果符合 `UX.md`

测试不是装饰；测试的目标是证明用户故事真的被满足。

## 代理执行准则

在这个仓库里工作时，默认按下面的顺序思考：

1. 先确认这次任务对应哪个用户故事
2. 再确认 UI 是否已经定义清楚
3. 再确认架构是否已经允许这样做
4. 最后才决定如何实现

如果发现三份文档互相冲突，优先停下来修正文档，不要直接用代码“拍板”。

## Harness Lint

<!--HARNESS LINT START-->
Development in this project should follow LDD (Lint Driven Development). When user feedback or code review points out how a class of code should or should not be written, do not only fix the current instance. Create or update a `harness-lint` rule that can catch the issue, run lint so it reports the problem, and then modify the code until lint passes.

When creating a local rule, use this workflow:

1. Run `harness-lint rule list` to inspect existing lint rules and decide whether an existing rule should be updated.
2. Before creating a new rule, decide whether the feedback can be expressed as a reliable GritQL pattern. If it cannot, do not create a harness-lint rule; keep it in agent instructions, review notes, or project documentation instead.
3. If a new rule is needed, run `harness-lint rule create "<feedback>" --language <language> --grit <gritql>` to create the local rule file.
4. Edit the generated rule file and fill in the rule description and Bad / Good examples.
5. Run `harness-lint doctor` to confirm that the configuration, rules, and Grit environment are healthy.
6. Run `harness-lint check --all --rule <rule-id>` and confirm the new rule reports the expected file(s). Do not pass paths to `check` to simulate rule scope; if the rule should only apply to certain files, encode that in GritQL with `$filename`.
7. Run `harness-lint check --changed` to execute lint and confirm that the rule can be loaded and works as expected.

Follow these best practices when writing local rules:

- Each rule should express exactly one stable, repeatedly checkable team constraint.
- Rule `id` values and filenames should be readable and stable. Chinese and other languages are allowed, but do not use path symbols or decorative symbols. Replace spaces with `-`. English should preferably use lowercase kebab-case, such as `local.no-print-debug`. Chinese can use short phrases, such as `local.禁止使用UI` or `local.禁止-使用-UI`.
- Keep the `id` and filename aligned whenever possible. For example, `id: local.no-print-debug` should correspond to `no-print-debug.md`.
- Each rule file must contain exactly one executable `grit` fenced code block. Start the GritQL with the smallest and most certain bad-code shape. Use metavariables such as `$value`, `$name`, and `$body` for parts that vary. If the GritQL is not reliable enough, do not create a harness-lint rule.
- If a rule should only apply to certain files, express that directly in GritQL with `$filename` conditions, such as `$filename <: r".*src/.*\.ts"` and `!$filename <: r".*\.test\.ts"`.
- Bad examples should show the smallest violating code. Good examples should show the replacement pattern recommended by this project. Example languages must match `language`.
- Use `level: error` only when the GritQL, description, and Bad / Good examples are all clear enough. Otherwise, keep `level: warn`.

If you need to write a rule or are not familiar with harness-lint, load the harness-lint skill first. If that skill is not available, install it with `npx skills add CorrectRoadH/harness-lint`.

If lint fails, first run `harness-lint rule explain <rule-id>` to read the specific rule. When the rule is correct, fix the code. When the rule is a false positive, narrow the GritQL, add clarification, or adjust the Bad / Good examples, but do not delete or weaken the rule just to make lint pass.
<!--HARNESS LINT END-->

---
> Source: [CorrectRoadH/obsidian-task-center](https://github.com/CorrectRoadH/obsidian-task-center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
