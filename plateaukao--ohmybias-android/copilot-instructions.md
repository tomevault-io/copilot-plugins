## ohmybias-android

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

OhMyBias 米 Android — Android 嘸蝦米（Boshiamy）鍵盤，**自 `~/src/ohmybias-ios` 移植**（引擎層一對一對應
iOS `Shared/`）。純 Kotlin、零第三方執行期依賴（僅 Android 平台 API；org.json 為 test-only 依賴）。
極簡版：保留**基本聯想詞**（萌典詞組 `phrases.bin`，CC0）＋使用者自訂詞，其餘語料 API 保留簽名、回傳空結果。
文件、commit、註解、UI 皆用繁體中文。

與家族的關係：`~/src/yabomish` = macOS 完整版（上游）；`~/src/ohmybias` = macOS 極簡版；
`~/src/ohmybias-ios` = iOS 極簡版（**本專案的直接移植來源**）。從上游移植修正時：
引擎層以 ohmybias-ios `Shared/` 為準（其又以 yabomish Shared/ 為準）。

## Build

```bash
./gradlew assembleDebug        # app/build/outputs/apk/debug/app-debug.apk
./gradlew testDebugUnitTest    # JVM 引擎測試（無模擬器）
```

Gradle 8.13（wrapper）+ AGP 8.10 + Kotlin 2.1；compileSdk 36 / targetSdk 36 / minSdk 28（Android 9）。
單一 module `app`。

Google Play 上架（同 einkbro/calliplus 做法）：`playRelease` build type =
`info.plateaukao.ohmybias.g`，與 GitHub 版（`release`，自家 keystore）並存。
Upload key／publisher 憑證讀 `~/.secrets/ohmybias-keystore.properties`（不進 repo）。
`./gradlew publishPlayReleaseBundle` 上傳 AAB（預設 internal 軌；首次上架草稿要
`--release-status draft`；正式發佈 `--track production`）。Listing 文案在
`app/src/main/play/`（Gradle Play Publisher 版面）。

模擬器驗證（AVD `Pixel_7_API_34`）：
```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
adb shell ime enable info.plateaukao.ohmybias/.keyboard.OhMyBiasImeService
adb shell ime set    info.plateaukao.ohmybias/.keyboard.OhMyBiasImeService
# 測試字表（debug build）：
adb push test.cin /data/local/tmp/liu.cin
adb shell "run-as info.plateaukao.ohmybias sh -c 'cp /data/local/tmp/liu.cin files/shared/liu.cin'"
adb shell am force-stop info.plateaukao.ohmybias   # 重啟讓 IME 重載表
adb shell ime set info.plateaukao.ohmybias/.keyboard.OhMyBiasImeService  # force-stop 後系統會退回預設鍵盤
```

## Architecture

- `shared/`（`app/src/main/java/.../shared/`）：**禁止 import Android API**（org.json 除外 — Android
  runtime 內建、JVM 測試由 test 依賴補齊）。與 iOS `Shared/` 一對一：`InputEngine`（核心狀態機）、
  `CINTable`/`CINCompiler`（CINM 二進位，格式與 iOS 完全相同可跨平台共用）、`CandidateRanker`、
  `SuggestionEngine`、`WikiCorpus`（PHM2 phrases.bin，與 iOS 共用同一份；`tools/convert_phrases_v2.py` 自舊 PHMM 轉製）、`ZhuyinLookup`（zhuyin/pinyin/char_freq mmap 二進位 ZYM2/PYM2/CFM2，`tools/gen_data_bins.py` 自 iOS 版 JSON 轉製；格式見該腳本，讀取端 `DataMaps.kt`）、`BigramSuggest`、`UserPhrases`、
  `SkinSettings`。平台抽象：`AppEnv`（sharedDir 路徑）、`BinData`（mmap LE 讀取）、
  `FreqTracker` interface（排序邏輯在 default method）、`ClipboardBridge`、`DefaultPreferences.backing`。
