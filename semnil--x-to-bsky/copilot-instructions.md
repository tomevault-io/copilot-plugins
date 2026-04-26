## x-to-bsky

> X (Twitter) の投稿ボタンをフックし、同じテキストを Bluesky に同時投稿する Chrome 拡張 (Manifest V3)。

# CLAUDE.md — X to Bluesky Crossposter

## プロジェクト概要

X (Twitter) の投稿ボタンをフックし、同じテキストを Bluesky に同時投稿する Chrome 拡張 (Manifest V3)。
外部依存ゼロ、全ファイル自作。API キー不要（X 側は DOM フック、Bluesky 側は AT Protocol 直接通信）。

## リポジトリ構成

```
x-to-bsky/
├── manifest.json        # Manifest V3 設定
├── background.js        # Service worker: Bluesky API (認証・投稿・画像・スレッド)
├── lib.js               # 純粋関数 (grapheme処理・テキスト分割・facet解析・URL検出)
├── shared.js            # content.js / options.js 共有定数 (DEFAULT_SELECTORS)
├── content.js           # x.com 上のコンテンツスクリプト (投稿・画像・スレッドフック)
├── content.css          # トースト通知・🦋バッジのスタイル
├── popup.html           # ツールバーポップアップ UI
├── popup.js             # ポップアップロジック (トグル ON/OFF)
├── options.html         # 設定画面 UI (認証・セレクタ設定)
├── options.js           # 設定ロジック (認証・セレクタ・i18n)
├── icons/               # 16/48/128px PNG + SVG ソースアイコン
│   ├── icon.svg         # SVG ソース (PNG 生成元)
│   └── generate-png.html  # PNG 生成ヘルパー
├── build.sh             # zip パッケージング
├── test/lib.test.js     # Vitest 単体テスト (lib.js)
├── package.json         # dev dependencies (vitest)
├── SECURITY_AUDIT.md    # セキュリティ監査レポート
├── README.md            # セットアップ手順・仕様説明
└── CLAUDE.md            # このファイル
```

## アーキテクチャ

### データフロー

```
[x.com compose area] → content.js (capture phase click) → chrome.runtime.sendMessage
  → background.js → AT Protocol (bsky.social) → createRecord → response
  → content.js → toast notification
```

### content.js

- `document.addEventListener("click", handler, true)` で capture phase フック
- セレクタはデフォルト定義 + `chrome.storage.local.customSelectors` で上書き可能
- テキスト抽出: `tweetTextarea_${n}` を 0 からインクリメントしてスレッド全体を取得
- 画像抽出: textarea 近傍の `<img>` を canvas 経由で base64 キャプチャ (同期処理)。`object-fit: cover` の画像は `getVisibleRect()` で表示領域のみをクロップしてキャプチャ。キャプチャ結果に `width`/`height` を含め、AT Protocol `aspectRatio` フィールドとして Bluesky に送信
- フォールバック: `.DraftEditor-root [data-text="true"]`
- 投稿ボタンに 🦋 バッジを MutationObserver で動的付与
- トースト通知で Bluesky 投稿結果を表示 (成功: 青, 失敗: 赤, スレッド件数表示)
- Service worker 起動: `sendMessageWithWakeup()` で `GET_STATUS` ping 後に本メッセージ送信。1秒タイムアウトのフォールバック付き。非冪等メッセージはリトライしない
- 拡張コンテキスト無効化ガード: `chrome.runtime?.id` チェックでリロード後のサイレント失敗を防止

### background.js

- `createSession()`: 認証 + セッションキャッシュ。401 時の明確なエラーメッセージ
- `resolveMentionFacets()`: メンション handle → DID 解決
- `uploadImage(base64, mimeType)`: `com.atproto.repo.uploadBlob` で画像アップロード
- `fetchOgpMetadata(url)`: 任意 URL から OGP メタデータ取得 (head のみストリーム読み取り)
- `uploadThumbnail(url, jwt)`: 画像 URL をダウンロードして Bluesky にアップロード
- `buildLinkEmbed(url, jwt)`: リンクカード (`app.bsky.embed.external`) 構築。OGP メタデータからタイトル・説明・サムネイルを取得
- `createPost(text, images, parent, root)`: 画像 embed / リンクカード / reply chain 対応
- `postThread(thread)`: 自動分割 + reply chain によるスレッド投稿
- メッセージハンドラ: `POST_TO_BSKY`, `TEST_LOGIN`, `GET_STATUS`

