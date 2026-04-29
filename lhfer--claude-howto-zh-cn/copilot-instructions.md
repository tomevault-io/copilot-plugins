## claude-howto-zh-cn

> 这个文件给 Claude Code 提供“如何在这个仓库里协作”的工作说明。

# CLAUDE.md

这个文件给 Claude Code 提供“如何在这个仓库里协作”的工作说明。

## 项目定位

`claude-howto-zh-cn` 是一个 **documentation-as-code** 仓库。

- 主体产物是 Markdown 教程与示例，不是可执行应用
- 核心目标是把上游 `luongnv89/claude-howto` 做成更适合中国小白的中文主线指南
- 兼容性优先级很高：命令名、frontmatter、JSON/YAML key、环境变量、路径约定都不能被本地化改坏

## 你最常用的命令

### 本地化校验

```bash
uv run python scripts/validate_localization.py
```

检查重点包括：

- frontmatter 是否可解析
- 关键保留词是否被误翻
- Markdown 链接是否有效
- JSON / YAML / shell 脚本是否还能正常解析

### 测试脚本

```bash
uv run pytest scripts/tests/ -q
```

### EPUB 构建

```bash
uv run scripts/build_epub.py
```

### Python 质量检查

```bash
uv run ruff check scripts/
uv run ruff format scripts/
```

## 仓库结构

```text
01-slash-commands/      slash commands 教程与模板
02-memory/              CLAUDE.md / memory 教程与模板
03-skills/              skills 教程与示例
04-subagents/           subagents 教程与示例
05-mcp/                 MCP 教程与配置示例
06-hooks/               hooks 教程与脚本示例
07-plugins/             plugins 教程与完整样例
08-checkpoints/         checkpoints / rewind 教程
09-advanced-features/   高级能力说明
10-cli/                 CLI 指南
.claude/skills/         仓库内置的教学 skills
scripts/                校验、构建、测试脚本
```

## 修改文档时的核心原则

1. 先保兼容，再谈翻译  
   任何会被 Claude Code 直接读取或执行的标识，优先保留原样。

2. 先讲用途，再讲命令  
   中文读者更容易先理解“这是什么 / 什么时候用 / 为什么有价值”，再接受 CLI、配置和脚本细节。

3. 不要把正文写成翻译腔  
   优先自然中文表达；必要时保留 `skills`、`CLI`、`hooks`、`MCP`、`subagents` 这类英文术语。

4. 高风险文件少改、谨慎改  
   `.sh`、`.py`、`.json`、`.yml` 默认只同步必要的兼容性变化；注释能不动就不动。

## 修改示例文件时要特别注意

以下内容默认不要翻译：

- 文件名、目录名
- slash command 名称
- skill / subagent / plugin 名称
- YAML frontmatter key
- JSON / YAML key
- CLI flags
- 环境变量名
- MCP server 名
- 代码块里的可执行命令

## 推荐提交流程

1. 改文档或示例
2. 跑 `uv run python scripts/validate_localization.py`
3. 跑 `uv run pytest scripts/tests/ -q`
4. 如果改了 EPUB 构建，再跑 `uv run scripts/build_epub.py`
5. 在 `README.md`、`UPSTREAM.md`、`CHANGELOG.md` 里记录最近同步内容

## 提交信息风格

优先使用 conventional commits，例如：

- `docs(readme): sync upstream April 2026 updates`
- `feat(subagents): add performance-optimizer example`
- `fix(hooks): switch shell hooks to stdin JSON protocol`
- `refactor(epub): polish zh-cn cover and reading experience`

---
> Source: [lhfer/claude-howto-zh-cn](https://github.com/lhfer/claude-howto-zh-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
