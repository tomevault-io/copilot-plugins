## music-collector

> 自動從 13 個音樂評論網站蒐集推薦曲目，同步至 Spotify 與 Apple Music 播放清單。

# Music Collector — 專案指引

## 專案概述

自動從 13 個音樂評論網站蒐集推薦曲目，同步至 Spotify 與 Apple Music 播放清單。

## 開發指令

```bash
# 安裝依賴
uv sync

# 安裝含 Playwright 瀏覽器渲染
uv sync --extra browser

# 安裝含測試工具
uv sync --extra test

# 安裝含 Web 介面
uv sync --extra web

# 安裝所有可選依賴
uv sync --all-extras

# 乾跑測試（不需 Spotify 憑證）
./run.sh --dry-run

# 完整執行（需 .env 中的 Spotify 憑證）
./run.sh

# 查看近期蒐集紀錄
./run.sh --recent 7

# 檢視備份（列出所有 / 指定季度）
./run.sh --backup
./run.sh --backup Q1

# 匯出備份供 Apple Music 匯入
./run.sh --export Q1              # 匯出為 CSV
./run.sh --export Q1 --format txt # 匯出為純文字
./run.sh --export Q1 --all        # 包含未在 Spotify 找到的曲目

# 輸出 Spotify 播放清單連結（供轉換至 YouTube Music / Tidal）
./run.sh --export-spotify-url

# 完整執行 + Apple Music 同步（擷取 → Spotify → Apple Music）
./run.sh --import Q1

# 資料分析
./run.sh --stats              # 總覽
./run.sh --stats overlap      # 跨來源重疊分析
./run.sh --stats sources      # 各來源效能比較

# Web 介面
./run.sh --web

# 清除歌單與資料庫，重新蒐集
./run.sh --reset

# 執行測試
PYTHONPATH=src uv run pytest tests/ -v

# Docker 執行
docker compose run collector --dry-run
```

> `run.sh` 等同 `PYTHONPATH=src uv run python -m music_collector`。

## 架構要點

- `src/music_collector/scrapers/base.py` — `BaseScraper` 抽象類別、`Track` 資料模型、`_get_rendered()` Playwright 方法
- `src/music_collector/scrapers/__init__.py` — `ALL_SCRAPERS` 註冊表（13 個擷取器）
- `src/music_collector/spotify.py` — Spotify 整合（搜尋驗證、播放清單管理、季度歸檔）
- `src/music_collector/db.py` — SQLite 去重，以 `(artist, title)` 為唯一鍵
- `src/music_collector/backup.py` — 季度 JSON 備份至 `data/backups/YYYY/QN.json`
- `src/music_collector/export.py` — 匯出為 CSV/TXT（`export_combined_spotify()` 合併主歌單 + 歸檔歌單並去重匯出，供 `--apple-music` 使用；`export_from_spotify()` 僅匯出主歌單；舊函式 `export_csv()`/`export_playlist()` 從備份 JSON 讀取）+ Spotify URL 匯出
- `src/music_collector/apple_music/` — Apple Music 自動匯入（模組化套件）
  - `api.py` — **主要入口**：直接呼叫 Apple Music REST API（搜尋曲目、建立/刪除播放清單、分批加入曲目）
  - `browser.py` — Chrome driver 建立與反偵測措施
  - `playlist.py` — 播放清單管理（MusicKit JS API + AppleScript）
  - `transfer.py` — TuneMyMusic 自動化轉移（備援方案，含 Premium 付費牆偵測修正）
- `src/music_collector/tunemymusic.py` — 向後相容，重新匯出 `apple_music` 套件
- `src/music_collector/notify.py` — LINE + Telegram + Slack 多通道通知
- `src/music_collector/stats.py` — 資料分析（總覽、重疊、來源比較）
- `src/music_collector/web.py` — Streamlit Web 介面
- `src/music_collector/main.py` — 主流程與 CLI（13 個擷取器透過 `asyncio.gather` 並行執行）
- `tests/` — 82 項測試（pytest + respx mock）

### 擷取器技術細節

| 擷取器 | 方式 | 解析策略 |
|--------|------|----------|
| Pitchfork | HTML | `div[class*='SummaryItemWrapper']` 容器，`h3` 取曲名，`div sub-hed` 取藝人 |
| Stereogum | RSS | feedparser + 分類過濾 + 多格式標題解析 |
| NME | HTML | `/reviews/track` 頁面，敘述性標題解析（所有格 + 動詞短語分離） |
| SPIN | HTML | `/new-music/` 頁面，typographic 引號匹配 + 動詞短語分離 |
| Consequence | HTML | 引號提取曲名 + `_extract_artist_from_prefix()` 動詞邊界偵測 |
| Line of Best Fit | HTML | 所有格 `'s` 優先策略 + 擴展動詞清單 |
| Rolling Stone | HTML | 二階段：索引頁篩選推薦文章 → 文章頁提取曲目 |
| Slant | HTML | 引號提取 + JS/Cloudflare 偵測 |
| Complex | HTML | 多 URL 嘗試 + JS 偵測 + Playwright fallback |
| Resident Advisor | HTML | Next.js 偵測 + Playwright fallback |
| Gorilla vs. Bear | RSS | feedparser + mp3/video/on-blast 分類過濾 |
| Bandcamp Daily | RSS | feedparser + Album of the Day 分類 + 逗號分隔解析 |
| The Quietus | RSS | feedparser + Reviews 分類過濾 |

