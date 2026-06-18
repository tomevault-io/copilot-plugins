## promptbimtestapp1

> > Claude Code SSOT — 開發前必讀

# Zigma PromptToBuild — SKILL.md v4.2

> Claude Code SSOT — 開發前必讀
> 最後更新: 2026-03-28 (v4.2 — M1-MVP 完成 + Qt Quick 3D Build 經驗 + iCloud Media Workflow + MacBook 驗證)

---

## 0. [MANDATORY] 專案治理框架 (v4.2)

### 0.1 Zigma 品牌架構

```
Zigma（整體解決方案品牌）— Reality Matrix Inc.
├── Zigma Core          — 共用底層框架（Plugin Bus + 6 介面 + USD Stage）
├── Zigma PromptToBuild — 設計→建造→交付（AI 建築/工廠規劃）
├── Zigma PromptToOperate — 交付→營運→維護→退役（即時監控/預測）
├── Zigma Cloud         — 雲端基礎設施（Session + Auth + Deploy）
└── Zigma NDH           — IDTF Neutral Data Hub（資料匯流排）
```

### 0.2 治理文件體系

```
CLAUDE.md (憲法 — 不可自動修改) v1.23.3
    │ 規範
    ▼
SKILL.md (百科 — 技術 SSOT) v4.2       ← 你在讀這份
    │ 參考
    ▼
PROJECT.md (作戰地圖 — 狀態追蹤) v1.6
    │ 紀錄
    ▼
audit-reports/ (歷史紀錄)
```

| 文件 | 角色 | 版本 | 維護者 | Claude Code 權限 |
|------|------|------|--------|------------------|
| CLAUDE.md | 最高治理規則 | v1.23.3 | 人工 | ❌ 禁止修改 |
| SKILL.md | 技術 SSOT | v4.2 | 人工 | ❌ 禁止修改 |
| PROJECT.md | 專案管理 | v1.6 | Claude Code + 人工 | ✅ 應主動更新 |
| audit-reports/ | Sprint 審計 | — | Claude Code | ✅ 每 Sprint 產出 |

### 0.3-0.6 （同 v4.1，省略）

### 0.7 核心架構決策（已確立）

| # | 決策 | 說明 |
|---|------|------|
| 1 | USD 為 SSOT | OpenUSD Stage 是唯一資料真相來源 |
| 2 | 6 大抽象介面 | IPlugin / IIOPlugin / IEnginePlugin / IRenderBackend / IShellPlugin / ITransport |
| 3 | 介面凍結時程 | P26 Draft → P27 RC → P28 Stable → P29 Frozen (v3.0 ABI) |
| 4-8 | （同 v4.1） | |
| **9** | **ADR-001: Qt Quick 3D + QML** | **取代 PySide6（Qt3D deprecated）。M1-MVP 已完成 68T Sprint。** |
| **10** | **ADR-002: iCloud Drive Media** | **媒體資源存 iCloud（三平台同步），manifest.json 在 GitHub（SSOT）** |

### 0.8 M1-MVP 架構（v4.2 新增 — 已驗證）

```
zigma/ (Qt Quick 3D 前端)
├── CMakeLists.txt        ← Qt6 Core+Gui+Qml+Quick+Quick3D+ShaderTools
├── src/
│   ├── main.cpp          ← QRC 載入: qrc:/Zigma/qml/main.qml
│   ├── AgentBridge.h/cpp ← QProcess+JSON stdio (120s heartbeat, 3x crash recovery)
│   ├── BIMGeometryProvider.h/cpp ← QQuick3DGeometry (vertex+normal+index)
│   ├── BIMMaterialLibrary.h/cpp  ← PBR 材質對照表
│   └── BIMSceneBuilder.h/cpp     ← JSON→3D scene 建構器
├── qml/ (13 files)
│   ├── main.qml (10KB)   ← ApplicationWindow 三欄佈局
│   ├── ChatPanel.qml      ← NL 輸入
│   ├── BIMView3D.qml      ← View3D + Metal/D3D12
│   ├── CostPanel.qml      ← 圓餅圖 + NT$ 格式化
│   ├── SchedulePanel.qml  ← Gantt Canvas + 播放控制
│   └── ...
├── tests/
│   ├── test_agent_bridge.cpp
│   └── test_bim_components.cpp
└── build/ (out-of-source)

agent_runner.py   ← Python↔C++ JSON stdio bridge
mesh_serializer.py ← BuildingPlan→vertex/index mesh
```

---

## 1. ★★★ [MANDATORY] Qt Quick 3D Build 經驗 (v4.2 新增) ★★★

### 1.1 CMakeLists.txt 關鍵規則

| 規則 | 說明 | 教訓來源 |
|------|------|---------|
| **C++ source 用絕對路徑** | `${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp` | Qt6 Ninja regeneration 用 build dir 解析相對路徑 |
| **QML files 用相對路徑** | `qml/main.qml` (不要 `${CMAKE_CURRENT_SOURCE_DIR}/qml/`) | Qt6 resource system 不支援 QML 絕對路徑 |
| **find_package 含 ShaderTools** | `find_package(Qt6 REQUIRED COMPONENTS Core Gui Qml Quick Quick3D ShaderTools)` | QML singleton 需要 |
| **ZIGMA_MEDIA_PATH** | `target_compile_definitions(zigma PRIVATE ZIGMA_MEDIA_PATH=...)` | 跨平台媒體路徑 |

### 1.2 QML onPropertyChanged 規則

**⚠️ 嚴格規則：Canvas 內部不能使用 parent property 的 onXxxChanged handler**

