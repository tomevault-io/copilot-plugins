## openclaw-model-upgrade

> OpenClaw 模型升級與新模型支援工具。當需要升級 OpenClaw 版本以支援新模型（如 GPT-5.4）、從 GitHub 主分支安裝未發布版本、或排查模型不可用問題時使用此技能。涵蓋版本升級、從源碼構建、forward-compat 合成模型機制、API Key 配置等完整流程。觸發關鍵字：upgrade、模型升級、gpt-5.4、新模型、model support、forward-compat、from source、源碼安裝、npm install、pnpm build。


# OpenClaw Model Upgrade - 模型升級與新模型支援完整手冊

> 版本：1.0.0 | 最後更新：2026-03-07
> 基於 GPT-5.4 支援升級的實戰經驗整理。

---

## 一、適用場景

- 需要支援尚未包含在 npm 發行版中的新模型（如 PR 已合併但未發布）
- 從 npm 發行版升級到 GitHub 主分支最新代碼
- 排查 `models list` 中看不到某個模型的問題
- 了解 OpenClaw 的 forward-compat 合成模型機制

---

## 二、升級決策樹

```
需要新模型支援
    │
    ├── 檢查 npm 最新版是否已包含 → `npm view openclaw version`
    │       │
    │       ├── 已包含 → `npm install -g openclaw@latest`（簡單安全）
    │       │
    │       └── 未包含 → 需要從 GitHub 主分支安裝
    │               │
    │               ├── 方案 A：克隆 → 構建 → 本地安裝（推薦）
    │               │
    │               └── 方案 B：`npm install -g github:openclaw/openclaw`
    │                          （極慢，容易超時，不推薦）
    │
    └── 檢查 PR 是否已合併 → `gh pr view <PR號> --repo openclaw/openclaw`
            │
            ├── 已合併 → 可以從主分支安裝
            │
            └── 未合併 → 需要手動 cherry-pick 或等待合併
```

---

## 三、踩坑記錄與解決方案

### 坑 1：`brew upgrade openclaw` 失敗

**症狀**：
```
Error: Cask 'openclaw' is not installed.
```

**原因**：OpenClaw 是通過 `npm install -g` 安裝的，不是 Homebrew cask。

**正確做法**：
```bash
# 確認安裝方式
npm list -g openclaw
# 升級
npm install -g openclaw@latest
```

**教訓**：升級前先確認安裝方式（npm vs brew vs 手動）。

---

### 坑 2：npm 最新版不包含最新 PR

**症狀**：升級到最新 npm 版本（如 2026.3.2）後，`grep gpt-5.4` 在安裝目錄中找不到任何結果。

**原因**：PR 已合併到 `main` 分支但尚未發布到 npm。npm 發行版和 GitHub HEAD 之間存在時間差。

**排查方法**：
```bash
# 檢查 CHANGELOG 是否包含目標 PR
grep "<PR號>" /opt/homebrew/lib/node_modules/openclaw/CHANGELOG.md

# 如果沒有，說明該 PR 不在當前版本中
```

**教訓**：不要假設「最新版 = 包含所有已合併 PR」。一定要驗證。

---

### 坑 3：`npm install -g github:openclaw/openclaw` 極慢/超時

**症狀**：命令掛起數分鐘無任何輸出，最終 `ECONNRESET` 或超時。

**原因**：npm 從 GitHub 直接安裝需要克隆完整倉庫 + 執行 prepare 腳本，對大型 monorepo 極不友好。

**正確做法（方案 A：本地構建）**：
```bash
# 1. 淺克隆（只取最新一層歷史）
git clone --depth 1 https://github.com/openclaw/openclaw.git /tmp/openclaw-build

# 2. 安裝 pnpm（如果沒有）
npm install -g pnpm@10.23.0

# 3. 安裝依賴
cd /tmp/openclaw-build && pnpm install

# 4. 構建
pnpm build

# 5. 全局安裝（從本地構建產物）
npm install -g .

# 6. 清理
rm -rf /tmp/openclaw-build
```

