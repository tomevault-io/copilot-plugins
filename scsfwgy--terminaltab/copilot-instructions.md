## terminaltab

> 本文件为 Claude Code (claude.ai/code) 在本仓库中工作时提供指导。

# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在本仓库中工作时提供指导。

## 项目概述

TerminalTab 是一个 zsh 插件，按 Ctrl+L 键调用大模型 API 提供命令建议，按 Ctrl+G 向 AI 提问。支持纠正拼写错误、补全部分输入、或为已有命令推荐有用的参数组合。

仅两个文件，除 `curl` + `jq` 外零依赖：
- `ai-suggest.sh` — Bash 脚本，调用 LLM API，返回换行分隔的完整命令建议
- `ai-complete.zsh` — ZLE（Zsh 行编辑器）插件，拦截 Ctrl+L/Ctrl+G/上/下/回车 键

## 架构

```
用户按 Ctrl+L (l = list)
  → ai-complete.zsh (_ai_trigger widget)
    → ai-suggest.sh（通过 &! 后台运行）
    → 通过 POSTDISPLAY 在输入内容后显示单行 loading 动画
    → _ai_show: zle redisplay → 清理旧列表 → DEC 保存光标 → printf 边框列表 → DEC 恢复光标
  → 用户按 上/下
    → _ai_show 重新渲染列表，更新选中项到 LBUFFER
  → 用户按 回车
    → 选中命令填入缓冲区，列表清除并重置状态
  → 用户按 Ctrl+C
    → 恢复原始输入，列表清除并重置状态
```

状态通过 zsh 会话中的全局 `_AI_*` 变量管理。`_AI_ACTIVE` 标志控制方向键和回车键是路由到 AI 导航还是回退到正常 zsh 行为。

## 配置

```bash
# 在 .zshrc 中：
export AI_COMPLETE_API_KEY="sk-..."
export AI_COMPLETE_MODEL="gpt-4o-mini"                    # 必填
export AI_COMPLETE_API_URL="https://api.openai.com/..."    # 必填
export AI_COMPLETE_API_TYPE="openai"                       # 可选：openai（默认）或 claude
export AI_COMPLETE_MAX_ITEMS=5                             # 可选
source ~/path/to/TerminalTab/ai-complete.zsh
```

## 关键经验（ZLE + 终端显示）

以下是开发中发现的重要踩坑点，后续修改必须遵守：

### 1. ZLE 显示管理
- **绝不**在未与 ZLE 协调的情况下直接 `printf` 到终端。ZLE 的自动刷新（widget 返回时）会覆盖或破坏它不知道的任何内容。
- 渲染列表的正确顺序：先 `zle redisplay`（让 ZLE 处理命令行），再清理旧列表，然后 `\e7`（DEC 保存光标），接着 `printf` 列表内容，最后 `\e8`（DEC 恢复光标）。使用 DEC 保存/恢复而非 CSI s/u，避免与 ZLE 内部冲突。
- 列表清理逻辑必须统一，避免 `_ai_show` / Enter / Ctrl+C 各自使用不同的 ANSI 序列，否则容易出现残影、错位和状态不同步。
- 导航（上/下）时旧列表必须先清掉再重绘；关闭菜单时也必须走同一套清理路径。
- 如果已经记录了菜单高度（如 `_AI_LIST_LINES`），清理时应按实际行数逐行清除，而不是直接 `\e[J` 清到屏幕底部，避免误清终端其它内容。

### 2. Loading 动画与列表显示要分层处理
- 单行 loading 动画适合用 `POSTDISPLAY`，这样可以直接显示在用户当前输入内容后面，比如 `ls ⠧`。
- 多行边框列表**不能**用 `POSTDISPLAY`，因为它只支持单行；多行列表仍需使用 `printf` + 光标保存/恢复来绘制。
- `POSTDISPLAY` 使用完后必须及时清空，否则会残留在命令行尾部。

### 3. `zle -R "" list...` 不适合自定义布局
- zsh 的补全列表系统会按字母排序，破坏原有顺序（如第一个建议应该默认选中）。
- 它还会自动横向排列成多列。用 200+ 字符填充强制单列属于 hack 方案，且会导致 `item=` 变量回显问题。
- `zle -R "多行\n消息"` 方式不会渲染多行字符串——只显示第一行。

### 3. `POSTDISPLAY` 仅支持单行
- 尽管文档暗示支持多行，`POSTDISPLAY` 只在光标后同一行内渲染。含 `\n` 的多行内容不会显示为多行。

### 4. 后台任务通知
- 必须使用 `{ cmd & } &!` 语法——不能仅用 `&` 加 `disown`。`&!`（zsh 专用）会立即剥离任务，防止出现 `[N] + done cmd...` 通知。
- **不要**在 ZLE widget 内使用 `wait $pid`——它会触发任务完成通知。改用 `kill -0 $pid` 轮询循环。

### 5. ZLE widget 循环内的变量声明
- ZLE widget 的 `for` 循环内使用 `local var` 会导致赋值被回显到终端（显示为 `item='value'` 文本）。所有变量需在循环外声明，循环内赋值。

### 6. 建议结果清洗必须在源头完成
- `ai-suggest.sh` 返回值不能直接信任。模型可能返回编号列表、项目符号、代码块围栏、空行、重复命令。
- 应在 `ai-suggest.sh` 内统一做清洗：trim、去编号/项目符号、去代码块残留、去重、保序、限制最大条数。
- 前端菜单应尽量只处理“已经清洗好的完整命令列表”，不要把脏数据留给 `ai-complete.zsh` 再兜底。

### 7. 菜单关闭必须统一重置状态
- 关闭菜单时不能只清屏，还必须统一重置 `_AI_ACTIVE`、`_AI_SUGGESTIONS`、`_AI_INDEX`、`_AI_SCROLL`、`_AI_LIST_LINES`。
- Enter 接受和 Ctrl+C 取消都应复用同一套状态重置逻辑，否则下一次打开菜单时容易出现状态残留。

### 8. Esc 键绑定冲突
- **不要** `bindkey '\e'`（裸 Escape）——它会与方向键序列（`\e[A`、`\e[B`）冲突，因为 Escape 是所有 CSI 序列的起始。改用 `^C` 取消。

### 9. zsh 字符串中的 ANSI 转义码
- `\e[7m`（反色显示）必须用 `$'\e[7m'`（`$'...'` 引用语法），不能用双引号中的 `\e[7m`，后者会原样输出 `e[7m` 文本。

## 扩展

- 修改 LLM 提示词：编辑 `prompts/suggest.prompt` 和 `prompts/ask.prompt`
- 修改最大显示条目数：`export AI_COMPLETE_MAX_ITEMS=N` 或编辑 `_AI_MAX_ITEMS` 默认值
- 修改边框样式：编辑 `_ai_show()` 中的 `printf` 格式字符串
- 边框宽度根据最长可见条目自动计算（最小 15，最大 50 字符）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scsfwgy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
