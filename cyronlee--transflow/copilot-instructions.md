## transflow

> - 使用 `apple-docs-mcp` 搜索 macOS 开发文档

# TransFlow 项目指令

## 开发文档
- 使用 `apple-docs-mcp` 搜索 macOS 开发文档
- 修改语音转写相关文件前，先读 `docs/speech-model-lifecycle.md`
- Fluid Audio 本地文档位于 `build/DerivedData/SourcePackages/checkouts/FluidAudio/Documentation/`

## 质量检查
- 确保 xcode build 成功

## PKG 发布
- 仅使用 `./scripts/build-pkg.sh` 生成安装包，发布流程统一为 PKG-only
- 发布 GitHub Release 时只上传经过 Developer ID 签名、notarization 和 stapling 的 PKG
- 默认使用 `TransFlowNotary` notarytool profile；如本机证书名变化，显式传入 `--app-sign` 和 `--installer-sign`

## 国际化（i18n）
- 所有用户可见字符串使用 `Localizable.xcstrings` String Catalog，key 格式 `模块.描述`（如 `settings.language`）
- 支持语言：`en`、`zh-Hans`；新增字符串须同时提供两种翻译
- SwiftUI 中用 `Text("key")` / `LocalizedStringKey`；纯 `String` 上下文用 `String(localized: "key")`

## 外观模式
- 通过 `AppSettings.shared.appAppearance` 控制，支持跟随系统 / 浅色 / 深色
- 在 `TransFlowApp` 根视图使用 `.preferredColorScheme(settings.appAppearance.colorScheme)` 应用

---
> Source: [Cyronlee/TransFlow](https://github.com/Cyronlee/TransFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
