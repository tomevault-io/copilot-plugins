## timerabbit-jira

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

各ディレクトリの詳細は対応する `CLAUDE.md` に記載する。

- `src/background/CLAUDE.md` — Service Worker（タイマー制御・ストレージ）
- `src/content/CLAUDE.md` — Content Script（Jira ページ検出・API・ウィジェット）
- `src/popup/CLAUDE.md` — Popup UI（React）
- `src/dashboard/CLAUDE.md` — Dashboard UI（React・集計・エクスポート）
- `src/shared/CLAUDE.md` — 共有型・メッセージ定義・ユーティリティ

## Commands

```bash
pnpm dev                                    # ウォッチビルド
pnpm build                                  # dist/ にビルド出力
pnpm test                                   # Vitest でテスト実行（全件）
npx vitest run src/path/to/file.test.ts     # 単一テストファイルを実行
npx tsc --noEmit                            # 型チェックのみ
```

ビルド後、Chrome の `chrome://extensions` で `dist/` フォルダを「パッケージ化されていない拡張機能を読み込む」で読み込む。

> **Note**: 現時点でテストファイルは存在しない（vitest は設定済み）。

## アーキテクチャ概要

Manifest V3 の Chrome 拡張。4つのコンテキストが `chrome.runtime.sendMessage` / `chrome.storage.local` で通信する。

```
Content Script  ──sendMessage──▶  Service Worker (Background)
     ▲                                      │
     │ tabs.sendMessage                 storage.local
     │                                      │
  Popup (React)                    Dashboard (React)
```

パスエイリアス: `@/*` → `src/*`

ビルドは `vite-plugin-web-extension` が `manifest.json` を起点として全エントリ（background / content / popup / dashboard）を自動解決する。**Tailwind CSS v4** を使用（`@tailwindcss/vite` プラグイン経由）。

`src/global.d.ts` で `chrome` グローバル型（TypeScript 6 対応）と CSS モジュール宣言を提供している。

## メッセージ送受信

`src/shared/messages.ts` の型付き `sendMessage()` ラッパーを使う。直接 `chrome.runtime.sendMessage` を呼び出すと型安全性が失われる。

```typescript
import { sendMessage } from "@/shared/messages";

// Popup / Dashboard → Service Worker
const state = await sendMessage({ type: "GET_STATE" });

// Popup → Content Script（tabId 必須）
const task = await sendMessage({ type: "GET_CURRENT_TASK" }, tabId);
```

## ストレージキー（`src/shared/utils/storage.ts`）

| キー | 内容 |
|---|---|
| `timeRabbit_timeLogs` | `TimeLog[]` — 全ログ |
| `timeRabbit_activeTimer` | `ActiveTimer \| null` — 計測中タイマー |
| `timeRabbit_taskMetaCache` | `Record<string, JiraTaskMeta>` — Jira メタキャッシュ |
| `timeRabbit_settings` | `ExtensionSettings` — 拡張設定 |

## ExtensionSettings（`src/shared/types.ts`）

```typescript
interface ExtensionSettings {
  jiraBaseUrls: Record<string, string>;    // プロジェクトキー → ベースURL（例: { "PROJ": "https://org.atlassian.net" }）
  showFloatingTimer: boolean;              // Jira ページ右下ウィジェットの表示制御
  floatingTimerCollapseToCorner: boolean; // 最小化時に画面右下へ移動するか
  storyPointsFieldId: string;
  sprintFieldId: string;
  metaCacheTtlMs: number;
  archiveThresholdDays: number;
}
```

`jiraBaseUrls` は「Jiraで開く」リンク生成に使用。`src/shared/utils/jira.ts` の `buildJiraUrl(taskKey, jiraBaseUrls)` 経由で参照する。

## Content Script の注入対象URL

`manifest.json` の `content_scripts.matches` に以下を登録済み:

- `/browse/*` — タスク詳細ページ
- `/jira/*/issues/*` — Issues ビュー
- `/jira/software/projects/*/boards/*` — ボード
- `/jira/software/c/projects/*/boards/*` — Company-managed ボード（バックログ含む）

ボード・バックログで課題パネルを開いた場合は URL の `?selectedIssue=TASK-123` クエリパラメータからタスクキーを抽出する（`JiraPageDetector._extractTaskKey`）。

## Extension context invalidated の扱い

拡張機能のリロード後、旧 Content Script が `chrome.storage` / `chrome.runtime` を呼ぶと `Extension context invalidated` エラーが発生する。`src/content/index.ts` でこのエラーを検知したら `detector.stop()` + `floatingTimer.unmount()` を呼んで処理を止める。`FloatingTimer` 内でも同様に自身を `unmount()` する。

---
> Source: [ITK13201/TimeRabbit-Jira](https://github.com/ITK13201/TimeRabbit-Jira) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
