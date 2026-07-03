## claude-code-notify

> 发布新版本时，**必须同步更新以下两个文件的版本号**：

# Claude Code Notify - 开发指南

## 版本发布流程

发布新版本时，**必须同步更新以下两个文件的版本号**：

| 文件 | 用途 | 位置 |
|---|---|---|
| `pyproject.toml` | PyPI 包版本 | 第 7 行 `version = "x.x.x"` |
| `plugin/.claude-plugin/plugin.json` | Claude Code 插件市场版本 | 第 3 行 `"version": "x.x.x"` |

### 发布步骤

1. **更新版本号**（两个文件必须同步）
   ```bash
   # pyproject.toml
   version = "1.0.x"

   # plugin/.claude-plugin/plugin.json
   "version": "1.0.x"
   ```

2. **更新 README 更新日志**
   - `README.md` (English)
   - `README_CN.md` (中文)

3. **运行测试确保通过**
   ```bash
   pytest tests/ -v
   ```

4. **提交并推送**
   ```bash
   git add .
   git commit -m "chore: bump version to x.x.x"
   git push origin main
   ```

5. **创建 GitHub Release**
   ```bash
   gh release create vx.x.x --title "vx.x.x" --notes "Release notes..."
   ```
   - Tag 格式：`v1.0.x`（小写 v）
   - 这将自动触发 GitHub Action 发布到 PyPI

## 项目结构

```
claude-code-notify/
├── src/claude_code_notify/     # PyPI 包源码
│   └── notify.py
├── plugin/                     # Claude Code 插件
│   ├── .claude-plugin/
│   │   └── plugin.json         # 插件版本配置
│   └── hooks/
│       └── notify.py           # 插件 hook 脚本
├── tests/                      # 测试用例
│   └── test_notify.py
├── pyproject.toml              # PyPI 包配置
├── README.md                   # 英文文档
└── README_CN.md                # 中文文档
```

## 测试

```bash
# 安装测试依赖
pip install -e ".[test]"

# 运行所有测试
pytest tests/ -v

# 运行测试并生成覆盖率
pytest tests/ --cov=src/claude_code_notify --cov-report=html
```

## 调试

调试日志保存在 `/tmp/claude_code_notify_debug.log`

---
> Source: [starpipi/claude-code-notify](https://github.com/starpipi/claude-code-notify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
