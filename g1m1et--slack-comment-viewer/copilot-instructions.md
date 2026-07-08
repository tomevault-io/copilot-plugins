## slack-comment-viewer

> Webページ上にSlackチャネルのメッセージをオーバーレイ表示するChrome拡張（Manifest V3）。

# CLAUDE.md - Slack Comment Viewer

Webページ上にSlackチャネルのメッセージをオーバーレイ表示するChrome拡張（Manifest V3）。
ウェビナー等でプレゼン画面上にリアルタイムのコメントを流す用途。

## アーキテクチャ

サーバー不要。Chrome拡張内で完結する。

```
popup.html/js ── 設定UI（token, channel, 表示設定）
       │
       ▼ chrome.storage.local
       │
background.js ── Slack API ポーリング（3秒間隔）
       │
       ▼ chrome.tabs.sendMessage
       │
content.js/css ── ページ上にオーバーレイ描画
```

- **background.js**: Service Worker。`conversations.history` をポーリングし、新着メッセージを全タブに送信。`users.info` でユーザー名解決（キャッシュ付き）。`_lastTs` を chrome.storage.local に永続化してワーカー再起動に対応
- **content.js**: 全ページにインジェクト。2つの表示モードを持つ
  - **Ticker**: 画面右下（or左下）に固定表示。FIFO で古いメッセージを除去
  - **Flow**: ニコニコ動画風の右→左スクロール。8レーン制で重複回避
- **popup.html/js**: 設定画面。変更は即座に chrome.storage.local に反映

## Slack App 設定

Bot Token Scopes:
- `channels:history`, `channels:read` — パブリックチャネル
- `groups:history`, `groups:read` — プライベートチャネル
- `users:read` — ユーザー名解決

## 開発

ビルド不要。ソースをそのまま Chrome にロードする。

1. `chrome://extensions` → デベロッパーモード ON
2. 「パッケージ化されていない拡張機能を読み込む」→ このディレクトリを指定
   - WSL環境: `\\wsl$\Ubuntu\home\<user>\ghq\github.com\g1m1et\slack-comment-viewer`
3. コード変更後は拡張を「更新」ボタンで再読み込み

デバッグ:
- background.js: `chrome://extensions` → Service Worker の「検証」リンク
- content.js: 対象ページの DevTools コンソール
- popup: ポップアップ上で右クリック →「検証」

## 設計上の注意点

- **MV3 Service Worker のライフサイクル**: ワーカーは Chrome により随時終了・再起動される。対策として 2 段構え：
  - `setInterval(pollOnce, 3000)` で通常時の 3 秒間隔ポーリング
  - `chrome.alarms` を 1 分間隔の watchdog として並走させ、SW 再起動時に `pollingTimer === null` を検知して `startPolling()` で自己治癒
  - `_lastTs` は `chrome.storage.local` に永続化され、SW 再起動時も差分取得を維持。`startPolling()` は冒頭で `pausePolling()`（タイマー停止のみ）を呼び、`_lastTs` の wipe は `resetPollingState()`（無効化や channel 変更時のみ呼ばれる）に分離されている
- **ポーリング重複防止**: `polling` フラグで同時実行を排除
- **number入力の中間値問題**: popup.js の number フィールドは `change` イベントのみで保存（`input` イベントだとキー入力中の空文字が `0` として保存され表示が停止する）
- **seenTs による重複排除**: content.js 側で ts ベースの重複チェック。background からの再送に対応
- **`oldest` パラメータ**: Slack API の inclusive 境界を回避するため `+0.000001` している

## Linear

- チーム: KIRI（個人プロジェクト扱い）
- プロダクト識別ラベル: `SLACKCV`

このリポジトリ関連の observation / hypothesis / implementation には `SLACKCV` ラベルを付与する。ナレッジ種別ラベル（`observation` / `hypothesis` / `implementation` / `feedback`）と併用する。

---
> Source: [g1m1et/slack-comment-viewer](https://github.com/g1m1et/slack-comment-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
