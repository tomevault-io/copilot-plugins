## ai-editing

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概述

本專案包含兩個獨立的影片處理系統：

1. **auto-cut-agent/** — A-Roll 粗剪工具：語意型重錄偵測、多軌麥克風同步、繁體中文字幕、EDL/XML/SRT 匯出
2. **auto-process/** — 課程影片自動剪輯：靜音偵測裁剪（開頭/結尾/中間休息）+ YouTube 自動上傳

## 常用指令

### auto-process（課程影片自動剪輯）

```bash
# 安裝
cd auto-process
setup.bat                                    # 一鍵安裝（pip + 建資料夾）
python youtube_uploader.py --auth            # 首次 YouTube 認證

# 手動裁剪單一影片
python auto-process/course_trimmer.py "影片.mp4"

# 手動上傳到 YouTube
python auto-process/youtube_uploader.py "影片.mp4"

# 啟動資料夾監聽 daemon（放影片到 inbox/ 自動處理+上傳）
python auto-process/daemon.py
```

### auto-cut-agent（A-Roll 粗剪）

```bash
# 安裝依賴
pip install -r auto-cut-agent/requirements.txt

# 基本使用
python auto-cut-agent/main.py "影片.mp4"

# 完整參數
python auto-cut-agent/main.py "影片.mp4" \
  --external_audio "麥克風A.wav" "麥克風B.wav" \
  --xml "PluralEyes.xml" \
  --model medium \
  --threshold 80 \
  --merge-gap 0.3 \
  --render

# 批量處理（需編輯腳本內的 target_dirs）
python auto-cut-agent/batch_process.py

# 只重新生成字幕（使用 Whisper 快取）
python auto-cut-agent/srt_only.py

# 從既有 EDL/SRT 重新生成 XML
python auto-cut-agent/regenerate_from_existing.py

# 測試
python auto-cut-agent/test_spacy.py
python auto-cut-agent/test_sync_direction.py
```

## 系統需求

- Python 3.8+、FFmpeg（須在 PATH）、PyTorch（建議 CUDA 版）
- Spacy 中文模型 `zh_core_web_md`（首次執行自動下載）

## 架構與資料流

```
main.py（主入口，串接所有步驟）
  │
  ├─ audio_extractor.py    影片 → 16kHz mono WAV（FFmpeg）
  ├─ audio_sync.py         外部麥克風同步（cross-correlation @8kHz）
  ├─ transcriber.py        Whisper 轉錄 + 多檔快取（whisper_cache_v2.json）
  ├─ intelligence.py       retake 偵測（FuzzyWuzzy 後向掃描）+ segment 合併
  ├─ edl_generator.py      CMX 3600 EDL 輸出
  ├─ subtitle_generator.py SRT 字幕（OpenCC 簡繁 + Spacy NLP 斷句，≤12字/行）
  ├─ xml_exporter.py       Final Cut Pro 7 XML（含嵌入字幕軌）
  └─ video_renderer.py     FFmpeg stream copy 無損渲染
```

### 輔助工具

| 腳本 | 用途 |
|------|------|
| `batch_process.py` | 多目錄批量處理（跳過 `_synced`/`_cut` 檔案） |
| `organize_files.py` | 整理成 `01_Originals/02_Synced/03_Exports` 結構 |
| `regenerate_from_existing.py` | 從現有 EDL+SRT 重新生成 XML |
| `regenerate_xmls.py` | 用 Whisper 快取快速重新生成 XML |
| `srt_only.py` | 單獨重新生成字幕 |
| `xml_parser.py` | 解析 PluralEyes XML 取得 video→audio 對應 |

## 核心演算法

### Retake Detection（intelligence.py）
從後向前掃描相鄰 segment，用 `fuzz.ratio()` 計算文本相似度。超過閾值（預設 80）則標記前段為重錄、保留後段（後面的版本通常更完整）。

### Audio Sync（audio_sync.py）
用 cross-correlation 計算外部麥克風與機身音訊的時間偏移量（lag）。降取樣至 8kHz 加速運算。lag < 0 表示外麥先開，需 trim；lag > 0 表示外麥晚開，需 pad。

### Smart Subtitles（subtitle_generator.py）
OpenCC 簡繁轉換 → Spacy 依賴解析 → 智慧斷句（標點 > 動詞標記 > 連接詞 > 代詞主語），強制每行 ≤ 12 字。

## 輸出檔案命名

| 後綴 | 說明 |
|------|------|
| `_synced.mp4` | 音訊已替換版本 |
| `_cut.edl` | CMX 3600 剪輯清單 |
| `_cut.srt` | 繁體中文字幕 |
| `_cut.xml` | FCP7 序列（含字幕軌） |
| `_final.mp4` | 渲染後最終影片（`--render`） |
| `_final.srt` | 重新轉錄的精確字幕 |

## 開發注意事項

- Whisper 快取存在系統 TEMP 目錄下的 `whisper_cache_v2.json`，避免重複轉錄
- XML 匯出使用 NTSC 標準（29.97 FPS），EDL 預設 30 FPS non-drop frame
- `video_renderer.py` 用 stream copy 不重新編碼，可能有 ±1 frame 誤差
- `auto-cut-agent/` 的腳本位於該目錄下
- `auto-process/` 是獨立套件，不依賴 auto-cut-agent，可單獨複製到其他電腦
- `auto-process/` 的設定透過專案根目錄的 `.env` 管理（參考 `.env.example`）
- `/auto-process` Claude Code Skill 可手動觸發裁剪、查看狀態、管理 daemon

---
> Source: [joe51317-dotcom/AI-editing](https://github.com/joe51317-dotcom/AI-editing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
