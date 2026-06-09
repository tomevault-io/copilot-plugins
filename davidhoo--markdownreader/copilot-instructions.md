## markdownreader

> > 本文件为 Claude Code 提供项目上下文，确保代码修改遵循项目规范。

# CLAUDE.md — MarkdownReader 项目指南

> 本文件为 Claude Code 提供项目上下文，确保代码修改遵循项目规范。

## 项目简介

MarkdownReader 是一个原生 macOS Markdown 阅读器应用。不是编辑器，只是一个安静的阅读器。三栏布局：左侧目录树 + 中间渲染视图 + 右侧大纲导航。

- **当前版本**: 2.0.3
- **最低部署**: macOS 26.0
- **Bundle ID**: `com.markdownreader.app`
- **许可证**: MIT

## 技术栈

| 组件 | 选择 |
|------|------|
| 语言 | Swift 6.0（严格并发） |
| UI 框架 | SwiftUI |
| Markdown 渲染 | cmark-gfm + WKWebView（v2.x 已从 Textual 迁移） |
| 状态管理 | `@Observable`（非 ObservableObject） |
| 并发 | Swift Concurrency（async/await, `@MainActor`） |
| 构建系统 | Swift Package Manager |
| 本地化 | 自定义字典方案（L10n），非 Apple String Catalog |

## 项目结构

```
Sources/MarkdownReader/
├── App/
│   ├── MarkdownReaderApp.swift    # @main 入口，WindowGroup，菜单命令，Notification.Name 常量
│   └── AppDelegate.swift          # NSApplicationDelegate，冷/热启动文件处理
├── Models/
│   ├── FileNode.swift             # 目录树节点
│   ├── Document.swift             # 文档模型
│   ├── FileError.swift            # 错误类型枚举
│   ├── OutlineItem.swift          # 大纲标题模型（level, title, lineNumber）
│   ├── DisplayMode.swift          # 枚举：.rendered / .raw
│   ├── ThemeDefinition.swift      # 5 色主题 + 33 预设 + 自定义覆盖
│   └── SettingsModel.swift        # 设置单例（@Observable + UserDefaults）
├── ViewModels/
│   ├── AppViewModel.swift         # 全局 UI 状态（侧边栏、大纲、设置、窗口标题）
│   ├── DocumentViewModel.swift    # 文档加载/保存/脏跟踪/文件监控（最复杂，~568 行）
│   └── FileTreeViewModel.swift    # 目录树管理/键盘导航/文件操作
├── Views/
│   ├── ContentView.swift          # 主布局（~885 行），ViewModifier 事件处理模式
│   ├── SidebarView.swift          # 左侧目录树
│   ├── DetailView.swift           # 右侧主体区容器
│   ├── RenderedMarkdownView.swift  # WKWebView + cmark-gfm 渲染
│   ├── RawMarkdownView.swift      # 原文编辑包装
│   ├── SyntaxHighlightedEditor.swift  # NSTextView 编辑器（~534 行），per-file undo
│   ├── OutlineView.swift          # 右侧大纲面板
│   ├── SettingsView.swift         # 设置视图
│   ├── ResizeHandle.swift         # 侧边栏拖拽（NSViewRepresentable）
│   ├── OutlineResizeHandle.swift  # 大纲面板拖拽
│   ├── TrafficLightButtons.swift  # 自定义窗口控制按钮
│   ├── FileRowView.swift          # 文件/目录行
│   ├── WelcomeView.swift          # 空状态占位
│   └── ErrorView.swift            # 错误提示
└── Services/
    ├── FileService.swift          # 文件系统操作
    ├── OutlineService.swift       # Markdown 标题解析
    ├── ThemeColors.swift          # 主题色彩派生（5 色 → 12+ 语义 token）
    ├── MarkdownSyntaxHighlighter.swift  # 正则语法高亮（~729 行）
    ├── LanguageService.swift      # 系统语言检测
    ├── LocalizationService.swift  # L10n 字典（80+ 键，3 语言）
    ├── OpenPanelHelper.swift      # NSOpenPanel/NSSavePanel 工具
    └── FileSystemWatcher.swift    # FSEventStream 包装
```

