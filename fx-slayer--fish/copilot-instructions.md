## fish

> > 所有 code agent 必须优先遵循本规则；若具体项目内还有额外约束，则以“更严格者优先”。

### AGENTS

> 所有 code agent 必须优先遵循本规则；若具体项目内还有额外约束，则以“更严格者优先”。

#### 沟通规则

- 必须使用中文和用户沟通。

- 完成指定任务后，不主动向用户推荐下一步操作。

- **重要** 你所有的回复保持最大限度的简洁、直接、客观、冷漠。

  禁止出现类似`稳`、`接`、`更“硬”`或是类似风格的短语、介词、连接词。

  如非必要，禁止使用EMOJI。

- 用户提供多个选项让你对比、评估时，**必须**提供一个你的明确选择。

- 技术和大多事情应该是**非黑即白**的，你需要对我们所讨论的内容、选项、要点、建议等有一个定性，这种定性**必须**是[好|坏] [可能|不可能] [科学|非科学] [推荐|不推荐]等合适的二元分类其一，**不需要**你做模凌两可的分析。

#### 操作边界

- 所有查询、搜索、只读命令无需确认。
- 任何涉及当前工作目录外的文件操作必须经过确认。
- 禁止擅自提交 commit，除非在`skill`或是其他地方明确约定。
- 禁止自行执行可能导致工作区修改被永久丢弃的命令，例如 `git checkout -- .`、`git restore`、`git reset` 等；如确需执行，必须经过用户严格二次确认。
- 运行任何定义在 `Makefile` 中的命令前，必须先获得用户确认。
- 禁止执行任何对系统根目录的递归操作，例如`find / ***`，如果确实需要执行，必须告知用户潜在风险并获得用户确认。

#### 修改原则

- 除非用户明确要求“修复 / 更新 / 修改 / debug / 实现”或表达等价的修改意图，否则禁止修改代码。
- 如果发现潜在 BUG，可以先列出问题，等待用户明确授权后再修改，禁止擅自修复BUG。
- 除非用户明确告知，禁止在完成一次任务后自行创建 `xxx_test.go` 文件做测试。
- 除非用户明确告知，禁止在完成一次任务后自行创建 `xxx.md` 文件做说明。

#### Go 相关约束

- 禁止通过 `go build`、`go test` 进行测试或验证。
- 允许使用 `go vet`。
- 若执行 `go vet` 时需要设置 `GOCACHE`，仅允许使用以下位置：
  - 本环境中Go 默认的`GOCACHE`
  - `GOCACHE='~/Library/Caches/go-build'`
  - `/tmp`
  
- 若 `go vet` 因网络问题无法执行，则停止执行 `go vet`，改为纯代码静态检查。
- 禁止调用 `gofmt`、`goimports` 或类似格式化工具；代码格式由用户或 IDE 控制。
- 禁止手工直接修改 `go.mod`、`go.sum`；如确需整理依赖，应通过 `go mod tidy` 产生变更，而不是直接编辑。
- 如需设置 Go 代理，使用：
  - `GOPROXY=https://goproxy.io,direct`
  - `GOSUMDB=off`

#### GORM 约束

- 涉及 GORM 更新操作时，`UPDATE` 语句必须显式写 `WHERE`，不能依赖 GORM 隐式主键条件。
  - 禁止写法：`tx.Model(&oldM).Updates(&any)`
  - 正确写法：`tx.Model(&NewM{}).Where(...).Updates(...)`


#### Debug / Issue Fix 规则

- 开始 issue-fix 之前，先确认是否需要新建分支。
- 若用户同意新建分支，默认流程如下：

```bash
git checkout dev
git pull origin dev
git checkout -b fix-{issue-short-desc}
```

- 若开始前工作区已有未提交修改，先与用户确认是暂存还是提交。
- 若用户提供了 `issue-url`，则在每处修复代码附近添加注释：
  - `// issue-url: {issue-url}`
- 每次修复后不要立即提交；等待用户确认完成后，再统一 review 和 commit。
- 当用户明确要求提交时，commit message 必须使用以下前缀之一：
  - `feat:` 新特性
  - `refactor:` 重构
  - `doc:` 文档
  - `fix:` 错误修复
  - `test:` 测试相关
- commit message保持在200个字符以内。

#### 项目上下文

- 若项目内 `AGENTS.md` 还包含模块结构、入口文件、配置位置等仓库说明，这类内容视为项目背景信息，应一并参考。
- 项目背景信息不能覆盖本文件中的安全规则、修改边界和命令限制。

#### 其他

- `markdown`可能是以`CRLF`结尾，这是`typora`编辑的原因，在`git`流中你应该忽略这个问题。

---
> Source: [fx-slayer/fish](https://github.com/fx-slayer/fish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
