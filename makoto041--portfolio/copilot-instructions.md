## portfolio

> Next.js 15とPayloadCMS 3.38.0で構築された現代的なポートフォリオサイトです。タイムライン、ブログ投稿、ギャラリー、イベント、制作物紹介機能を備えています。

# ポートフォリオプロジェクト - Claude Code アシスタントガイド

## プロジェクト概要

Next.js 15とPayloadCMS 3.38.0で構築された現代的なポートフォリオサイトです。タイムライン、ブログ投稿、ギャラリー、イベント、制作物紹介機能を備えています。

## 開発フロー（毎回この手順で進める。ユーザーの再指示は不要）

変更対応は、以下を**自走で最後まで**完結させる。軽微な変更（typo・単純修正）は 1・3 を省略してよい。

1. **設計/UX を変える変更はまずモック**: Artifact で HTML モックを作り、方向性の合意を得てから実装に入る（既存の「Mist Terminal」トークン/意匠を踏襲）。**サーバー環境で作業していて Artifact が使えない/実機で見せたい場合は、下記「サーバー環境でのプレビュー」の tailnet 配信で URL を渡し、実機（iPhone / Mac など SP・PC 両方）で確認してもらう。**
2. **差分で実装**: 既存パターン（コンポーネント構成・Payloadコレクション・`src/app/(frontend)/mist.css`）に沿って、ゼロから作り直さず改修する。
3. **Fable 5 レビュー**: 非自明な実装は Fable 5 をサブエージェント（Agentツール `model: fable`）に指名して設計/コードレビュー → 指摘修正 → 再確認のループを回す（専用 `advisor()` が使えない環境のため、Fableサブエージェントで代替）。
4. **自走で PR → マージ**: 完了したらブランチを切って PR を作成（**main への直コミット禁止**）。CI（Vercelビルド）と PRレビュー（CodeRabbit）の完了を待ち、**指摘があれば修正して再確認、無ければ `gh pr merge --squash` でマージ**する。
5. **DBマイグレーション**: コレクション/グローバルのフィールド変更は `pnpm payload migrate:create <name>` で**生成してコミットするだけ**。適用（`pnpm payload migrate`）はデプロイ側に委ねる（`.env` は本番 Neon を指すためローカル/CIから本番へは適用しない）。
6. **検証**: `pnpm generate:types` / `pnpm build` / `pnpm lint` をローカルで通してから PR にする。

## サーバー環境でのプレビュー（tailnet 経由で実機確認）

サーバー（`debian-ai`）で作業していて、モックや変更を **iPhone / Mac などの実機ブラウザ（SP・PC 両方）**で見せたいときの確立済み手順。SP は実機幅での確認、PC はブラウザ幅を変えてブレークポイントを確認、と用途に応じて下記 A / B を使い分ける。共通の前提: tailnet IP のみにバインドして tailnet 内限定で配信する（`0.0.0.0` は不可）。停止は `fuser -k <port>/tcp`（`pkill -f http.server` は**コマンド文字列が自分自身にマッチして落ちる**ため使わない）。URL は `http://<tailscale-ip>:PORT/` か `http://debian-ai.<tailnet>.ts.net:PORT/`（MagicDNS 名・分かりやすい）を渡す。

### A. 静的モック（HTML 単体を素早く見せる。方向性合意フェーズ向き）
1. **secret を含まない配信ディレクトリを用意**: リポジトリ直下は `.env`（本番 secret）を含むため**絶対に配信しない**。scratchpad 等へ HTML を `index.html` としてコピーした専用ディレクトリを作る。
2. tailscale IP のみにバインドして配信:
   ```bash
   TS_IP=$(tailscale ip -4)            # 例: 100.126.238.0
   cd <secret無しの配信ディレクトリ>
   python3 -m http.server 8000 --bind "$TS_IP"
   ```
   → `http://<TS_IP>:8000/` を渡す。PC で見る場合はモックの枠を固定幅にせず、実機幅（SP=375px 等）とデスクトップ幅の両方を並べると誤解が少ない。

