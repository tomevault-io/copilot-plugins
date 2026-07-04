## acp-deepseek-adapter

> - 只读检查优先，不做破坏性修改（除非明确要求）

# acp-deepseek-adapter — 项目规则

## 核心原则
- 中文回复，简洁直接，不废话
- 只读检查优先，不做破坏性修改（除非明确要求）
- 每次修改后验证适配器能否加载：`python3 -c "import sys; sys.path.insert(0,'.'); import acp_deepseek_adapter"`

## 当前配置
- deepseek 二进制：`/Users/rk/deepseek`（已更新到 v0.8.32）
- cc-connect 配置：`/Users/rk/.cc-connect/config.toml`
- 适配器日志：`/tmp/deepseek-ccconnect.log`
- ADAPTER_HIDE_TOOLS=1, ADAPTER_STRIP_THINKING=0
- auto_compress：100k token / 15min 间隔

## 已知问题 & 约束
- `deepseek exec` 不支持 `--resume`，会话连续性无法保证
- `_TOOL_DONE_RE` 可能匹配到非 tool 完成行，导致 `_tool_depth` 提前归零
- `_stream_stderr` 中已知 NoneType 错误（raw_line 为 None）
- 飞书 Markdown：`#` 开头行需插入 `\u200B`，`---` 需转义

## 版本发布流程
1. 更新 `acp_deepseek_adapter.py` 中 3 处版本号
2. 更新 `README.md` Changelog
3. `git add` + `git commit` + `git push origin main`
4. 需要时同步更新 `cc-connect-config.toml`

## 测试方法
飞书发送消息到 deepseek-tui 机器人，观察：
- 无"史山代码"泄漏（源代码、diff、JSON 元数据）
- 分段输出清晰（`---` 分隔线可见）
- 末尾 `[ctx: ~X%]` 显示正常

---
> Source: [rockeverm3m/acp-deepseek-adapter](https://github.com/rockeverm3m/acp-deepseek-adapter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
