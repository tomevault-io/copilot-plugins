## flex-writer-share

> 仓库面向 AI 协作（Codex、Claude Code 等）的约定。任何 agent 在改这个 repo 之前必读。

# AGENTS.md

仓库面向 AI 协作（Codex、Claude Code 等）的约定。任何 agent 在改这个 repo 之前必读。

## 项目定位

装逼文案 skill 集合。每个 skill 同时给 Codex 和 Claude Code 使用，**作用域是项目级**（`./.codex/skills/` 和 `./.claude/skills/`），不污染全局。
克隆即生效——`.codex/skills/<name>` 和 `.claude/skills/<name>` 是仓库里提交的 symlink，指向 `skills/<name>/`。

设计文档：[docs/skill-structure.md](docs/skill-structure.md)。
写作指南：[docs/authoring.md](docs/authoring.md)。

## 必须遵守

1. 新增 skill 前先看 [docs/skill-structure.md](docs/skill-structure.md) 的"单 skill 布局"和 [docs/authoring.md](docs/authoring.md) 的写作清单。
2. frontmatter 必填项：`name`、`description`。可选但推荐：`metadata.version`、`metadata.author`、`metadata.category`、`license`。
3. body 内禁止使用 `${CLAUDE_SKILL_DIR}`、`$ARGUMENTS` 这类 Claude Code 专属变量。用相对路径。
4. 修改或新增 skill 后必须跑 `python3 tools/lint_frontmatter.py skills/<name>/`，确保零错误。
5. 跨平台验证：每改一个 skill，至少在一个 runtime 里跑过 example 才算完成。
6. 新增 skill 后跑 `tools/install_skills.sh`（默认项目级 symlink）让 `.codex/skills/<name>` 和 `.claude/skills/<name>` 同步生成；新增的 symlink 要进 git 仓库。

## 禁止事项

1. 不引入 Python/Node 之外的构建依赖（仓库不该有 requirements.txt/package.json）。
2. 不在 `skills/<name>/SKILL.md` 里硬编码平台专属字段而忘了另一边。
3. 不在没有 example 的情况下发布新 skill。
4. 不在 git 仓库根目录散落临时文件——临时验证用 `/tmp/`。
5. 不在 `.codex/` 或 `.claude/` 根下提交任何文件，只允许 `skills/` 子目录的 symlink 进来（gitignore 已限制）。

## 提交规范

1. commit message 用中文，描述做了什么（例：`新增 writing-headlines skill：公众号标题套路集合`）。
2. 一个 commit 只做一件事（新 skill / 改 skill / 改文档 / 改工具）。
3. 提交前跑 `tools/install_skills.sh` 自检一次（默认就是项目级 symlink 安装）。新增 skill 时会生成对应 symlink，记得 `git add .codex/ .claude/`。

---
> Source: [CaptivePig/flex-writer-share](https://github.com/CaptivePig/flex-writer-share) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