- `android/`：`OhMyBiasApp`（Application — 建目錄、assets→filesDir/shared 複製、掛橋接）、
  `Prefs`（SharedPreferences 實作 IMEPreferences）、`SqliteFreqTracker`（freq.db 三表同 iOS）、
  `ClipboardProcessor`（ICU Transliterator 簡繁轉換）。
- `keyboard/`：`OhMyBiasImeService`（InputMethodService = InputEngineDelegate）、`KeyboardView`
  （字母/數字/符號/注音/九宮格五頁，自訂 ViewGroup 手動排版）、`KeyButton`（自繪+觸控：點按/上下滑/長按/
  連刪/空白鍵游標拖曳）、`CandidateBar`（候選列+工具列）、`CollectionPanelView`（符號/emoji/顏文字/常用語面板）、
  `CollectionData`（由 iOS 版機械轉換）、`LongPressData`（android.icu 曆法日期）、
  `HardwareKeyHandler`（實體鍵盤 KeyEvent → 引擎，語意同 macOS 版 YabomishInputController）、
  `FloatingCandidateView`（實體鍵盤「游標旁浮動」模式的候選氣泡；位置來自 CursorAnchorInfo）。
  實體鍵盤三種畫面模式（`Prefs.hardKeyboardMode`：keypad／floating／bar）：浮動與底列把 IME 根視圖
  撐滿整個視窗、以 `onComputeInsets` 回報內容高度與可觸區（其餘觸控穿透到 app）。
- `MainActivity`：設定頁（啟用鍵盤/匯入 liu.cin(SAF)/皮膚/偏好/自訂詞/測試輸入框）。

單一 APK（IME + 設定同 process）→ 不需 App Group，資料統一在 `filesDir/shared/`。
Application ID：`info.plateaukao.ohmybias`。

## Tests

`./gradlew testDebugUnitTest` — 引擎層在 JVM 上測（移植自 iOS `Tests/main.swift`，JUnit 形式）。
`TestEnv` 單例把 `AppEnv.sharedDir` 指到 temp dir 並複製 assets 資料檔 — 測試類別開頭 `TestEnv.touch()`。
`MockEngineDelegate` 記錄所有 delegate callback，是測引擎的標準做法（同 iOS）。

## 注意事項

- **liu.cin 有版權（行易），只能使用者自行匯入、on-device 編譯，絕不預編/隨附 liu.bin。**
- `phrases.bin` 的 key 是詞首單字；萌典用「臺」不用「台」（打「台」不會出聯想，打「臺」才會）。
- `KeyboardView`/根視圖 `clipChildren=false`（長按氣泡要凸出鍵外）— 因此子視圖 onDraw **不可用
  `canvas.drawColor`**（剪裁範圍超出邊界會蓋掉候選列），一律畫有界矩形。
- IME 對 EditText 的游標操作走 `InputConnection`：`commitText`／`deleteSurroundingTextInCodePoints`／
  `setSelection`（`engineDidCommitPair` 依此把游標放進成對標點中間）。
- Android IME 無 iOS 鍵盤 extension 的 60MB 上限 — `MemoryBudget.canAfford` 恆真（保留 API 對齊上游）。
- **minSdk 28（Android 9）**：主題不能用 `Theme.DeviceDefault.DayNight`（API 29）— 走
  `values/`＋`values-night/` 的 `@style/AppTheme`。`android.icu.text.Transliterator` 公開
  SDK 自 API 29 起（runtime 其實 API 24 就在、Android 9 @hide 可用，但不能假設 OEM ROM 放行）—
  `ClipboardProcessor` 以 `SDK_INT >= 29` 守衛，28 退回 s2t/t2s.json 逐字對照。
  驗證用不了 sim-use（其 device bridge APK 是 minSdk 30），改用 adb + uiautomator dump。

---
> Source: [plateaukao/ohmybias-android](https://github.com/plateaukao/ohmybias-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