## 构建与运行

```bash
# 构建（调试）
swift build

# 构建（发布）
swift build -c release

# 构建 .app 包（含签名）— arm64 only
./build-app.sh --release --sign

# 打包 DMG — arm64 only
./package.sh

# 本地发布到 GitHub
./release-local.sh
```

## 依赖

唯一外部依赖：**Textual** v0.3.1+（过渡期保留，渲染已迁移至 cmark-gfm + WKWebView）
- 传递依赖：swift-concurrency-extras 1.4.0, swiftui-math 0.1.0
- 锁定策略：`.upToNextMinor(from: "0.3.1")`（Textual 处于 v0.x，API 可能变化）

## 架构模式 (MVVM)

```
用户操作 → View → ViewModel → Service → 文件系统
                ↑
                └── State 更新 → View 刷新
```

- **ViewModels**: 全部 `@MainActor @Observable`
- **Services**: 纯逻辑层，无 UI 依赖
- **通信**: ViewModel → View 通过状态绑定；App 菜单 → ViewModel 通过 `Notification.Name`

## 关键设计决策

1. **自定义 HStack 三栏布局**（非 NavigationSplitView）— 支持拖拽阈值、单文件模式、圆角 Detail 区域
2. **`.windowStyle(.hiddenTitleBar)`** + 自定义 TitleBar — Buddy 风格布局
3. **NSViewRepresentable ResizeHandle** — SwiftUI `DragGesture` 在 macOS 上不可靠
4. **`@Observable` + 手动 UserDefaults** — `@AppStorage` 与 `@Observable` 不兼容
5. **Per-file undo** — 通过 ObjC runtime swizzling `NSWindow.undoManager`
6. **Notification 通信** — 10 个 `Notification.Name` 常量连接 App 菜单和 ViewModel
7. **SettingsModel.shared 单例** — 跨视图共享设置状态

## 编码规范

- **Swift 6 严格并发**：ViewModel 必须标注 `@MainActor`，注意 Sendable 合规性
- **命名**：ViewModel 用 `XxxViewModel`，Service 用 `XxxService`，Model 用名词
- **视图事件处理**：使用 ViewModifier 模式（参见 ContentView 中的各种 ViewModifier）
- **本地化**：所有面向用户的字符串必须通过 `L10n` 枚举，支持简中/繁中/英文
- **主题**：颜色必须通过 `ThemeColors` Environment 获取，不硬编码色值
- **文件操作**：统一通过 `FileService`，不直接调用 FileManager

## 测试

当前项目无测试目标。如需添加测试，在 Package.swift 中添加 `.testTarget`。

## CI/CD

GitHub Actions (`.github/workflows/release.yml`)：
- 触发：版本 tag (`v*`) 或手动 dispatch
- 流程：构建 → 组装 .app → ad-hoc 签名 → 创建 DMG → 发布 GitHub Release
- 发布前需确认 CHANGELOG.md 包含对应版本号

## 已知注意事项

- 同类型视图替换内容时 SwiftUI 可能不触发 `.onAppear`，需用 `.id(fileURL)` 强制重建
- Textual v0.x API 不稳定，锁定小版本号（渲染已迁移，Textual 仅过渡期保留）
- `SyntaxHighlightedEditor` 使用 ObjC runtime swizzling，修改需谨慎
- `.hiddenTitleBar` 模式下全屏时红绿灯行为需特殊处理（76px → 32px）
- 全局设置单例 `SettingsModel.shared` 不适合多窗口场景

## 文档

- `docs/architecture.md` — 详细架构文档
- `docs/design.md` — UI/UX 设计文档
- `docs/requirements.md` — 需求追踪（P0/P1/P2）
- `CHANGELOG.md` — 版本变更记录

---
> Source: [davidhoo/MarkdownReader](https://github.com/davidhoo/MarkdownReader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