### B. 実アプリ / レスポンシブ確認（PC でブレークポイントまで見たいとき）
静的モックでなく**実際のページ**を確認したい場合は、Next dev サーバーを tailscale IP にバインドして公開する。Next は任意ファイルを列挙配信しない（`.env` はファイルとして露出しない）ため、この用途では実アプリを直接出してよい。
```bash
TS_IP=$(tailscale ip -4)
pnpm dev -H "$TS_IP"               # 既定 3000 番
```
→ `http://debian-ai.<tailnet>.ts.net:3000/` を渡す。PC ブラウザで幅を狭めれば `@media (max-width:760px)` 等の切替を実データで確認できる。確認後は dev サーバーを停止（`fuser -k 3000/tcp`）。

補足: `tailscale serve`（HTTPS・きれいな URL）は tailnet 側で未有効。使うなら管理コンソールで一度有効化が必要（`tailscale serve` 実行時に案内 URL が出る）。有効化しない限りは上記の tailnet IP 直アクセス方式を使う。

## コード規約（Mac/サーバ含む全環境で遵守）

上記「開発フロー」に加え、環境を問わず守る規約。

- **ライブ配信物に生成ツールの痕跡を残さない**: 公開サイトが配信する HTML/CSS/JS（第三者が DevTools/View Source で見える成果物）に、AI/ツール由来の文字列・メタ・可視コメント・attribution を残さない。本番ビルドは minify でコメント除去されるため通常は自然に満たされる。**GitHub のソース/コミット履歴は対象外**（判断基準は「通常のサイトから見えるか」だけ）。
- **設計トークン体系（Mist Terminal）に沿う**: `src/app/(frontend)/mist.css` の `.mist` スコープに集約。タイポは6段 `--fs-caption/meta/ui/body/h3`（display は各要素の `clamp` を維持）、カラーは本文系10トークン `--m-ink / ink-2 / sub / faint / accent-ink / accent / line / hairline / surface / bg`（本文系は生 `oklch` を持たずトークン経由・装飾色は literal 例外）、余白は 4px 系。テキストトークンは `#eef1f4` 上で AA(4.5:1) 以上を維持。新規実装もゼロから色/寸法を足さずトークン経由で。
- **Fable 5 レビューを必須ゲートに**: 非自明な実装は Agent ツール（`model: fable`）でサブエージェント指名し設計/コードレビュー → Blocking/Should-fix を修正・再確認してから PR。

## 技術スタック

- **フレームワーク**: Next.js 15.3.2 with React 19.1.0
- **CMS**: PayloadCMS 3.38.0 with Neon DB
- **スタイリング**: TailwindCSS 4.1.7
- **UIライブラリ**: Framer Motion, React Icons, Lucide React
- **クラウドストレージ**: Cloudflare with S3 integration
- **データベース**: Neon DB
- **デプロイ**: Vercel

## 開発コマンド

```bash
# 開発
pnpm dev                    # 開発サーバー開始
pnpm devsafe               # クリーンスタート（.nextを削除）

# ビルド
pnpm build                 # プロダクション用ビルド
pnpm start                 # プロダクションサーバー開始

# リント & 型生成
pnpm lint                  # ESLint実行
pnpm generate:types        # PayloadCMS型生成

# PayloadCMS
pnpm payload               # PayloadCMS CLIコマンド
```

## プロジェクト構造

```
src/
├── app/
│   ├── (frontend)/           # 公開ページ
│   │   ├── timeline/         # タイムラインページ
│   │   ├── posts/           # ブログ投稿
│   │   ├── gallery/         # 画像ギャラリー
│   │   ├── profile/         # プロフィールページ
│   │   └── products/        # 製品紹介
│   ├── (payload)/           # PayloadCMS管理画面
│   └── api/                 # APIルート
├── collections/             # PayloadCMSコレクション
├── components/              # Reactコンポーネント
├── lib/                     # ユーティリティ関数
└── globals/                 # グローバル設定
```

## 主要機能

### 1. タイムラインシステム
- 画像サポート付きリッチテキストコンテンツ
- アニメーション付きハートエフェクトのいいね機能
- 楽観的UIによるリアルタイム更新
- 一括操作サポート

### 2. ブログ投稿
- リッチテキストエディタ（Lexical）
- 画像アップロードと管理
- SEOフレンドリーなURL

### 3. ギャラリー
- Cloudflare変換付き画像モーダル
- レスポンシブグリッドレイアウト
- サムネイルプレビュー

