## claude-code-video-kit

> > **你是 Claude Code。** 這個 repo 是「給 Claude Code agent 用的三類影片製作 kit」。

# CLAUDE.md — Claude Code 影片製作 Kit（入口）

> **你是 Claude Code。** 這個 repo 是「給 Claude Code agent 用的三類影片製作 kit」。
> 使用者說「我要做影片 / 做一支教學影片 / 做社群科普 / 按三類影片規範做…」時，依本檔的流程執行。
> 本 kit 是自包含的：specs（規範）、pipeline（可重用管線）、examples（可 fork 的完整範例）、GOTCHAS（踩坑）全在 repo 內，不依賴外部。

---

## 為什麼有這個 repo（給你看的背景）

原始 `claude-video-specs` 是給四種 agent（Claude Code / Codex / OpenCode / Antigravity）共用的，入口是 `AGENTS.md` + `opencode.json`。**但 Claude Code 的原生記憶檔是 `CLAUDE.md`，不會自動讀 AGENTS.md**，導致 Claude Code 進場時不會自動進流程、且 bash-first 的工具鏈在 Windows 上常卡住。

這個 repo 把「Claude Code 跑得順」的版本獨立出來：

- 入口就是這份 **CLAUDE.md**（你會自動讀到）
- 管線**不用 HyperFrames CLI**（它在 Windows + Node 24 會 crash），改用純 CSS/JS + Playwright + FFmpeg
- 所有踩坑修法已內建在 pipeline 腳本裡

---

## 你的工具對照（Claude Code）

| 動作 | 你的工具 |
|------|---------|
| 執行 shell | `Bash`（Windows 也可用 `PowerShell`） |
| 讀 / 寫 / 編輯檔 | `Read` / `Write` / `Edit` |
| 抓網頁 / 搜尋 | `WebFetch` / `WebSearch` |
| 打包技能路徑 | `~/.claude/skills/<name>/SKILL.md` |

---

## 🔴 最高鐵律（無可妥協，對應 GOTCHAS A-1/A-2/A-3）

**任何 code / TTS / 渲染之前，第一步一定是產出 `SCRIPT.md` 與 `DESIGN.md`，在對話中展示給使用者審查，等使用者明確說「go」才動工。**

- `SCRIPT.md`：5 段敘事逐頁稿（旁白 + 字幕 + 版面 + 動畫描述）+ 素材清單
- `DESIGN.md`：字體 / 配色 / 字級 / 版面 / 動畫節奏
- 不要悶頭做；每完成一個里程碑就回報。

---

## 5 階段流程

### 階段 1｜環境檢查
依序確認並把結果整理成表格給使用者（缺的才逐項問要不要裝）：

| 元件 | 檢查 | 安裝 |
|------|------|------|
| Python 3.8+ | `python --version` | python.org |
| edge-tts | `pip show edge-tts` | `pip install edge-tts` |
| Node 18–22（**避開 24**） | `node --version` | nodejs.org，建議 20 LTS |
| ffmpeg / ffprobe | `ffmpeg -version` | `winget install Gyan.FFmpeg` |
| Playwright（裝在 %TEMP%） | 見下方 | `pipeline/PIPELINE.md` |
| 源石黑體 | 系統字體夾有 `GenSekiGothic2TW-H.otf` | ButTaiwan/genseki-font |

> ⚠️ Node 24 會讓部分 native 工具 crash（GOTCHAS C-4）。本 kit 純 CSS/JS 管線在 Node 24 上**可正常錄影**，但若你要用任何 CLI 渲染器，請降 Node 20 LTS。

### 階段 2｜介紹三類影片
讀 `specs/` 的定位段，給**簡短**對照表（不要倒整份 spec）：

| # | 類型 | 片長 | 核心 |
|---|------|------|------|
| 01 | 活動紀錄 | 60–180s | 口白 + 大字卡 + BGM |
| 02 | 教學影片 | 4–8 min | SOIL 脈絡 + 動畫 + TTS |
| 03 | 社群科普 | 2–3 min | 強 Hook + 多版面 + 照片 |

問使用者要試做哪一類 + 主題 + 素材狀況。

### 階段 3｜試作（先讀 GOTCHAS.md）
1. **fork** `examples/` 對應範例到使用者工作目錄
2. 依該類 spec 的 checklist 寫 `SCRIPT.md` + `DESIGN.md` →（鐵律）等「go」
3. 找素材：Unsplash 直連下載到本地 `assets/images/`（用 `images.unsplash.com/photo-XXX`）
4. 跑 pipeline（見下）→ 渲染 mp4 → 給使用者預覽
5. 每個里程碑回報

### 階段 4｜調整
依使用者反饋改字幕/視覺/節奏/素材/片長，**重新渲染**再確認。

### 階段 5｜打包技能（選配）
把流程打包成 `~/.claude/skills/<name>/SKILL.md`，附觸發詞。

---

## 管線（pipeline/，已驗證可在 Windows + Node 24 跑）

```
generate_narration.py   # Edge-TTS 序列生成 page-NN.mp3（改 SCRIPT 陣列）
get_durations.py        # ffprobe 量時長，輸出 PAGES 表（改 SUBTITLES 陣列）
index.html              # 13 頁多版面動畫骨架，?render=true 自動播放（見 examples）
record.cjs              # Playwright 錄無聲 webm（改錄製秒數）
render.py               # 合成主音軌 → 錄影 → mux 成 final.mp4
```

**三個必改點**（詳見 `pipeline/PIPELINE.md`）：
1. `generate_narration.py` 的 `SCRIPT`（旁白）
2. `get_durations.py` 的 `SUBTITLES` + index.html 的 `PAGES`（兩者時長必須一致）
3. `render.py` 的 `PAGES_TIMINGS` + `record.cjs` 的錄製秒數（= 總時長 + 0.8s）

**內建避坑**：
- ffmpeg mux 一律 `-map 0:v -map 1:a`（GOTCHAS E-2，避免沒聲音）
- node_modules / 渲染暫存放 `%TEMP%\cvs-render\`，NODE_PATH 指過去（GOTCHAS D-1/D-2，避開 GDrive + 中文路徑）
- HTML 內 `@font-face` 指向本地 `.otf`（GOTCHAS C-5）
- `?render=true` 隱藏遮罩自動播放（GOTCHAS D-3）
- Python 強制 utf-8 輸出（GOTCHAS F-1，避開 CP950）

---

## 參考

- `specs/` — 三類影片硬規範
- `pipeline/PIPELINE.md` — 管線使用 + Playwright 安裝
- `examples/03-opus-4-8/` — 完整 03 社群科普範例（Claude Code 實作，可直接 fork）
- `GOTCHAS.md` — 開工前必讀的踩坑清單

---
> Source: [mathruffian-dot/claude-code-video-kit](https://github.com/mathruffian-dot/claude-code-video-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
