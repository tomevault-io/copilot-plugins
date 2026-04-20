## valdi

> This is an open source project. Never commit secrets, API keys, or proprietary information.

# Valdi Open Source - Cursor Rules

## ⚠️ Open Source Project

This is an open source project. Never commit secrets, API keys, or proprietary information.

## 🚨 CRITICAL: This is NOT React!

Valdi uses TSX/JSX syntax but is **fundamentally different from React**. 

**Common AI mistakes:**
- ❌ Suggesting `useState`, `useEffect`, `useContext` (don't exist!)
- ❌ Functional components (don't exist!)
- ❌ `this.props` (should be `this.viewModel`)
- ❌ `markNeedsRender()`, `onMount()`, `onUpdate()` (wrong names/don't exist!)

**Correct Valdi:**
- ✅ `class MyComponent extends StatefulComponent`
- ✅ `state = {}` + `this.setState()`
- ✅ `this.viewModel` for props
- ✅ `onCreate()`, `onViewModelUpdate()`, `onDestroy()` lifecycle

## 📁 Context-Specific Rules

Cursor automatically loads additional rules based on where you're working:

| Working In | See |
|-----------|-----|
| TypeScript/TSX components | `.cursor/rules/typescript-tsx.md` |
| Swift compiler | `.cursor/rules/compiler.md` |
| C++ runtime | `.cursor/rules/cpp-runtime.md` |
| Android (Kotlin) | `.cursor/rules/android.md` |
| iOS (Objective-C) | `.cursor/rules/ios.md` |
| Bazel files | `.cursor/rules/bazel.md` |
| Tests | `.cursor/rules/testing.md` |

**→ Check `.cursor/rules/README.md` for the full list**

## Quick Commands

```bash
bazel build //...          # Build everything
bazel test //...           # Run all tests
valdi install ios          # Build & install iOS app
valdi hotreload            # Start hot reload
```

## More Information

- **Comprehensive guide**: `/AGENTS.md` (621 lines)
- **AI tooling**: `/docs/docs/ai-tooling.md`
- **Support**: `/SUPPORT.md`
- **Discord**: https://discord.gg/uJyNEeYX2U

---
> Source: [Snapchat/Valdi](https://github.com/Snapchat/Valdi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
