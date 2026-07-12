## kj-skills

> 本仓库是本地 Agent Skill 的集合。每个 skill 位于 `skills/<skill-id>/`，入口文件为 `SKILL.md`。较长的支撑材料放在 `references/`，可执行脚本放在 `scripts/`，可复用示例放在 `examples/`，演示或报告资产放在 `assets/`。仓库级文档位于 `README.md`、`CONTRIBUTING.md` 和 `docs/`。Claude、Cursor、Codex 和 Agents 的插件元数据分别在 `.claude-plugin/`、`.cursor-plugin/`、`.codex-plugin/` 和 `.agents/plugins/`。

# 仓库规范

## 项目结构与模块组织

本仓库是本地 Agent Skill 的集合。每个 skill 位于 `skills/<skill-id>/`，入口文件为 `SKILL.md`。较长的支撑材料放在 `references/`，可执行脚本放在 `scripts/`，可复用示例放在 `examples/`，演示或报告资产放在 `assets/`。仓库级文档位于 `README.md`、`CONTRIBUTING.md` 和 `docs/`。Claude、Cursor、Codex 和 Agents 的插件元数据分别在 `.claude-plugin/`、`.cursor-plugin/`、`.codex-plugin/` 和 `.agents/plugins/`。

## 构建、测试与开发命令

- `npm run sync`：将全部 skill 软链到 `~/.claude/skills/`。
- `npm run sync:one -- <skill-name>`：按需安装单个（或多个）skill 到 `~/.claude/skills/`。
- `npm run sync:codex`：将全部 skill 软链到 `$CODEX_HOME/skills/` 或 `~/.codex/skills/`。
- `npm run sync:all`：同时更新 Claude 和 Codex 的 skill 注册。
- `./sync-codex-skills.sh content-creator git-push`：仅注册指定的 skill。
- `WI_SKIP_CLAUDE_TESTS=1 bash skills/wiki-intelligence/tests/test-wiki-intelligence.sh`：不调用真实 Claude 的情况下运行 wiki-intelligence shell 测试。

## 代码风格与命名规范

skill 目录使用小写 kebab-case，例如 `skills/ai-coding-weekly-report`。入口文件固定命名为 `SKILL.md`，以 YAML frontmatter 开头，后接简洁的 Markdown 说明。Shell 脚本使用 Bash，路径加引号，新脚本优先使用 `set -euo pipefail`。Python 辅助脚本放在所属 skill 目录下，保持 CLI 风格、体积小巧。Markdown 标题要有描述性，示例要可直接运行，参考资料放在使用它的 skill 附近。

## 测试规范

目前没有全仓库统一的测试运行器。在功能旁边添加针对性测试，参照 `skills/wiki-intelligence/tests/test-*.sh` 的模式编写 shell 测试库。对于需要调用外部工具或模型的 skill，提供环境变量以跳过真实集成调用，如 `WI_SKIP_CLAUDE_TESTS=1`。提交前运行相关脚本测试，若注册行为有变动还需运行至少一条 sync 命令。

## 提交与 PR 规范

Git 历史遵循 Conventional Commits，例如 `feat(skills): add multiple new skills`、`docs(ai-daily-from-x): introduce new skill`。格式为 `type(scope): summary`，scope 可为 `skills`、`CONTRIBUTING` 或某个 skill id。每个 PR 聚焦于单个 skill 或单项文档变更。PR 描述应列出变更的 skill、更新的文档或插件元数据、已执行的测试命令，以及有意跳过的检查项。禁止提交真实的 API Key、Token、私有 wiki 内容或仅限本机的配置。

## Agent 专项说明

涉及信贷前端、滴水贷或信用购问题时，先读取 `/Users/didi/ai_space/loanfe-llm-wiki/wiki/index.md`，再查阅具体页面。完整的外部 wiki 规则见 `/Users/didi/ai_space/loanfe-llm-wiki/AGENTS.md`。

## local-stt-transcription 说明

- `skills/local-stt-transcription/` 是本地音视频转最终校对文字稿的统一 skill。
- 该 skill 的脚本和校对规则必须保持在 skill 目录内；外部仅依赖 `/Users/chengkangjian/work/stt` 这类 `jianchang512/stt` runtime 和模型目录。
- 新电脑或 runtime 缺失时，先使用 `skills/local-stt-transcription/scripts/bootstrap_stt_runtime.sh` 初始化。
- 对接 `kj-llm-wiki` 时，短视频输出目录为 `raw/studio/transcripts/`。该目录是已发布短视频的文字稿资产库，用于复盘已发布内容并做连载选题规划；规划结果应标注来源文字稿、所属系列和下一集角度。

## 语言规范

本仓库所有 `AGENTS.md` 文件（包括子目录下的）**必须使用中文**撰写。新增或修改 skill 时，若需为该 skill 创建 `AGENTS.md`，同样使用中文。技术术语、命令、代码标识符保持原文，说明文字用中文。

---
> Source: [chengkj99/kj-skills](https://github.com/chengkj99/kj-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
