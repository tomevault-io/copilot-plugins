## openfrence

> Stardock Fences 的开源替代。Windows 桌面图标分组管理工具，使用 C++/Win32 原生构建。

# openFences

Stardock Fences 的开源替代。Windows 桌面图标分组管理工具，使用 C++/Win32 原生构建。

## 核心架构

- **语言**: C++17（MSVC v141 工具链，VS2017），`CMAKE_CXX_STANDARD 20`
- **窗口**: Win32 原生 `WS_CHILD` 窗口，挂载在 explorer 的 `SysListView32` 桌面图标列表下
- **渲染**: Direct2D + DirectWrite + WIC，`WS_EX_LAYERED` + `UpdateLayeredWindow` 管线
- **序列化**: nlohmann/json，配置文件 `%APPDATA%\openFences\config.json`
- **构建**: CMake 3.20+，MSVC（VS2017 generator），x64，`/MT` 静态链接 CRT
- **产物**: 单 exe（~780KB），无依赖，便携运行

## 源文件（`cpp/src/`）

| 文件 | 职责 |
|------|------|
| `main.cpp` | 入口、托盘菜单、桌面图标管理、隐藏/显示调度 |
| `fence_window.cpp` | Fence 窗口：创建、拖拽、缩放、折叠、重命名、右键菜单 |
| `render.cpp` | Direct2D 渲染管线：毛玻璃（Acrylic）、边框、文字、布局 |
| `context_menu.cpp` | Win11 风格 Fluent 自绘菜单 + LL 钩子关闭 |
| `settings_panel.cpp` | 外观设置面板（背景色、标题色、对齐、字体大小等） |
| `config.cpp` | 配置持久化（读写 JSON、自动保存、开机自启注册表） |
| `icon_extract.cpp` | 从文件/快捷方式提取图标像素数据 |
| `menu_icons.cpp` | 菜单项图标的程序化光栅化（矢量字形） |

## 构建与运行

```bash
cmake --build cpp/build --config Release --target openfences
cpp/build/Release/openfences.exe
```

## 编码约定

- 无注释解释"做什么"（命名即文档），注释仅解释"为什么"
- 禁止每帧/热路径上的堆分配（笔刷缓存复用 `SetColor`）
- 所有 GDI/D2D/WIC 资源在窗口销毁时 RAII 释放
- UI 统一：颜色、间距、字体等全局常量定义，禁止硬编码

---
> Source: [weiweigogo/openFrence](https://github.com/weiweigogo/openFrence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
