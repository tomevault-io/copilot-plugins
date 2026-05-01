## python-runtime

> 项目 Python 运行时规则 — 所有涉及 Python 执行、包安装、脚本运行的操作必须遵守


# Python 运行时规则（强制）

本项目内嵌了独立的 Python 3.12 运行时，**严禁使用系统自带的 Python**。

## 必须使用的路径

| 用途 | 命令 |
|------|------|
| 运行 Python | `./python/bin/python3.12` |
| pip 安装 | `./python/bin/pip3.12 install <包名>` |
| 运行模块 | `./python/bin/python3.12 -m <模块>` |

## 绝对禁止

- **不要**使用 `python3`、`python`、`/usr/bin/python3` 或任何系统 Python
- **不要**使用 `pip`、`pip3` 或系统 pip — 只用 `./python/bin/pip3.12`
- **不要**编辑 `python/` 目录下的任何文件
- **不要**尝试升级项目内嵌 Python 本身

## 示例

```bash
# 正确 ✅
./python/bin/python3.12 -m pip install httpx
./python/bin/python3.12 script.py
./python/bin/python3.12 -m pytest tests/

# 错误 ❌
python3 -m pip install httpx
pip install httpx
python script.py
```

## 原因

系统 Python（macOS 自带 3.9）版本低且缺少项目依赖。所有依赖只安装在项目内嵌 Python 中。使用错误的 Python 会导致 `ModuleNotFoundError` 和兼容性问题。

---
> Source: [flywhale-666/whaleclaw](https://github.com/flywhale-666/whaleclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
