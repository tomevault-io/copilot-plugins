## pdf-tool

> ~/workspace/vs code/pdf/ — 純前端 SEN 教師 PDF 工具(零後端、零外流)。

# AGENTS.md — PDF Workstation

## Product Scope (locked 2026-06-26)

~/workspace/vs code/pdf/ — 純前端 SEN 教師 PDF 工具(零後端、零外流)。

User-confirmed product decisions:
- **OCR 比例**: 80% 數位 PDF / 20% scan → Hybrid 路線(先 PDF.js text layer, fallback Tesseract.js WASM)啱用
- **脫敏 trigger**: 用戶 supply keyword list (eg upload 學生名 .txt), 唔做 manual rect select
- **浮水印**: generic placeholder 5 個 + 用戶 upload 自選校徽, generic 即可 demo, 校徽之後用戶自己換
- **3 個 P0 已 confirm**: OCR+搜尋、keyword redaction、watermark

## Hard Constraints

- **DO NOT use OCR API** (Google Vision / MiniMax) — 違反零外流 stance, 即使更準都唔揀
- **零後端** — 所有 processing 喺 browser
- **純 client-side** — 唔好 propose backend proxy / server endpoint
- 學生私隱優先 — 唔可以 leak filename / content 到 third-party service

## A5 PDF metadata strip — known limitation (v3.0h)

`stripPdfMetadata` 喺 v3.0f commit,然後 v3.0h 強化。**pdf-lib v1.17.1 嘅 `setProducer()` / `setModificationDate()` 喺 save 嘅時候會被無條件覆蓋返 default**(`pdf-lib (https://github.com/Hopding/pdf-lib)` + fresh ModDate)。`setCreator()` 同 `setCreationDate()` 只係 conditional fallback,如果事先 set 咗會保留。

**結論**:
- ✅ 真係 strip 到:Author / Title / Subject / Keywords / CreationDate(privacy-critical, AGENTS.md stance 滿足)
- ⚠️ Library fingerprint 仲 leak:Producer / Creator / ModDate(只係 stack info,唔係學生私隱 — **stance satisfied**)

**T11 test harness 自動 catch 任何 regression**。改 `stripPdfMetadata` 嘅時候要 verify 嗰 5 個欄位仲 strip 到。**唔好隨便升 pdf-lib** — 升之前先 audit `updateInfoDict()` 嘅 producer/creator/modDate 行為有冇變。

## Test harness(v3.0h 新增)

`tests/` 目錄有 E2E test harness:
- **`npm test`** 喺 tests/ 入面跑 T1 + T2.merge + T11
- **`puppeteer-core` + system Chrome** — 唔下載 280MB bundled Chromium
- **Fixtures 即時生成** — 唔 commit binary blobs
- **T11 自動 catch A5 regression**

每個新 mode / 新 feature commit 之前必須 verify `npm test` 仲 pass。**改 `stripPdfMetadata` / watermark / `downloadPdfWithWatermark` 嘅時候必跑 `npm run test:t11`** — silent bug 嘅 detector。

## Workflow Notes

- User 提到呢個 project 時，唔好 propose 新 backend architecture
- Default posture：fixes only 或者極 small enhancement
- 真要加 features 要先 confirm scope
- **改 metadata strip / watermark / global download path 之前必跑 T11**

---
> Source: [ihateusingai-beep/pdf-tool](https://github.com/ihateusingai-beep/pdf-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
