## translate-app-ios

> Flutter 日本旅遊導覽即時翻譯 App：語音轉文字（STT）+ Gemini 翻譯，斷網時備援 ML Kit 離線翻譯。主邏輯集中在 `lib/main.dart`（單檔較大，修改前先搜尋相關區塊）。

# translate_app — Cursor 專案脈絡（Windows 建立，Mac / iOS 接續開發）

## 專案是什麼

Flutter 日本旅遊導覽即時翻譯 App：語音轉文字（STT）+ Gemini 翻譯，斷網時備援 ML Kit 離線翻譯。主邏輯集中在 `lib/main.dart`（單檔較大，修改前先搜尋相關區塊）。

## 技術棧（pubspec）

- `google_generative_ai`：Gemini，`startChat` 維持上下文；Primary / Secondary API Key，**備援只換 Key、模型與 Primary 相同**。
- `speech_to_text`：**線上** STT；iOS 離線 STT 已改走 `OfflineAIService`（見下）。Android 離線仍用此套件 `onDevice`。
- `google_mlkit_translation`：**Android** 離線翻譯備援；**iOS 離線翻譯已改用 Qwen（`OfflineAIService`）**。有「離線語言包管理」頁仍保留給 Android。
- `connectivity_plus` + `InternetAddress.lookup`：自動模式判斷有無網路。
- `shared_preferences`：API Key、模型、歷史、每日用量、離線提示是否已顯示、`auto_return_primary` 等。
- `permission_handler`：麥克風 + **語音辨識**（iOS 需 `Permission.speech`）。
- `wakelock_plus`：收音時常亮。
- `share_plus`、`path_provider`：匯出 txt、分享。

## iOS 離線 AI 架構（2026/04 新增；Qwen 已改為 bundle 內建）

- **語音辨識**：`Whisper-small` Core ML，打包在 `ios/Runner/Models/openai_whisper-small/`（bundle，folder reference）。
- **翻譯**：`Qwen2.5-1.5B-Instruct Q4_K_M GGUF`，直接打包於 Runner bundle（`Runner/Models/Qwen2.5-1.5B-Instruct-Q4_K_M.gguf` 或 Runner 根目錄）。App 啟動時由 `LlamaService.prewarmIfReady()` 從 `Bundle.main` 載入，毋須下載。相容 lower-case 舊檔名與先前下載到 `Application Support/OfflineAI/` 的 fallback。
- **Flutter 橋接**：`lib/offline_ai_service.dart`（MethodChannel `com.translate_app/offline_ai`、EventChannel `/events`）。`ensureLlmDownloaded` 介面保留但實際只會嘗試從 bundle 再載入一次（無下載邏輯）。
- **原生實作**：`ios/Runner/OfflineAI/`（`OfflineAIPlugin.swift`、`WhisperService.swift`、`LlamaService.swift`、`OfflineAIError.swift`）。`ModelDownloader.swift` 保留檔案但已不再被 plugin 引用，僅為 Xcode 專案參照穩定保留。
- **SPM 依賴**（Runner.xcworkspace 的 Runner target）：
  - `WhisperKit` — https://github.com/argmaxinc/WhisperKit
  - `LLM.swift` — https://github.com/eastriverlee/LLM.swift
- **主流程分流**：`_listenNow` / `_translateOffline` 用 `_offlineAi.isSupported`（= `Platform.isIOS`）決定走 Whisper/Qwen 還是既有 ML Kit / `speech_to_text` 路徑。iOS 全程離線，`_useOnDeviceForSpeech()` 於 iOS 強制回傳 `true`。
- **bundle 缺檔提示**：若 Qwen GGUF 未加入 Runner target，`OfflineAiModelPage` 會顯示「未就緒 / 未內建」，翻譯時會丟出 `離線翻譯尚未就緒：Qwen 模型未內建於 app bundle…`。

## 行為摘要（實作時勿默默改掉）

- **翻譯方向**：`日/英 → 繁中` 與 `繁中 → 日/英`，STT 語言與 system prompt 隨模式切換。
- **語音辨識模式**：開 = 連網辨識；關 = 強制 onDevice；自動 = 有網連網、無網 onDevice；自動切換時會提示離線精準度可能下降。
- **翻譯**：先 Gemini（含 429 重試與 Secondary Key）；最終失敗且像網路錯誤 → ML Kit；設定裡有「自動切回 Primary」（預設開），會嘗試從 Secondary 切回 Primary 並有冷卻避免狂試。
- **用量**：本地每日計數、日本時間 16:00（UTC 07:00）重置邏輯已寫在 `main.dart`。
- **歷史**：離線暫存、搜尋、分組、匯出、長按複製。

## iOS（Mac 上必查）

- `ios/Runner/Info.plist` 必須含：
  - `NSMicrophoneUsageDescription`
  - `NSSpeechRecognitionUsageDescription`（線上模式仍使用 Apple Speech Framework）
- 程式已請求 `Permission.microphone` 與 `Permission.speech`；缺 plist 可能於首次使用時異常。
- 若 `onDevice` 不支援，已有錯誤字串偵測與降級提示。
- **WhisperKit / LLM.swift 需透過 SPM 加到 Runner target**（不是 Pod）。建置前請務必在 Xcode 中完成：File → Add Packages…
- **模型檔**：先依 `ios/Runner/OfflineAI/README.md` 放 `Models/openai_whisper-small/` 並加入 Runner target（folder reference），否則 App 啟動後 Whisper 不會就緒。

## Android

- `android/app/src/main/AndroidManifest.xml`：INTERNET、RECORD_AUDIO、WAKE_LOCK；既有 `<queries>` 保留。

## 本機開發注意

- **Windows**：Flutter plugin 建置需 **開發人員模式**（symlink）；磁碟空間不足時 `.dart_tool` 寫入會失敗。
- **多裝置**：`flutter run` 需 `-d <deviceId>`。
- **模擬器**：若 `flutter emulators` 為空，需先建立 Android AVD 或用實機。

## 程式風格

- 以現有 `main.dart` 風格為準；非必要不拆檔、不大範圍重構；新功能優先沿用既有 helper 與命名。

## Git commit（使用者偏好）

**不要**自動 commit。僅在使用者明確說要 commit（或「幫我提交」等）時，再執行 `git add` / `git commit`，並撰寫清楚的中文提交說明。

---
> Source: [avogau-spec/translate_app_ios](https://github.com/avogau-spec/translate_app_ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
