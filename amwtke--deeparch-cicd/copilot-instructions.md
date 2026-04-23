## deeparch-cicd

> 一个用 Rust 实现的轻量级 CI/CD CLI 工具，通过 Docker 容器隔离执行流水线步骤。

# Pipelight - 轻量级 CLI CI/CD 工具

## 项目概述
一个用 Rust 实现的轻量级 CI/CD CLI 工具，通过 Docker 容器隔离执行流水线步骤。
目标：替代 Jenkins/GitLab CI 等重量级方案，提供本地优先、快速、可离线使用的 CI/CD 体验。

## 架构分层
```
CLI 层 (clap)        → 子命令: run / validate / list / logs
    ↓
Pipeline 模型层       → YAML → DAG 解析, 变量插值, 条件表达式
    ↓
调度器层             → DAG 拓扑排序, 并行 step 调度 (tokio)
    ↓
执行器层             → Docker API 交互 (bollard)
    ↓
输出层               → 实时日志流, 彩色终端输出
```

## 编码约定
- 错误处理统一用 `anyhow::Result` (应用层) + `thiserror` (库层自定义错误)
- 所有异步操作基于 `tokio`
- 日志统一用 `tracing` (不要用 println! 或 log crate)
- Docker 交互统一通过 `bollard` crate，不要 shell 调用 docker CLI
- YAML 解析用 `serde` + `serde_yaml`
- DAG 用 `petgraph` 构建和调度
- 代码注释用英文，用户面向的输出用英文

## 关键设计决策
- Pipeline 配置文件默认名: `pipeline.yml`
- Step 间通过共享 Docker volume 传递 artifact
- 环境变量支持 `${VAR}` 插值语法
- 并行执行无依赖关系的 step
- PMD 使用独立 PMD CLI（非 Maven 插件），缓存在 `~/.pipelight/cache/`
- `pipelight clean` 清除项目产物（pipeline.yml + pipelight-misc/），不影响全局缓存

## Git HTTPS 认证 (git_credentials)
pipeline.yml 支持 `git_credentials` 字段，用于内网 HTTPS 环境下 Docker 容器内的 git 操作：
```yaml
name: my-ci
git_credentials:
  username: your_username
  password: your_token_or_password
steps:
  - name: git-pull
    ...
```
- 凭据通过环境变量注入容器，用 `git credential helper` 传递给 git
- **安全提示**：不要将包含明文密码的 pipeline.yml 提交到版本控制，建议加入 `.gitignore`
- 如果不配置 `git_credentials`，git-pull 会尝试 SSH key（挂载 `~/.ssh`）

## Skill 回调 (Skill Callback)
Pipelight 与调用它的 LLM 之间的协作协议。Pipelight 本身不执行智能操作（搜索文档、生成配置、修复代码），
而是通过 JSON 输出中的 **回调命令 (CallbackCommand)** 告知 LLM 应当执行的操作，LLM 按照 skill 中的指令完成后重试。

### 回调命令 (CallbackCommand)
定义在 `src/ci/parser/mod.rs` 的 `CallbackCommand` 枚举，YAML/JSON 字段名为 `on_failure.callback_command`：
- `auto_fix` — 通知 LLM 自动修复源代码，然后 retry 该 step
- `auto_gen_pmd_ruleset` — 通知 LLM 搜索项目中的 PMD ruleset 或 coding guideline 文档，生成 `<目标项目根目录>/pipelight-misc/pmd-ruleset.xml`，然后 retry 该 step；若都找不到则 skip PMD。**注意：`pipelight-misc/` 必须位于执行 `pipelight` 命令的目标项目根目录下**（即 `pipeline.yml` 所在目录），而非 pipelight 工具自身的目录
- `abort` — 不可重试，直接标记为 failed
- `notify` — 不可重试，通知用户

### auto_gen_pmd_ruleset 搜索规则
搜索分两轮，优先复用项目已有配置，找不到才从 guideline 文档生成：

**第一轮：搜索项目中已有的 PMD 配置文件**（直接复制到 `pipelight-misc/pmd-ruleset.xml` 使用）
- `**/pmd-ruleset.xml`、`**/pmd.xml`、`**/config/pmd/*.xml`
- `pom.xml` 中 `maven-pmd-plugin` 引用的 ruleset 路径
- `build.gradle` 中 `pmd { ruleSetFiles = ... }` 引用的文件
- **禁止**使用 `target/` 或 `build/` 目录下的文件（构建产物，不可靠）

**第二轮：搜索编码规范文档**（读取内容后生成 PMD ruleset）
- 不同项目使用不同的编码规范，不要假设特定规范（如阿里巴巴、Google 等），根据实际文档内容生成
- 搜索路径优先级：`doc/` → `docs/` → 项目根目录下的 `*规范*`、`*guideline*`、`*coding*` 文件
- 支持 PDF/MD 格式，任何语言

**都找不到**：skip PMD（`--skip pmd`）

### 回调协议流程
```
Pipelight step 失败 → JSON 输出包含 callback_command + stderr
    ↓
pipelight-run skill 根据 callback_command 指导 LLM 执行相应操作
    ↓
LLM 完成操作（修复代码 / 生成配置 / 跳过 step）
    ↓
pipelight retry --step <name> 重试
```

### 术语约定
- 当说 **"skill 回调"** 时，指的就是这种 pipelight ↔ LLM 之间的回调设计模式
- 当说 **"回调命令"** 时，指的是 `CallbackCommand` 枚举中的具体值（如 `auto_fix`、`auto_gen_pmd_ruleset`）

## 目录结构
```
src/
  main.rs              → 入口
  cli/                 → clap 命令定义
  run_state/           → 运行状态持久化
  ci/                  → CI 流水线核心 (6 个阶段)
    detector/          → 检测器: 项目类型检测 (策略模式)
      base/mod.rs      →   ProjectDetector trait, ProjectInfo, ProjectType, 检测编排
      maven.rs / …     →   各语言检测策略实现
    pipeline_builder/  → 流水线步骤生成 (StepDef trait + 策略模式)
      base/            →   5 个公共 step: git_pull/build/test/lint/fmt (*_step.rs)
      maven/ / …       →   各语言策略 + 特有 step (*_step.rs, 实现 StepDef trait)
    parser/            → 解析器: Pipeline YAML 解析 & 校验
    scheduler/         → 调度器: DAG 构建 & 拓扑排序
    executor/          → 执行器: Docker 容器执行
    output/            → 输出器: 终端日志格式化 (tty/plain/json)
```

## Skill 编辑规则
- 修改 skill 时，必须先改 `global-skills/<skill-name>/SKILL.md`（repo 源文件），再通过 `cp -r global-skills/<skill-name>/ ~/.claude/skills/<skill-name>/` 同步到本地
- **禁止**直接编辑 `~/.claude/skills/` 下的文件，因为 `/pipelight-sync` 会用 `global-skills/` 覆盖本地
- 改完后需要 commit + push 到 repo，确保跨机器同步

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amwtke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