### 4. イベント管理
- アクティブイベント表示
- 日付フォーマットユーティリティ
- イベントカードコンポーネント

### 5. 製品紹介
- 画像付き製品カタログ
- リッチな説明文
- レスポンシブカード

## APIエンドポイント

### タイムライン
- `GET /api/timeline` - タイムライン投稿取得
- `POST /api/timeline` - タイムライン投稿作成
- `DELETE /api/timeline` - タイムライン投稿一括削除
- `POST /api/timeline/[id]/like` - タイムライン投稿にいいね

### その他のAPI
- `/api/products` - 製品管理
- `/api/letter` - コンタクトフォーム処理
- `/api/metadata` - URLメタデータ抽出

## データベースコレクション

1. **TimelinePosts** - リッチテキスト付きタイムラインエントリ
2. **BlogPosts** - ブログ記事（タイトル、本文、画像、URL、日付、カテゴリー）
3. **Events** - イベント管理（時刻、タイトル、説明、画像、プラットフォーム）
4. **Products** - 作成したプログラムのカタログ
5. **Letters** - コンタクトフォーム送信
6. **Media** - 画像ファイルアップロード
7. **Users** - 認証

## 環境変数

必要な環境変数：
```
DATABASE_URL=              # Vercel Postgres接続
PAYLOAD_SECRET=           # PayloadCMSシークレットキー
S3_ENDPOINT=              # Cloudflare R2エンドポイント
S3_ACCESS_KEY_ID=         # Cloudflareアクセスキー
S3_SECRET_ACCESS_KEY=     # Cloudflareシークレットキー
S3_BUCKET=                # Cloudflareバケット名
S3_REGION=                # Cloudflareリージョン
```

## コンポーネントガイドライン

### スタイリング
- TailwindCSSクラスを使用
- レスポンシブデザインパターンに従う
- グラスモーフィズム効果（`glass`クラス）を使用
- セクションパディング（`section-pad`）で一貫したスペーシング

### 状態管理
- ローカル状態にはReactフックを使用
- 楽観的UI更新を実装
- ローディング状態を適切に処理

### エラーハンドリング
- 常にtry-catchブロックを実装
- ユーザーフレンドリーなエラーメッセージを使用
- デバッグ用にエラーをログ出力

## 開発のヒント

### PayloadCMS
- 管理画面は`/admin`で利用可能
- スキーマ変更後は型生成：`pnpm generate:types`
- データベース操作にはPayloadCMSクライアントを使用

### 画像処理
- 画像はCloudflare R2に保存
- 変換には`toCFUrl()`ヘルパーを使用
- 適切なalt textとsizesを実装

### パフォーマンス
- Next.js Imageコンポーネントを使用
- 適切なローディング状態を実装
- データベースクエリを最適化

### テスト
- いいねボタンアニメーションのテスト
- レスポンシブレイアウトの確認
- APIエンドポイントのチェック

## よくある問題と解決方法

1. **いいねボタンが動作しない**
   - APIエンドポイントパス（`/api/timeline/[id]/like`）を確認
   - データベース接続を確認
   - try-catch内のエラーハンドリングを確認

2. **PayloadCMS管理画面エラー**
   - `PAYLOAD_SECRET`環境変数を確認
   - データベース接続を確認
   - スキーマ変更時は型を再生成

3. **画像アップロード問題**
   - S3/Cloudflare認証情報を確認
   - バケット権限を確認
   - 適切なCORS設定を確認

4. **ビルドエラー**
   - ビルド前に`pnpm generate:types`を実行
   - TypeScriptエラーを確認
   - すべての環境変数を確認

## 最近の更新

- いいねボタンAPIエンドポイントパスを修正
- いいね機能の楽観的UI更新を実装
- Xスタイルのハートアニメーションを追加
- PayloadCMS一括削除機能を修正
- コンポーネント全体のエラーハンドリングを強化

## デプロイメント

プロジェクトはVercelデプロイ用に設定済み：
- プッシュ時の自動ビルド
- 環境変数管理
- PostgreSQLデータベース統合
- 静的ファイル最適化

デプロイメント問題の場合、以下を確認：
1. 環境変数が設定されている
2. データベースマイグレーションが適用されている
3. ビルドコマンドがローカルで成功している

---
> Source: [Makoto041/portfolio](https://github.com/Makoto041/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
