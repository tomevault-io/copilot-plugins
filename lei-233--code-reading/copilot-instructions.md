## code-reading

> >


# Skill：代码阅读分析

> 导航中心。**根据子命令只读对应 workflow，不要预读其他文件。**

## 子命令路由

| 调用方式 | 加载文件 | 预计耗时 |
|----------|----------|----------|
| `/code-reading quick` | `workflows/quick.md` | ~1 分钟 |
| `/code-reading normal` | `workflows/normal.md` | ~3 分钟 |
| `/code-reading deep` | `workflows/deep.md` | ~8 分钟 |

未指定子命令时默认执行 `quick`。

## 速度优化原则

- **文件读取并行化**：workflow 中标注 `[并行读取]` 的步骤，同时发起所有读取，不等待串行完成
- **SVG 默认关闭**：默认只生成 Markdown；需要 SVG 时在命令后加 `svg`，如 `/code-reading normal svg`
- **输出长度受控**：各章节有字数上限，不展开不必要的细节

## 输入

| 参数 | 必填 | 说明 |
|------|------|------|
| `repository_path` | ✅ | 代码库本地路径 |
| `target_scope` | ❌ | `full`（默认）或模块路径 |
| `svg` | ❌ | 追加此词时生成 SVG（normal/deep 生效） |

## 进度机制

每级完成后写入 `{repository_path}/analysis_output/.reading-state.md`。  
后一级执行时先读取此文件，若存在则**复用已有输出，仅补充缺失内容**。

## 约束

- 禁止全量读取代码库，遵循关键路径优先
- 每项结论附源文件路径和行号（`../` 相对路径可点击格式）
- SVG 节点文字默认中文

---
> Source: [Lei-233/code-reading](https://github.com/Lei-233/code-reading) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
