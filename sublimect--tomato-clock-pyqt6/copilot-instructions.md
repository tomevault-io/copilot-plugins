## tomato-clock-pyqt6

> - 这是一个 pyqt6 项目, 必须使用 pyqt6 相关的 API

## 规范
- 这是一个 pyqt6 项目, 必须使用 pyqt6 相关的 API
- 禁止直接使用 `python` / `python3` / `pip` / `pip3` 命令, 必须使用 `uv` 命令完成任务
- 禁止使用 `pyside6`, 必须使用 `pyqt6`
- 所有改动必须保证在应用打包后, 生产环境下可用
- 代码必须符合基本的开发原则和设计模式, 每个文件不允许超过 300 行
- 禁止使用 QML, 必须使用 QWidget 风格
- 全部涉及 pyqt / qt 的不确定的 API, 都必须调用 context7 获取确切的信息

---
> Source: [SublimeCT/tomato-clock-pyqt6](https://github.com/SublimeCT/tomato-clock-pyqt6) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
