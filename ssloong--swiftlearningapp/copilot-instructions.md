## swiftlearningapp

> - Source: `SwiftLearningApp/` — SwiftUI views/components（源码：SwiftUI 视图与组件）。

# Repository Guidelines（仓库指南）

## Project Structure & Modules（项目结构与模块）
- Source: `SwiftLearningApp/` — SwiftUI views/components（源码：SwiftUI 视图与组件）。
- Components: `SwiftLearningApp/Components/Day1..DayN/` — migrated interactive cards（按日拆分的交互组件，逐步迁移中）。
- Tests: `SwiftLearningAppTests/`, `SwiftLearningAppUITests/` — unit and UI（测试：单元与 UI）。
- Assets: `SwiftLearningApp/Assets.xcassets/` — icons/colors（资源：图标与颜色）。
- Entrypoint: `SwiftLearningApp.swift`; main shell: `ContentView.swift`（入口与主界面）。
- Learning modules: `DayN*LearningView.swift`; shared UI in `SharedComponents.swift`, `InteractiveComponents.swift`（学习模块与共享 UI）。

## Build, Test, and Development（构建、测试与开发）
- Open in Xcode: `open SwiftLearningApp.xcodeproj`（在 Xcode 中打开开发）。
- Build (CLI): `xcodebuild -project SwiftLearningApp.xcodeproj -scheme SwiftLearningApp build`（命令行构建）。
- Test: `xcodebuild test -project SwiftLearningApp.xcodeproj -scheme SwiftLearningApp -destination 'platform=iOS Simulator,name=iPhone 15'`（运行单元与 UI 测试）。
- Targets: macOS 13+, Xcode 15+, iOS 16+（最低环境要求）。

## Coding Style & Naming Conventions（代码风格与命名）
- Indent 4 spaces; ~120 cols; brace on same line（4 空格缩进，约 120 列，左花括号同行）。
- Names: PascalCase types/views; lowerCamelCase vars/functions; file names `...Card.swift`（类型/视图用帕斯卡命名，变量/函数小驼峰，文件以 ...Card.swift 结尾的组件）。
- One main type per file; use `// MARK:` to organize（每文件一个主要类型，用 MARK 分组）。
- Format with Xcode before commit（提交前用 Xcode 格式化）。

## Testing Guidelines（测试规范）
- Frameworks: XCTest, XCUITest（测试框架：XCTest 与 XCUITest）。
- Mirror source names, e.g., `Day6LearningViewTests`（测试文件名与源文件对应）。
- Cover new logic and non-trivial views; add UI assertions/snapshots（覆盖新增逻辑与重要视图，添加 UI 断言/快照）。
- Run locally; keep CI green before PR（本地通过并保持 CI 通过后再提 PR）。

## Commit & Pull Requests（提交与合并请求）
- Conventional Commits + emoji as in history（遵循含表情的约定式提交）。
  e.g., `✨ feat: 完成Day 6继承与多态学习功能实现`, `📚 docs: 更新README`。
- Keep changes scoped; link issues（变更聚焦并关联 Issue）。
- PR must include description, screenshots/GIFs, test results, and doc updates when needed（PR 需含描述、截图/GIF、测试结果及必要文档更新）。

## Security & Configuration Tips（安全与配置）
- Never commit secrets; keep assets in `Assets.xcassets/`（不要提交密钥；资源放入资产目录）。
- Discuss third‑party deps; see `REFACTORING_PLAN.md`（第三方依赖需讨论，参考重构计划）。
- Do not change bundle IDs/targets without approval（未经批准不要修改 Bundle ID 与目标版本）。

## Component Migration（组件迁移）
- Place day-specific cards in `SwiftLearningApp/Components/DayN/`；shared in `Components/Shared/`（按日与共享目录归档）。
- Keep type/file names stable with `...Card.swift`；one main type per file（类型名与文件名一致，Card 后缀）。
- After moving, ensure Xcode Target Membership is checked（迁移后勾选目标成员，保证编译）。

## Component Protocol（组件协议）
- Protocol: `InteractiveComponent` at `Components/Shared/InteractiveComponent.swift`（统一元数据与构建）。
- Fields: `id` (default type name), `title`, `summary`, `makeView() -> AnyView`（标识、标题、摘要、构建）。
- Use `ComponentDescriptor` for lists/registry（用描述体统一注册与展示）。

---
> Source: [SSLoong/SwiftLearningApp](https://github.com/SSLoong/SwiftLearningApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