**教訓**：對於大型 TypeScript monorepo，本地克隆 → 構建 → 安裝是最可靠的方式。

---

### 坑 4：`corepack` 不可用

**症狀**：
```
command not found: corepack
```

**原因**：某些 Node.js 安裝方式（如 Homebrew）不附帶 corepack。

**正確做法**：
```bash
# 直接用 npm 安裝 pnpm
npm install -g pnpm@10.23.0
```

**教訓**：不要依賴 corepack，直接 `npm install -g pnpm` 更可靠。

---

### 坑 5：`models list --all` 看不到 forward-compat 合成模型

**症狀**：升級完成後，`openclaw models list --all` 沒有顯示 `openai/gpt-5.4`。

**原因**：OpenClaw 的 forward-compat 機制通過 `SYNTHETIC_CATALOG_FALLBACKS` 在**運行時**動態合成模型條目。`models list --all` 顯示的是 pi-ai 原始目錄，不包含合成條目。

**機制解說**：
```
pi-ai 目錄提供 openai/gpt-5.2（模板）
        │
        ▼
applySyntheticCatalogFallbacks() 在運行時：
  - 檢查 openai/gpt-5.4 是否已存在 → 不存在
  - 找到模板 openai/gpt-5.2 → 存在
  - 合成 openai/gpt-5.4 條目（複製 gpt-5.2 的屬性）
        │
        ▼
openai/gpt-5.4 可用於 models set / agent 調用
```

**驗證方法**：
```bash
# 不要看 models list，直接嘗試設為默認模型
openclaw models set openai/gpt-5.4

# 如果成功，說明 forward-compat 生效
openclaw models status
```

**教訓**：`models list` 不是最終判斷依據。合成模型只在運行時生成，直接 `models set` 才是驗證手段。

---

### 坑 6：升級後忘記重啟 Gateway

**症狀**：升級成功但新模型不生效。

**正確做法**：
```bash
# 重啟 gateway（使用 launchctl）
launchctl stop ai.openclaw.gateway
sleep 2
launchctl start ai.openclaw.gateway

# 或使用 openclaw 命令
openclaw daemon restart
```

**教訓**：任何版本升級後都必須重啟 gateway。

---

### 坑 7：升級後 OpenAI 認證缺失

**症狀**：`openclaw models status` 顯示：
```
Missing auth
- openai Run `openclaw configure` or set an API key env var.
```

**原因**：新設的模型（如 `openai/gpt-5.4`）需要 OpenAI API Key，但之前可能只配了其他 provider。

**正確做法**：
```bash
# 方法 1：通過配置嚮導
openclaw configure

# 方法 2：設置環境變數
export OPENAI_API_KEY="sk-..."

# 方法 3：通過 auth profiles
openclaw models auth add --provider openai --type api-key
```

**教訓**：切換模型 provider 後一定要檢查認證配置。

---

## 四、完整升級 SOP

以下是經過驗證的完整流程，可作為自動化腳本的基礎：

