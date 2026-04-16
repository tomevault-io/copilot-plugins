## no-game-no-life

> 本项目采用“双模式”文件结构，旨在分离开发环境（拆分文件）与生产环境（单文件）。

# 项目工作流规则：双模式文件系统 (Dual-Mode File System)

本项目采用“双模式”文件结构，旨在分离开发环境（拆分文件）与生产环境（单文件）。
AI 助手必须严格遵守以下规则进行操作。

## 1. 后端开发规范 (Python)
- **禁止 (DO NOT)** 直接编辑根目录下的 `main.py`。该文件是自动生成的产物。
- **必须 (MUST)** 编辑 `src/backend/` 目录下的各个组件文件：
  - 导入与设置 -> `src/backend/00_imports.py`
  - 配置参数 -> `src/backend/01_config.py`
  - 路由/接口 -> `src/backend/10_handlers_*.py`
- 如果需要添加新的 Python 库，请添加到 `src/backend/00_imports.py`。

## 2. 前端开发规范 (HTML/JS/CSS)
- **禁止 (DO NOT)** 直接编辑根目录下的 `index.html`。
- **必须 (MUST)** 编辑 `src/js/` 或 `src/css/` 中的源文件。
- 如果涉及 HTML 结构的修改，请检查 `src/` 下是否有模板文件，或查看 `build.py` 中的前端构建逻辑。

## 3. 构建流程 (关键步骤)
- 在对 `src/backend/*.py` 或前端源文件进行 **任何** 修改后，**必须** 运行构建脚本：
  `python build.py`
- 该脚本会自动合成：
  1. `src/backend/*.py` -> `main.py` (位于根目录)
  2. 前端资源 -> `index.html` (位于根目录)

## 4. 验证
- 始终通过运行以下命令并检查控制台输出来验证服务器是否运行了最新代码：
  `python main.py`

## 总结
- **源码唯一真理 (Source of Truth)**：`src/` 目录。
- **部署交付物 (Deployment Artifacts)**：根目录下的 `main.py` 和 `index.html`。
- **永远不要** 手动修改这些交付物文件。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rwangrwang31) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
