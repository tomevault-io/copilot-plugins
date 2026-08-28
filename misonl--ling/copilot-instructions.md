## ling

> - 结论必须基于仓库事实、命令输出、测试结果，不凭印象猜测。

# AGENTS.md

## 沟通与执行
- 全程使用中文。
- 结论必须基于仓库事实、命令输出、测试结果，不凭印象猜测。
- 优先做最小必要修改；不要顺手扩大范围。
- 不要引入静默降级、假成功路径或仅为过测试而写的硬编码。

## 仓库结构
- `bin/`：CLI 主链路。`bin/ling-cli.js` 负责命令分发；`bin/adapters/` 处理 `gemini` / `antigravity` / `codex` 目标差异；`bin/core/` 负责构建与转换；`bin/utils/` 放原子写入、manifest、托管区块等通用能力。
- `.agents/`：仓库内唯一 Canonical 模板源。项目安装时：
  - `gemini` / `antigravity` -> 项目内 `.agent/`
  - `codex` -> 项目内 `.agents/`
- `.spec/`：Spec 进阶能力的模板、profiles、references 与 skills 源。
- `tests/`：Node 内置测试（`*.test.js`）。
- `docs/`：对外技术文档。
- `reference/`：只保留受管参考资料；当前应跟踪 `reference/official/`、`reference/docs-archive/`，不要重新引入重复镜像目录。
- `web/`：Next.js 文档站。仅使用 `npm`，不要重新引入 `bun` 工作流。

## 关键实现约束
- `gemini` 与 `antigravity` 共用 `.agent/`，但逻辑目标身份必须通过项目内 `.ling/install-state.json` 区分；涉及共享 `.agent/` 的逻辑时，不要退回“默认全算 gemini”的旧行为。
- `codex` 受管目录是 `.agents/`；遗留 `.codex/` 只作为兼容迁移输入，不再作为主输出。
- 全局同步默认目标是 `codex + gemini + antigravity`；`gemini` 与 `antigravity` 不能再互相隐式代管。
- 全局 `codex` Skills 根目录是 `~/.agents/skills/`；若 `~/.gemini/skills/` 与 universal 根目录存在内容完全相同的同名 Skill，需要执行去重清理；若同名但内容不同，必须保留 Gemini 专用版本。
- `ling spec enable` 若涉及 `codex` 或 `gemini`，必须与全局去重逻辑保持一致，不能在启用 Spec 后重新写回 Gemini CLI 重复 Skill。
- 文档、测试、CLI 帮助文本必须与实际命令行为一致；改命令行为时同步更新 `README.md`、`docs/TECH.md`、`docs/PLAN.md` 与相关测试。

## 构建、测试与验证
- 安装依赖：`npm install`
- 全量测试：`npm test`
- CI 主链路验证：`npm run ci:verify`
- 健康检查：`npm run health-check`
- 清理预检：`npm run clean:dry-run`
- Web 文档站检查：`cd web && npm run lint`
- 仅改文档时，至少执行 `git diff --check`；若改 CLI、安装/更新逻辑、状态判定、Spec 或发布链路，必须跑相关测试，默认优先 `npm test`。

## 代码风格
- 遵循 `.editorconfig`：默认 4 空格缩进，JSON/YAML 2 空格，LF，UTF-8。
- 根目录 Node 代码保持 CommonJS 风格（`require` / `module.exports`）。
- 文件命名使用 kebab-case。
- 不要加入 Emoji、花哨 Unicode 装饰符或无意义注释。

## 测试要求
- 新功能或行为修正必须补回归测试，优先覆盖：
  - CLI 参数与默认目标
  - 工作区状态判定
  - 全局同步与备份语义
  - Spec enable/init/doctor
  - 文档与实现的一致性
- 当修改 `reference/` 跟踪策略时，同步更新 `tests/standards-compliance.test.js`。

## 参考资料与忽略规则
- 不要提交 `__pycache__/`、`*.pyc`、`.DS_Store`、`.next/`、`*.log`、`*.tmp`、`.eslintcache`、`.turbo/`、`coverage/`、`dist/`、`web/tsconfig.tsbuildinfo`、临时 tarball、测试临时工作区（如 `.temp-ling-*`、`.tmp-ling-*`）或其他构建噪声。
- `reference/` 使用白名单忽略策略：保留需要受管的官方资料与归档文档，其余快照默认忽略。
- 不要重新引入 `reference/official-docs/` 这类重复目录；官方资料只保留一份。

## 版本与发布
- npm 包名：`@mison/ling`
- 版本源：`package.json`
- Git 标签格式：`ling-<semver>`
- 发布前默认检查：
  - `npm test`
  - `npm run ci:verify`
  - `npm run health-check`
  - `cd web && npm run lint`
  - `npm pack --dry-run`
- 发布时同步更新 `CHANGELOG.md`，内容必须覆盖自上一标签以来的实际提交。
- 正式发版顺序保持一致：先提交并推送 `main`，再推送 `ling-<semver>` 标签，随后发布 npm，最后创建 GitHub Release；不得出现 npm 版本领先于远端提交或标签的情况。

## 提交规范
- 使用 Conventional Commits，例如：
  - `feat(cli): ...`
  - `fix(cli): ...`
  - `chore(reference): ...`
  - `chore(release): ...`
- 一次提交只做一类事情；功能修正、参考资料清理、版本发布分开提交。

---
> Source: [MisonL/Ling](https://github.com/MisonL/Ling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
