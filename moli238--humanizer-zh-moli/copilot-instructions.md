## humanizer-zh-moli

> 供维护本仓库的 AI 编程 Agent 使用。

# AGENTS.md

供维护本仓库的 AI 编程 Agent 使用。

## 仓库定位

这是一个便携式中文写作 skill。`SKILL.md` 是运行时行为的唯一来源；仓库没有构建步骤，也不应把支持范围限定在某一个 Agent 产品。

本仓库是莫理帮助归藏（歸藏）迭代的 Humanizer-zh 分支，继承 `blader/humanizer`。不要删除 README 和 LICENSE 中的来源、贡献与版权说明。

## 维护契约

- `SKILL.md` frontmatter 只保留 `name` 和 `description`，避免加入 `allowed-tools` 等宿主专用字段。
- 当前共有 36 个编号模式。增删或重排时，同步更新 README 的模式表和 `scripts/validate-package.py` 中的数量。
- 行为以事实保真为第一优先级。示例不得展示“把模糊说法改成凭空具体事实”的错误做法。
- 中文规则按中文习惯编写。英文标题大小写、连字符、引号等只作为中英混排条件规则，不要机械翻译上游偏好。
- 插件版本与 README 版本记录必须同步。Agent Skills frontmatter 不存版本。
- 非显而易见的行为调整要在 README 版本记录中说明原因。
- 保持宿主中立。Claude Code、Codex、OpenCode 等是兼容示例，不是支持边界。

## 校验

发布前运行：

```bash
python scripts/validate-package.py
npx skills add . --list
claude plugin validate .
```

---
> Source: [moli238/humanizer-zh-moli](https://github.com/moli238/humanizer-zh-moli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
