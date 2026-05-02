## claude-cookbooks-zh

> 一组用于构建 Claude API 的 Jupyter 笔记本和 Python 示例。

# Claude 菜谱

一组用于构建 Claude API 的 Jupyter 笔记本和 Python 示例。

## 快速开始

```bash
# 安装依赖
uv sync --all-extras

# 安装 pre-commit 钩子
uv run pre-commit install

# 设置 API 密钥
cp .env.example .env
# 编辑 .env 并添加你的 ANTHROPIC_API_KEY
```

## 开发命令

```bash
make format        # 使用 ruff 格式化代码
make lint          # 运行代码检查
make check         # 运行格式检查 + 代码检查
make fix           # 自动修复问题 + 格式化
make test          # 运行 pytest
```

或直接使用 uv：

```bash
uv run ruff format .           # 格式化
uv run ruff check .            # 检查
uv run ruff check --fix .      # 自动修复
uv run pre-commit run --all-files
```

## 代码风格

- **行长度：** 100 个字符
- **引号：** 双引号
- **格式化工具：** Ruff

笔记本文件对文件中间的导入（E402）、重新定义（F811）以及变量命名（N803, N806）有较宽松的规则。

## Git 工作流程

**分支命名：** `<用户名>/<功能描述>`

**提交格式（约定式提交）：**
```
feat(scope): add new feature
fix(scope): fix bug
docs(scope): update documentation
style: lint/format
```

## 关键规则

1. **API 密钥：** 切勿提交 `.env` 文件。始终使用 `os.environ.get("ANTHROPIC_API_KEY")`

2. **依赖项：** 使用 `uv add <包名>` 或 `uv add --dev <包名>`。切勿直接编辑 pyproject.toml 文件。

3. **模型：** 使用当前的 Claude 模型。请查阅 docs.anthropic.com 获取最新版本。
   - Sonnet: `claude-sonnet-4-5-20250929`
   - Haiku: `claude-haiku-4-5-20251001`
   - Opus: `claude-opus-4-5-20251101`

4. **笔记本：**
   - 在笔记本中保留输出内容（有意为之，便于演示）
   - 每个笔记本只讲解一个概念
   - 确保笔记本能够从上到下无错误运行

5. **质量检查：** 提交前运行 `make check`。预提交钩子会验证格式和笔记本结构。

## 斜杠命令

这些命令可在 Claude Code 和 CI 中使用：

- `/notebook-review` - 审查笔记本质量
- `/model-check` - 验证 Claude 模型引用
- `/link-review` - 检查已更改文件中的链接

## 项目结构

```
capabilities/      # Core Claude capabilities (RAG, classification, etc.)
skills/            # Advanced skill-based notebooks
tool_use/          # Tool use and integration patterns
multimodal/        # Vision and image processing
misc/              # Batch processing, caching, utilities
third_party/       # Pinecone, Voyage, Wikipedia integrations
extended_thinking/ # Extended reasoning patterns
scripts/           # Validation scripts
.claude/           # Claude Code commands and skills
```

## 添加新菜谱

1. 在相应目录中创建笔记本
2. 在 `registry.yaml` 中添加条目，包含标题、描述、路径、作者和分类
3. 如果是新贡献者，请在 `authors.yaml` 中添加作者信息
4. 运行质量检查并提交 PR

---

---
> Source: [zk-b612/claude-cookbooks-zh](https://github.com/zk-b612/claude-cookbooks-zh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