```bash
#!/bin/bash
set -euo pipefail

# === 配置 ===
OPENCLAW_REPO="https://github.com/openclaw/openclaw.git"
BUILD_DIR="/tmp/openclaw-build"
TARGET_MODEL=""  # 可選：升級後設為默認的模型

# === Step 1：記錄當前狀態 ===
echo "=== 當前版本 ==="
CURRENT_VERSION=$(openclaw --version)
echo "版本: $CURRENT_VERSION"

# === Step 2：備份配置 ===
echo "=== 備份配置 ==="
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.pre-upgrade.$(date +%Y%m%d%H%M%S)

# === Step 3：克隆並構建 ===
echo "=== 克隆倉庫 ==="
rm -rf "$BUILD_DIR"
git clone --depth 1 "$OPENCLAW_REPO" "$BUILD_DIR"

echo "=== 安裝依賴 ==="
command -v pnpm >/dev/null 2>&1 || npm install -g pnpm@10.23.0
cd "$BUILD_DIR"
pnpm install

echo "=== 構建 ==="
pnpm build

# === Step 4：安裝 ===
echo "=== 全局安裝 ==="
npm install -g .

# === Step 5：重啟 Gateway ===
echo "=== 重啟 Gateway ==="
launchctl stop ai.openclaw.gateway 2>/dev/null || true
sleep 2
launchctl start ai.openclaw.gateway
sleep 3

# === Step 6：驗證 ===
echo "=== 驗證 ==="
NEW_VERSION=$(openclaw --version)
echo "升級前: $CURRENT_VERSION"
echo "升級後: $NEW_VERSION"

if [ -n "$TARGET_MODEL" ]; then
    openclaw models set "$TARGET_MODEL"
    echo "默認模型已設為: $TARGET_MODEL"
fi

launchctl list | grep openclaw && echo "Gateway 運行正常" || echo "Gateway 異常！"

# === Step 7：清理 ===
echo "=== 清理構建目錄 ==="
rm -rf "$BUILD_DIR"

echo "=== 升級完成 ==="
```

---

## 五、Forward-Compat 合成模型機制詳解

### 源碼位置

- 目錄合成：`src/agents/model-catalog.ts` → `applySyntheticCatalogFallbacks()`
- 模型路由：`src/agents/model-forward-compat.ts`
- 模型兼容：`src/agents/model-compat.ts`
- 活躍模型過濾：`src/agents/live-model-filter.ts`
- Thinking 支援：`src/auto-reply/thinking.ts`

### 合成規則（以 GPT-5.4 為例）

| 合成模型 | Provider | 模板模型（按優先級） |
|----------|----------|---------------------|
| `gpt-5.4` | `openai` | `gpt-5.2` |
| `gpt-5.4-pro` | `openai` | `gpt-5.2-pro` → `gpt-5.2` |
| `gpt-5.4` | `openai-codex` | `gpt-5.3-codex` → `gpt-5.2-codex` |

### 額外參數支援

- **Thinking**：`gpt-5.4` / `gpt-5.4-pro` / `openai-codex/gpt-5.4` 支援 `xhigh` thinking 級別
- **serviceTier**：直接 `openai/*` 請求支援 `priority` 優先處理（`params.serviceTier`）
- **baseUrl/headers**：保留 `models.providers.openai` 中的自定義覆蓋

---

## 六、驗證清單

升級完成後逐項確認：

- [ ] `openclaw --version` 顯示目標版本
- [ ] `launchctl list | grep openclaw` 顯示 PID（Gateway 運行中）
- [ ] `openclaw models set <新模型>` 成功（如 `openai/gpt-5.4`）
- [ ] `openclaw models status` 顯示正確的默認模型
- [ ] `openclaw models status` 中目標 provider 無 "Missing auth" 警告
- [ ] 在 Dashboard 或 Telegram 中實際對話測試（如有 API Key）

---

## 七、給 AI Agent 的指引

如果你是一個 AI Agent，正在幫助用戶升級 OpenClaw 以支援新模型：

1. **先查 PR 狀態**：用 `gh pr view` 確認 PR 是否已合併
2. **先查 npm 版本**：用 `npm view openclaw version` 確認最新發行版
3. **比對 CHANGELOG**：grep PR 編號確認是否包含在當前版本中
4. **不要用 `npm install -g github:...`**：對大型 monorepo 極慢，用克隆+構建方式
5. **不要依賴 corepack**：直接 `npm install -g pnpm`
6. **不要只看 `models list`**：合成模型不顯示在列表中，用 `models set` 驗證
7. **升級後必須重啟 Gateway**
8. **切換 provider 後必須配置認證**
9. **不要在聊天中接受或使用 API Key**：指導用戶自行設置環境變數或使用 `openclaw configure`

---
> Source: [myyxfishcat/openclaw-model-upgrade](https://github.com/myyxfishcat/openclaw-model-upgrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
