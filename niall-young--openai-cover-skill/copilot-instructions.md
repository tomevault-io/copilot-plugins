## openai-cover-skill

> - 与用户沟通、说明改动和编写提交信息时统一使用中文。

# 项目协作约定

## 沟通语言

- 与用户沟通、说明改动和编写提交信息时统一使用中文。

## 修改与提交

- 本项目是 Skill 项目，主要代码载体为 Markdown，同时也可能包含 YAML、图片等配套资源；所有项目文件改动都必须纳入 Git 管理。
- 每完成一组独立且可验证的改动，就立即创建一次 Git 提交，不要把多个无关改动堆在同一个提交中。
- 提交前检查 `git status` 和实际差异，只暂存本次任务涉及的文件，不得擅自提交、覆盖或丢弃用户已有的未提交改动。
- 提交信息使用中文，并以 Conventional Commits 类型开头，格式为 `<类型>: <中文说明>`。
- 常用类型：
  - `feat`: 新增 Skill 能力、文档或资源。
  - `fix`: 修复错误、遗漏或不符合预期的行为。
  - `docs`: 仅调整说明性文档。
  - `refactor`: 重构现有内容且不改变外部行为。
  - `test`: 新增或调整测试与验证内容。
  - `chore`: 工具、配置或其他维护性改动。
- 示例：`feat: 新增杂志封面构图规则`、`fix: 修正图片生成提示词中的尺寸约束`。

## 同步 Codex 已安装 Skill

- 每次修改 `openai-editorial-cover/` 下的 Skill 文件并完成验证后，必须在结束任务前将整个 `openai-editorial-cover/` 目录同步到 Codex 当前安装目录；默认目标为 `~/.codex/skills/openai-editorial-cover/`，如当前环境设置了 `CODEX_HOME`，则使用对应的 `$CODEX_HOME/skills/openai-editorial-cover/`。
- 同步范围只包含正式 Skill 目录，不得把仓库根目录的 `output/`、Git 元数据或其他开发文件复制到安装目录。
- 同步完成后必须比较源码目录与安装目录，确认文件内容一致；如同步失败或目标目录不可用，必须在最终回复中明确说明，不得把未同步状态描述为完成。
- 最终回复中简要说明本次是否已更新 Codex 已安装 Skill。

## 本仓库的测试图片

- 本节只约束开发者或代理在当前源码仓库中测试 Skill 时产生的临时图片，不是 Skill 功能规范，也不规定用户安装 Skill 后的图片输出位置。
- 在当前仓库执行测试时，将临时生成的图片统一保存到仓库根目录的 `output/` 文件夹；如目录不存在，测试前先创建它。
- 不得因为本节约定修改 `SKILL.md`、提示词、脚本或其他正式资源，使安装后的 Skill 默认创建、依赖或引用 `output/` 目录。
- Skill 安装后的输出位置应继续由实际运行环境、用户请求或 Skill 自身的正式规则决定，不能硬编码为本仓库的 `output/`。
- `output/` 仅用于当前仓库的本地测试产物，整个目录已被 `.gitignore` 忽略，不得使用强制暂存方式将其中内容提交、打包或推送到远端。
- 需要交付为正式 Skill 资源的图片不属于测试产物，应放入对应的 `assets/` 目录并正常提交。

---
> Source: [Niall-Young/openai-cover-skill](https://github.com/Niall-Young/openai-cover-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