### lib.js

- `segmentGraphemes(text)` / `countGraphemes(text)`: `Intl.Segmenter` による正確な grapheme 単位処理
- `splitText(text)`: 300 grapheme 超のテキストを改行・スペースで自動分割
- `parseFacets(text)`: lookbehind 不使用。`(^|\s)` でバウンダリ判定し offset 補正
- `extractFirstUrl(text)`: テキスト中の最初の URL を検出 (末尾句読点除去)
- `calcCoverRect(nw, nh, dw, dh, objectPosition)`: `object-fit: cover` 時の表示領域クロップ計算 (pure function; content.js の `getVisibleRect` と同一アルゴリズム)

### 認証情報の保存

- `chrome.storage.local` に `bskyHandle`, `bskyAppPassword`, `crosspostEnabled`, `customSelectors`, `includeQuoteUrl`, `includeLinkCard`, `linkCardThumbnail` を保存
- App Password を使用（メインパスワードではない）
- `host_permissions`: `bsky.social`, `*.bsky.network`
- `optional_host_permissions`: `<all_urls>` — 設定画面でリンクカード機能を有効化した時点でウェブアクセス権限を一括付与。無効化すると自動解除。background.js は投稿時に `hasHostPermission()` で権限チェックし、未許可ドメインはカードなしで投稿

## コーディング規約

- 外部ライブラリ・ビルドツール不使用。生 JS のみ
- セミコロンあり、ダブルクォート不統一（manifest は JSON 標準）
- 関数に JSDoc コメント付与済み

## 既知の制限

- **画像キャプチャ**: canvas 経由のため、cross-origin 画像 (pbs.twimg.com 等) は tainted canvas で取得失敗する場合がある。blob: URL (ローカル添付) は問題なし
- **動画非対応**: Bluesky の動画投稿 API が安定したら対応を検討
- **スレッド**: X の「+」ボタンで複数ポストを一度に作成した場合のみ Bluesky でも reply chain として投稿される。既存ポストへの返信によるスレッド追加は、対応する Bluesky 側の親ポストを特定できないため未対応
- **引用 RT**: デフォルトではコメントテキストのみ投稿される。設定画面の「投稿オプション」で有効化すると、引用元ポストの X リンクをテキスト末尾に追加可能
- **リンクカード**: URL を含むポストで画像がない場合、OGP メタデータからサムネイル付きリンクカードを自動生成。OGP 非対応サイトはタイトルなしカードにフォールバック。デフォルト無効、設定画面で有効化時にウェブアクセス権限を一括付与 (無効化で自動解除)。未許可ドメインはカードなしで投稿
- **RT (リポスト)**: テキスト入力を伴わないため作用しない（意図通り）

## テスト方法

1. Chrome Web Store からインストール、または `chrome://extensions` → デベロッパーモード → フォルダ読み込み
2. 設定画面で Bluesky App Password を設定 → 接続テスト
3. x.com でテスト投稿 → Bluesky 側で投稿確認
4. 画像付き投稿 → Bluesky 側で画像が表示されることを確認
5. スレッド投稿 (X の「+」ボタンで複数ツイート) → Bluesky でも reply chain になることを確認
6. 300 文字超テスト → 自動分割されてスレッドとして投稿されることを確認
7. ポップアップで OFF → 投稿しても Bluesky に送信されないことを確認
9. 設定画面の Advanced → セレクタ変更・リセットが反映されることを確認
10. 設定でリンクカードを有効化 → 権限ダイアログが表示され、許可後にトグル ON になることを確認
11. URL 付きテスト投稿 → Bluesky 側でサムネイル付きリンクカードが表示されることを確認
13. 設定でリンクカードを無効化 → カードが生成されないことを確認
14. 設定でサムネイルを無効化 → タイトルのみのカードになることを確認

## ビルド

```bash
chmod +x build.sh
./build.sh
# → x-to-bsky-v1.0.0.zip が生成される
```

## 関連コンテキスト

- claude-voice 拡張の開発経験あり (Manifest V3, MutationObserver, content script パターン)
- check-handle プロジェクトで Playwright + DOM セレクタ探索の経験あり
- Bluesky AT Protocol の認証は App Password 方式 (OAuth は未使用)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/semnil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
