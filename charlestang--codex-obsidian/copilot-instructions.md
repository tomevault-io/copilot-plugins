## codex-obsidian

> 这个仓库是 Codex + Obsidian 的本地知识库工具包。Codex 通过 `skills/` 中的 Agent Skills 维护一个可长期演化的 Obsidian wiki。

# codex-obsidian: Codex 项目说明

这个仓库是 Codex + Obsidian 的本地知识库工具包。Codex 通过 `skills/` 中的 Agent Skills 维护一个可长期演化的 Obsidian wiki。

## 核心原则

- `.raw/` 是原始资料区。Codex 可以读取，但不修改原始文件。
- `wiki/` 是 Codex 生成和维护的知识库。
- `wiki/hot.md` 是热缓存，保存最近上下文，控制在约 500 字。
- `wiki/index.md` 是主索引，每次新增或重命名页面都要更新。
- `wiki/log.md` 是操作日志，新条目追加在文件顶部。
- 所有新 wiki 页面默认使用简体中文，frontmatter 写 `language: zh`。
- Obsidian 链接使用 `[[页面名]]`，不要用普通 Markdown 路径链接代替内部链接。

## Skills 发现

所有 skill 位于 `skills/<name>/SKILL.md`。安装到 Codex：

```bash
bash bin/setup-multi-agent.sh
```

等价手工安装：

```bash
ln -s "$(pwd)/skills" ~/.codex/skills/codex-obsidian
```

## 主要 Skills

| Skill | 用途 |
|---|---|
| `wiki` | 初始化 vault、路由子任务、维护 hot cache |
| `wiki-ingest` | 摄取文件、URL、图片，生成 source/entity/concept 页面 |
| `wiki-query` | 从 hot cache、索引和相关页面中回答问题 |
| `wiki-lint` | 检查孤儿页、死链、缺口和陈旧声明 |
| `save` | 保存当前对话、结论或决策 |
| `autoresearch` | 多轮自主研究并归档 |
| `canvas` | 创建和维护 Obsidian Canvas |
| `wiki-mode` | LYT / PARA / Zettelkasten / generic 路由 |
| `wiki-retrieve` | BM25 + contextual prefix + rerank 检索 |

## 迁移边界

这个仓库本身不把根目录 `wiki/` 当源码维护。`wiki/`、`.raw/`、`.obsidian/` 是 `bin/setup-vault.sh` 生成的运行态内容。示例 vault 如需添加，应放在 `examples/`，不要污染主工具包。

## 启动流程

新项目第一次使用时：

1. 运行 `bash bin/setup-multi-agent.sh` 安装 skills。
2. 运行 `bash bin/setup-vault.sh [vault 路径]` 生成 vault 结构。
3. 在 Obsidian 中打开 vault。
4. 在 Codex 中触发 `/wiki` 或说“设置这个 wiki”。

如果 `wiki/hot.md` 存在，Codex 应先静默读取它，再继续当前任务。

## 来源

本项目迁移自 `AgriciDaniel/claude-obsidian`。改造目标是 Codex-first 和中文-first。保留原项目许可与 attribution。

---
> Source: [charlestang/codex-obsidian](https://github.com/charlestang/codex-obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
