## cac-final-test

> cd code-fish && pip install -r requirements.txt

# AGENTS.md

## Build & Test

```bash
# 安装依赖
cd code-fish && pip install -r requirements.txt

# 验证题目格式
python scripts/validate_questions.py

# 运行测试
python cac.py --scope math              # 测试数理题库
python cac.py --scope math/base-test    # 测试基础题
python cac.py --scope math --range 001-005  # 指定范围
python cac.py --mode all --scope math -j 4  # 测试+评分，4并发
python cac.py --scope logic --dry-run   # 预览题目
```

## Architecture Overview

CLI 驱动的 LLM 评测系统。`cac.py` 为入口，支持交互式菜单和命令行模式。`src/scope.py` 解析测试范围，`src/runner.py` 并发执行测试，`src/judge.py` 调用评判模型打分，`src/reporting.py` 输出结果（Rich/Plain）。`src/providers/` 封装 OpenAI/Anthropic/Gemini/Custom 四种 LLM 接口。题库按 `{类别}/{难度}/{NNN-题目名}/` 组织，每题包含 `meta.yaml`、`prompt.md`、`reference.md`，结果输出到 `test-results/`。

## Security

- API Key 通过 `config.yaml` 配置，支持 `${ENV_VAR}` 环境变量引用
- `config.yaml` 已加入 `.gitignore`，禁止提交
- 敏感文件：`config.yaml`、`.env`、`*credentials*`
- 无远程端点暴露，纯本地 CLI 工具

## Git Workflows

- 主分支：`main`
- 开发分支：`main-fish`
- Commit 格式：`[emoji] [type]: [描述]`
  - 类型：`feat|fix|docs|style|refactor|perf|test|build|ci|chore|deps`
  - 示例：`🎨 style: Panel全宽显示统一边框对齐`
- Pre-commit Hook：验证 commit message 格式和题目格式
- 启用 hooks：`git config core.hooksPath .githooks`

## Conventions & Patterns

```
CAC-final-test/
├── code-fish/              # 评测系统
│   ├── cac.py              # CLI 入口
│   ├── config.yaml         # 模型配置（gitignore）
│   └── src/
│       ├── config.py       # 配置加载
│       ├── scope.py        # Scope 解析
│       ├── runner.py       # 测试执行
│       ├── judge.py        # 评分执行
│       ├── reporting.py    # 输出报告
│       ├── interactive.py  # 交互菜单
│       └── providers/      # LLM 适配器
├── 数理能力基准测试题库/    # 题库
├── 代码能力基准测试题库/
├── 自然语言与逻辑能力基准测试题库/
├── 幻觉控制与指令遵循测试/
└── scripts/                # 工具脚本
```

- 代码风格：Python 3.10+，UTF-8 编码
- 命名：snake_case（变量/函数），PascalCase（类）
- 题目 ID：`{category}-{difficulty}-{number}`（如 `math-base-001`）
- 输出文件：`test-results/{model-name}.md`、`{model-name}.judge.yaml`

---
> Source: [T-Auto/CAC-final-test](https://github.com/T-Auto/CAC-final-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
