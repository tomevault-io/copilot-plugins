## player-glass-sheet-dialog

> Player responsive single-choice dialogs — bottom portrait chrome and landscape settings-panel chrome


# 播放器响应式单列选择弹窗

完整约定见：**[docs/player-glass-sheet-dialog.md](docs/player-glass-sheet-dialog.md)**。

编写或修改播放器内长宽比、倍速、音轨、字幕等单列选择弹窗时遵守：

1. **统一入口**：使用 `PlayerGlassSheetDialog.showSingleChoice(...)`，不要回退 `MaterialAlertDialog.setItems`、系统 `RadioButton` 或 Activity 内私有 inflate helper。
2. **Chrome 分流**：播放页快捷项传 `chrome = quickChoiceChrome()` 和 `playerPrefs = playerPrefs`；竖屏应为 `PLAYER_BOTTOM`，横屏应为 `PLAYER_SETTINGS_PANEL`。
3. **横屏样式**：`PLAYER_SETTINGS_PANEL` 必须复用 `PlayerSettingsSheetChrome.applyWindowLayout`、`applyBackdrop`、`applyPanelOpacity`，不要单独硬编码宽高或透明度。
4. **竖屏样式**：`PLAYER_BOTTOM` 使用 `dialog_player_quick_bottom_sheet.xml` + `item_player_quick_bottom_sheet_row.xml`，从底部滑出，行为与选集样式一致。
5. **控制层**：快捷弹窗打开前隐藏播放器 controls；弹窗可见时不要露出播放/暂停/进度条等底部控制。
6. **方向策略**：`quickChoiceChrome()` 走 `PlayerConfigurationOrientationPolicy.isLandscape(...)`，不要在 Activity 里直接比较 `Configuration.ORIENTATION_LANDSCAPE`。

---
> Source: [Xunzi229/openvideo](https://github.com/Xunzi229/openvideo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
