## cs193p-swift

> `MemorizeApp.swift` 定义 SwiftUI `@main` 入口，每次迭代都从这里挂载 `ContentView`。`ContentView.swift` 是当前 UI 层，包含基本的卡片网格，可作为拆分子视图与预览的起点。`EmojiMemoryGame.swift` 充当 ViewModel，负责把 UI 与泛型 `MemoryGame` 模型串联；在引入主题或难度时应扩展此层。`MemoryGame.swift` 保存核心配对逻辑，目前 Card 工厂使用占位符，需要在实现时确保传入唯一的表情或资产。图像与配色集中在 `Assets.xcassets`，添加资源前请遵循现有 Catalog 结构。

# Repository Guidelines

## Project Structure & Module Organization
`MemorizeApp.swift` 定义 SwiftUI `@main` 入口，每次迭代都从这里挂载 `ContentView`。`ContentView.swift` 是当前 UI 层，包含基本的卡片网格，可作为拆分子视图与预览的起点。`EmojiMemoryGame.swift` 充当 ViewModel，负责把 UI 与泛型 `MemoryGame` 模型串联；在引入主题或难度时应扩展此层。`MemoryGame.swift` 保存核心配对逻辑，目前 Card 工厂使用占位符，需要在实现时确保传入唯一的表情或资产。图像与配色集中在 `Assets.xcassets`，添加资源前请遵循现有 Catalog 结构。

## Build, Test, and Development Commands
首选通过 `open Memorize.xcodeproj` 或 Xcode 的 “Open Quickly” 打开工程。命令行构建：`xcodebuild -scheme Memorize -destination 'platform=iOS Simulator,name=iPhone 15' build` 会编译并在默认 DerivedData 下生成产物。运行并附带日志可使用 `xcodebuild -scheme Memorize -destination 'platform=iOS Simulator,name=iPhone 15' test`，在首次执行前请先手动在 Xcode 里选择同名模拟器以避免设备解析失败。开发期间启用 Canvas 预览时务必保持 View 结构体 `body` 无副作用。

## Coding Style & Naming Conventions
遵循 Swift API Design Guidelines：类型、枚举 case 采用 UpperCamelCase，变量与函数使用 lowerCamelCase，布尔函数以前缀 `is`、`has`、`should`。保持 4 空格缩进，使用 `private(set)` 暴露只读状态。View 层尽量保持无逻辑，交互行为（例如翻牌）通过 ViewModel 方法触发。命名资源时沿用 `emoji_theme_<name>` 或 `color_<intent>` 前缀方便在 `Assets.xcassets` 中检索。

## Testing Guidelines
当前尚未创建测试 Target；请在添加 `MemorizeTests` 目录后立即在 Xcode 中勾选 “Include Tests”。测试框架建议使用 XCTest，文件命名规则 `SomethingTests.swift`，单测方法以 `test_shouldDoX_whenCondition` 形式描述行为。命令行执行使用 `xcodebuild -scheme Memorize -destination 'platform=iOS Simulator,name=iPhone 15' test`，并在 PR 描述中粘贴末尾的 `Test Suite 'All tests' passed` 摘要。若实现配对逻辑或计分规则，新增测试需覆盖匹配成功、失败和重置三类情境。

## Commit & Pull Request Guidelines
历史提交沿用简短的阶段说明（例如 `lecture3 finished`）；保持类似格式或改用 `topic: summary`（如 `model: finalize card factory`），一句概括即可。确保每个 PR 包含：变更概要、受影响视图截图或模拟器录屏、手动与自动测试结果以及关联的 issue/课程章节链接。当 PR 涉及模型或 ViewModel 时，在描述中写清现有状态与预期状态，以便评审快速验证。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ryusaksun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ryusaksun)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