### 引號處理注意事項

- Consequence：不可將直引號 `'` 放入引號匹配字元集，否則所有格會被誤判為開引號
- NME / SPIN：使用 `\u2019(?![a-zA-Z])` negative lookahead 避免將縮寫撇號（如 Where's）誤判為結尾引號
- SPIN：引號內曲名需 `rstrip(".,;:!?")` 移除尾端標點

## 新增擷取器

1. 在 `src/music_collector/scrapers/` 建立新檔案
2. 繼承 `BaseScraper`，實作 `fetch_tracks()` 回傳 `list[Track]`
3. 在 `scrapers/__init__.py` 的 `ALL_SCRAPERS` 中註冊
4. 在 `tests/scrapers/` 新增對應測試
5. 用 `--dry-run` 測試

## Playwright 瀏覽器渲染

用於 JS 重度渲染網站（Complex、Resident Advisor）：

1. 安裝：`uv sync --extra browser && uv run playwright install chromium`
2. 啟用：在 `.env` 設定 `ENABLE_PLAYWRIGHT=true`
3. 行為：httpx 請求失敗時自動 fallback 至 Playwright headless 瀏覽器
4. 未安裝/未啟用時靜默跳過，不影響其他來源

## Apple Music 直接 API 匯入

### 流程架構（`api.py`）

```
讀取 CSV → Selenium 開啟 music.apple.com → 提取 MusicKit token
→ 逐一搜尋 (artist, title) → 取得 catalog track ID
→ 刪除同名舊歌單 → 建立新播放清單 → 分批 POST 加入曲目（每批 ≤300）
```

- 不依賴任何第三方轉換服務，直接透過 `https://api.music.apple.com` 呼叫
- Token 從 `MusicKit.getInstance().developerToken` / `.musicUserToken` 提取
- 加入曲目使用 `POST /v1/me/library/playlists/{id}/tracks`，回傳 204 No Content 視為成功

### Token 取得

1. Selenium 開啟 `music.apple.com`，等待 MusicKit JS 初始化（約 5–10 秒）
2. 若已授權（`isAuthorized === true`），直接讀取 `developerToken` + `musicUserToken`
3. 若未授權，JS 點擊 Sign In 按鈕（Svelte class `button.signin`），等待 Apple ID 登入彈窗關閉，輪詢 token 寫入

### 搜尋策略

- 兩輪查詢：`"{title} {artist}"` 及 `"{artist} {title}"`
- 藝人 + 曲名雙重驗證（`_is_match` 子字串包含）
- 間隔 0.15 秒避免 rate limit

### 反偵測措施

MusicKit JS 會偵測無痕模式。`browser.py` 中的防護：

- `Page.addScriptToEvaluateOnNewDocument` 注入反偵測腳本
- `navigator.storage.estimate` quota 偽裝
- 持久化 `data/browser_profile/` 保存登入狀態（不可推送至 Git）

### 已知限制

- 首次使用需手動完成 Apple ID 登入（約 30 秒）
- Apple Music API 每次 POST 最多加入 300 首，超過需分批
- `data/browser_profile/` 儲存 Chrome profile（不可推送至 Git）

## TuneMyMusic 備援方案（`transfer.py`）

`transfer.py` 保留作為備援，使用 Selenium 自動化 TuneMyMusic UI。注意：

- TuneMyMusic 免費方案需要 Premium 才能轉移至 Apple Music
- Premium 付費牆偵測：`//button[contains(text(), 'Premium') and contains(text(), 'Transfer')]`
- `import_to_apple_music()` 目前由 `api.py` 提供（`__init__.py` 匯入 `api.py`）

## 自動排程（launchd）

### 設定檔

- 專案內：`com.music-collector.plist`
- 安裝位置：`~/Library/LaunchAgents/com.music-collector.plist`
- 每日 09:00 執行，log 輸出至 `data/collector.log`

### 排程指令

```bash
# 安裝排程
cp com.music-collector.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.music-collector.plist

# 移除排程
launchctl unload ~/Library/LaunchAgents/com.music-collector.plist

# 檢查狀態
launchctl list | grep music-collector

# 手動觸發（測試用）
launchctl start com.music-collector
```

### 執行流程（`run(sync_apple_music=True)`）

排程使用 `--apple-music` 旗標，`--import` 亦觸發相同流程。執行順序嚴格保證 **Spotify 先完成，再進行 Apple Music**：

```
1. 擷取新曲目（13 個來源）
2. Spotify 更新（僅有新曲目時）：搜尋 → 加入歌單 → 備份
3. Apple Music 匯入（無論是否有新曲目都執行）：從 Spotify API 合併匯出 CSV（主歌單 + 歸檔歌單，去重後全量匯出） → Apple Music REST API 直接匯入
4. 發送通知
```

Apple Music 匯入與新曲目解耦，確保即使當天無新曲目，前次失敗的匯入仍會重試。

## 注意事項

- `.env`、`.spotify_cache`、`data/` 不可推送至 Git
- 每個擷取器必須獨立處理例外，不可影響其他來源
- Spotify 搜尋先用精確查詢 `track: artist:`，失敗後再用寬鬆查詢，兩者皆需通過藝人 + 曲名雙重驗證
- 曲目去重以大小寫不敏感的 `(artist, title)` 比對
- 備份/通知各自 try/except，失敗不影響主流程
- `--dry-run` 模式不觸發 Spotify 操作、備份與通知

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/heinzyao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
