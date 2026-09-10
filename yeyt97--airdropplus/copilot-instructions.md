## airdropplus

> 本文件为本项目提供给 Claude Code 的指引。

# CLAUDE.md

本文件为本项目提供给 Claude Code 的指引。

## 运行环境

- 本项目基于 **miniforge3 的 `adp` conda 环境**运行。所有依赖、打包工具（pyinstaller）以及国际化工具（Babel / `pybabel`）都安装在该环境中。
- 运行命令前请先激活环境：`conda activate adp`。
- 该环境的可执行文件位于 `~\miniforge3\envs\adp\Scripts\`（如 `pybabel.exe`）。系统 PATH 中没有全局可用的 python / pybabel，必须走 adp 环境。

## 项目简介

AirDrop Plus —— Windows 与 iOS 之间的文件传输与剪贴板同步工具。PC 端为 Python 编写的本地 HTTP 服务（仅限 Windows），手机端配合 iOS 快捷指令（Shortcut）使用。通过局域网或热点通信，不经第三方服务器。

## 常用命令

```bash
conda activate adp

# 从源码运行
python src/AirDropPlus.py

# 编译翻译（修改 .po 后必须执行，运行时加载的是 .mo）
cd src && pybabel compile -d translations

# 提取/更新翻译模板（新增 _() 文案后）
cd src && pybabel extract -o translations/messages.pot . && \
  pybabel update -i translations/messages.pot -d translations

# 打包为独立 exe（产物在 src/dist/）
python src/build.py
```

## 代码结构（src/）

- `AirDropPlus.py` —— 程序入口：加载配置、初始化 i18n、启动 HTTP 服务、创建托盘图标。
- `server.py` —— Flask 服务，定义所有 HTTP 路由（文件、剪贴板、设置页）。
- `clipboard.py` —— Windows 剪贴板读写（文本 / 图片 / 文件）。
- `notifier.py` —— Windows 交互式通知（windows-toasts）与基础通知样式。
- `config.py` —— 读写 `config/config.ini`，含校验逻辑。
- `i18n.py` —— 基于标准库 gettext 的轻量国际化，全局单一翻译器，切换语言即时生效。
- `utils.py` —— 工具函数（本机 IP、端口占用、文件名处理、路径编解码）。
- `result.py` —— 统一的 HTTP 响应封装。
- `build.py` —— 打包脚本（编译翻译 + pyinstaller）。

## 国际化（i18n）约定

- 支持语言：英文（en）、简体中文（zh）、俄文（ru）。
- 所有面向用户的文案都用 `_()` 包裹，msgid 用英文原文。
- 翻译资源在 `src/translations/<lang>/LC_MESSAGES/`：`.po` 为源文件，`.mo` 为运行时加载的编译产物。
- **改动文案后务必重新 `pybabel compile`**，否则运行时不会生效。
- `.mo` 体积小且 clone 即用，已纳入版本库（见 .gitignore 注释）。

### ⚠️ 翻译易踩的坑（务必遵守）

- **跑完 `pybabel update` 后必须检查并清理 fuzzy 标记**。`update` 的模糊匹配常把原本译文正确的条目误标成 `#, fuzzy`，而 `pybabel compile` 会**静默跳过所有 fuzzy 条目**，运行时这些条目会回退成英文原文 —— 表现为"页面部分中文突然变英文"。清理命令：检查 `grep -c "#, fuzzy" translations/<lang>/LC_MESSAGES/messages.po`，逐条确认译文无误后删掉 `#, fuzzy` 行，再 `compile`。
- **英文（en）的 msgstr 必须等于 msgid 本身**，不能留空、也不能被 fuzzy 匹配成别的字符串（gettext 对 en 不会 fallback 到 msgid，msgstr 为空就显示空）。
- **`templates/*.html` 里的 `{{ _('...') }}` 文案不会被默认的 `pybabel extract` 抽取**：需要 `-F babel.cfg`（已配置 jinja2 提取器），但当前 adp 环境未装 jinja2 提取插件，且 `babel.cfg` 含中文注释会在 GBK 环境下报 `UnicodeDecodeError`。因此模板新增文案目前靠**手动**往三份 `.po` 补 msgid/msgstr，不要依赖 extract。


## 配置与隐私

- 运行时配置 `src/config/config.ini` 含隐私数据，不纳入版本库；仓库只跟踪 `config.ini.example` 模板。
- 首次运行需从模板复制：`cp src/config/config.ini.example src/config/config.ini`。

## 语言

与用户交流默认使用中文。

---
> Source: [yeyt97/AirDropPlus](https://github.com/yeyt97/AirDropPlus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
