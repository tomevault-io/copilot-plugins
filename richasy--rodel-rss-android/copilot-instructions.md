## rodel-rss-android

> Shell 环境是 bash，但 Gradle 必须通过 PowerShell 运行（Windows）。

# CLAUDE.md

## Build

Shell 环境是 bash，但 Gradle 必须通过 PowerShell 运行（Windows）。
从项目根目录执行：

```bash
powershell.exe -Command ".\gradlew.bat :app:assembleDebug 2>&1"
```

## UI 规范

### 溢出菜单

所有溢出菜单（MoreVert 触发）必须使用 Material3 Expressive 的 `DropdownMenuPopup` + `DropdownMenuGroup` 组合，不要使用旧的 `DropdownMenu`。每个菜单项使用带 `shape` 参数的 `DropdownMenuItem`，并配上 `leadingIcon`。参考 `ArticleListScreen.kt` 中 AppBar 的实现。

### 长按菜单

所有长按菜单使用 `ModalBottomSheet`，每个操作项为一行，文本在左，图标在右（`trailingIcon` 位置）。参考 `ArticleListScreen.kt` 中 `ArticleActionSheet` 的实现。

---
> Source: [Richasy/Rodel.Rss.Android](https://github.com/Richasy/Rodel.Rss.Android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