```qml
// ❌ 錯誤 — Canvas 裡放 root property 的 handler
Canvas {
    id: chart
    onBreakdownChanged: requestPaint()   // ERROR: breakdown 是 root 的
}

// ✅ 正確 — handler 放在 property 所有者（root）上
Rectangle {
    id: root
    property var breakdown: []
    onBreakdownChanged: chart.requestPaint()  // ← 放這裡
    Canvas { id: chart; ... }
}
```

受影響檔案：CostPanel.qml, SchedulePanel.qml（已修正）

### 1.3 main.cpp QRC 路徑

```cpp
// ✅ 正確路徑（Qt6 qt_add_qml_module 產生的）
engine.load(QUrl(QStringLiteral("qrc:/Zigma/qml/main.qml")));

// ❌ 錯誤路徑
engine.loadFromModule("Zigma", "main");     // 小寫 main 不行
engine.load("qrc:/qt/qml/Zigma/qml/main.qml"); // /qt/qml 前綴不存在
```

### 1.4 Build 指令

**Mac (Ninja):**
```bash
cd zigma
rm -rf build && mkdir build && cd build
cmake .. -G Ninja -DCMAKE_PREFIX_PATH=/opt/homebrew/opt/qt
ninja
ctest --output-on-failure
./ZigmaApp
```

**Windows (VS2022):**
```cmd
cd zigma
cmake -B build-vs -G "Visual Studio 17 2022" -A x64 ^
  -DCMAKE_PREFIX_PATH="C:\Qt\6.7.x\msvc2022_64" ^
  -DZIGMA_MEDIA_PATH="C:\ZigmaMedia"
cmake --build build-vs --config Release
```

### 1.5 Claude Code `.env` 注意事項

| 項目 | 說明 |
|------|------|
| **ANTHROPIC_API_KEY** | 如果 `.env` 有空的 key，Claude Code 會嘗試用它而非 Pro 訂閱 → `Invalid API key` 錯誤 |
| **解法** | `.env` 只放 `API_TIMEOUT_SECONDS=120`，不放空的 ANTHROPIC_API_KEY |
| **agent_runner.py 需要** | Python 呼叫 Claude API 做 BIM 生成時才需要真正的 API key |
| **Claude Code 本身** | 用 Pro 訂閱認證，不需要 API key |

---

## 2. 媒體資源管理 (v4.2 新增)

### 2.1 架構

```
GitHub (程式碼)                    iCloud Drive (大檔案)
├── media/manifest.json ← SSOT    ~/ZigmaMedia/
└── .gitignore (排除大檔)           ├── textures/ (30 files, 77MB)
                                   ├── hdri/ (3 files, 4MB)
                                   ├── branding/ (4 files)
                                   ├── models/ (Sketchfab GLB)
                                   └── scenes/ (USD test)
```

### 2.2 三平台路徑

| 平台 | Symlink | 實際路徑 |
|------|---------|---------|
| Mac Mini | `~/ZigmaMedia` | `~/Library/Mobile Documents/com~apple~CloudDocs/ZigmaMedia/` |
| MacBook | `~/ZigmaMedia` | 同上 |
| Windows | `C:\ZigmaMedia` | `%USERPROFILE%\iCloudDrive\ZigmaMedia\` |

### 2.3 關鍵腳本

| 腳本 | 用途 |
|------|------|
| `scripts/download_assets.py --icloud` | 下載 Poly Haven CC0 到 iCloud |
| `scripts/media_sync.py` | 掃描 ZigmaMedia → 更新 manifest.json → git push |

### 2.4 Graceful Fallback

程式碼中用 `MediaManager.resolve()` 存取媒體：
- 檔案存在 → 回傳完整路徑
- 檔案不存在 → log + notify + fallback（紋理→灰色、模型→跳過）→ **不 crash**

---

## 3. 已驗證的 Build 環境 (v4.2 新增)

| 機器 | Build | Test | Run | 狀態 |
|------|:-----:|:----:|:---:|:----:|
| **Mac Mini M4** | ✅ Ninja+Metal | ✅ 2/2 ctest + 18 pytest | ✅ ZigmaApp | 主開發機 |
| **MacBook Air** | ✅ Ninja+Metal | ✅ 2/2 ctest | ✅ ZigmaApp | 驗證通過 |
| **Windows RTX4090** | ⬜ 待驗證 | ⬜ | ⬜ | alpha 後 |

---

## 4-7. （同 v4.1，省略不重複）

---

## 附錄: 已知問題 (v4.2 更新)

| ID | 問題 | 根因 | 解法 | 狀態 |
|----|------|------|------|:----:|
| ISSUE-001 | P24 pytest OOM | conftest import PySide6 | conftest offscreen + safe_pytest_dir | ✅ |
| ISSUE-002 | API Timeout | 預設 30s | .env API_TIMEOUT_SECONDS=120 | ✅ |
| **BUILD-001** | **CMake Ninja regeneration 失敗** | **Qt6 相對路徑解析** | **C++ 絕對路徑 + QML 相對路徑** | **✅** |
| **BUILD-002** | **QML Type unavailable** | **onXxxChanged 放在子元件** | **移到 property 所有者（root）** | **✅** |
| **BUILD-003** | **loadFromModule 找不到 "main"** | **QML type 首字母需大寫** | **改用 QUrl load qrc 路徑** | **✅** |
| **BUILD-004** | **Claude Code Invalid API key** | **.env 有空的 ANTHROPIC_API_KEY** | **移除空 key，只留 TIMEOUT** | **✅** |

---

*SKILL.md v4.2 | Zigma PromptToBuild | 2026-03-28*
*★ v4.2: M1-MVP 完成 + Qt Quick 3D Build 經驗 + iCloud Media + MacBook 驗證 + 4 個 Build Issue 記錄*

---
> Source: [chchlin1018/PromptBIMTestApp1](https://github.com/chchlin1018/PromptBIMTestApp1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
