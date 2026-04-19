## idle-hippo

> 以 Flutter + Flame 交付一款放置＋節奏小遊戲的手遊。


以 Flutter + Flame 交付一款放置＋節奏小遊戲的手遊。
風格：zh-TW、工程導向、可直接複製到專案根目錄。

## 1. 檔案與目錄規範
- **lib/core/**：共用邏輯（router、assets 常數、事件 bus）
- **lib/game/**：遊戲主邏輯（HippoGame、components、systems）
- **lib/ui/overlays/**：Flutter UI Overlay（主選單、暫停選單、設定）
- **lib/services/**：儲存、音效、網路等服務
- **assets/images/**、**assets/audio/**、**assets/fonts/**：遊戲資產  
- **test/**：單元與整合測試，檔案結構需對應 lib

## 2. 任務卡規則
- 每張卡只對應「一個階段目標」或「一個子功能」  
- 任務卡內容必須包含：
  - **上下文**：對應的 spec 或設計描述  
  - **目標**：此階段要達成的核心功能  
  - **驗收標準**：1, 2, 3 條件化檢核點  
  - **限制**：框架版本、必須用的套件或 API  

## 3. AI 產出流程
1. **Diff-first 原則**  
   - 請 AI 先列出「將修改哪些檔案、意圖是什麼」  
   - 確認後才允許生成完整程式碼  
2. **檔案分批生成**  
   - 大功能需拆多張卡，每張卡控制在 2–3 檔案  
3. **測試同步產出**  
   - 功能程式碼產生時，同步要求 AI 補上最小可行測試（`flutter_test`）  

## 4. 驗收規則
- **人工測試**：逐一對照任務卡上的驗收標準  
- **自動測試**：跑 `flutter test`，必須綠燈才可合併  
- **Lint**：必須通過 `flutter analyze` 無錯誤  
- **README 更新**：每完成一階段需補充使用說明與資產規範  
- test 案例的描述都使用繁體中文

## 5. Pin 與上下文
- **必 Pin 檔案**：`main.dart`、`hippo_game.dart`、`router.dart`、`pubspec.yaml`、 `docs/config.md`、`lib/models/game_state.dart`
- **可 Pin 檔案**：當前開發中的 component/service  
- AI 回答必須檢查 pinned files，避免改錯或漏 context  

## 6. Branch & PR
- `main`：隨時可發佈  
- `develop`：日常整合  
- `feature/*`：對應單一任務卡  
- PR 必須附帶：
  - 改動檔案清單
  - 為什麼需要改
  - 驗收方式（手動/自動測試）  

## 7. 常見錯誤防呆
- **資產錯誤**：AI 必須檢查 `pubspec.yaml` 與 assets 資料夾是否一致  
- **座標/解析度**：固定使用 `FixedResolutionViewport(Vector2(1080,1920))`  
- **Overlay 切換**：禁止直接呼叫 Navigator，需透過 `router.dart` 控制  
- **狀態存取**：禁止使用全域變數，統一透過 service 或 event bus  

## 8. CI/CD
- GitHub Actions 最小流程：
  - `flutter pub get`
  - `flutter analyze`
  - `flutter test`  
- 必須在 PR 時自動跑  

## 9. 文件規範
- **README.md**：安裝、啟動、資產規範、場景擴充方式  
- **docs/**：可額外存放 PM spec、遊戲設計圖
- 每次實作需求前都需要查看 assets/config 內參數，是否新需求有可以共用的，那就不需要再新增而外變數
- 每當有新增新增數於 assets/config 內，則需要更新 docs/config.md 文件

## 10. UI 上顯示
- 當有顯示上的實作時，都需要實作多國語系，並且更新所有語系包 assets/lang/*

## 11. flutter 語法
- withOpacity 都改用 withValues 方法

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VagrantPi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
