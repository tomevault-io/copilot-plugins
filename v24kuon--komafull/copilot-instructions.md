## komafull

> ビルドなし（No-Build）環境における実装規約。Vite/Nodeを廃止し、HTMX/Alpine.js/標準CSSを利用する際の資産管理、読込、セキュリティ、UXの方針を定義する。


# ビルドなし（No-Build）実装規約

この規約は、Node.js/Vite等のビルドプロセスを廃止した環境において、保守性、パフォーマンス、セキュリティを担保するためのルールである。

## 1. アセットの配置と管理

### 1.1 配置場所 (`public/` 配下)
- ブラウザが直接取得する静的資産はすべて **`public/assets/`** 配下に集約する。
  - `public/assets/css/`: スタイルシート
  - `public/assets/js/`: JavaScriptファイル
  - `public/assets/img/`: 画像
  - `public/assets/vendor/`: CDNを使用しない場合の外部ライブラリ（HTMX/Alpine等）
- `resources/` 配下のアセット（`resources/css`, `resources/js`等）は原則として使用せず、配信もしない。

### 1.2 読み込み方法
- **`@vite` 指令の使用は全面禁止**とする。
- 代わりに、カスタムヘルパー `v_asset()` を使用してURLを生成する。
- レイアウト（`layouts/app.blade.php` 等）で一括読み込みを行い、ページ固有のものは `@stack` で管理する。

```blade
{{-- layouts/app.blade.php の例 --}}
<link rel="stylesheet" href="{{ v_asset('assets/css/app.css') }}">
@stack('styles')

<script defer src="{{ v_asset('assets/js/app.js') }}"></script>
@stack('scripts')
```

### 1.3 キャッシュバスティング
- ファイル名にハッシュを付与できないため、クエリストリングによる版数管理を行う。
- `v_asset()` ヘルパー内で `?v=YYYYMMDD_N` 形式のパラメータを付与する。
- 版数は `config/app.php` または `.env` の `ASSET_VERSION` を唯一の真実（Single Source of Truth）とする。

## 2. フロントエンド・ライブラリの利用ルール

### 2.1 共通原則
- **インライン `<script>` および `<style>` は原則禁止**とする（キャッシュ不可、CSP違反、差分追跡困難のため）。
- 外部ライブラリ（HTMX, Alpine.js等）のCDN利用時は、必ず **バージョン固定** を行う（`@latest` は禁止）。

### 2.2 HTMX (サーバー主導UI)
- **役割**: フォーム送信、部分更新、検索・ページングなどの「サーバー状態が正」のUIに限定する。
- **リクエスト制御**: `HX-Request` ヘッダーを確認し、HTMXリクエスト時は部分テンプレート（レイアウトなし）のみを返す。
- **CSRF対策**: `body` タグに `hx-headers` を付与し、全リクエストにトークンを含める。

```blade
<body hx-headers='{"X-CSRF-TOKEN":"{{ csrf_token() }}"}'>
```

### 2.3 Alpine.js (クライアント状態管理)
- **役割**: トグル、モーダル、タブ等の「UIの表示状態」など、サーバー通信を伴わない最小限の状態管理に限定する。
- **定義方法**: `x-data` にオブジェクトリテラルを直接書かず、外部JSファイルで `Alpine.data()` を使用して定義する。
- **HTMXとの共存**: HTMXで差し替えられる領域内にAlpineの状態を持たせることは避ける。

## 3. CSS設計方針（Tailwind CSS禁止）

- **命名規約**: BEM（Block Element Modifier）相当の命名規約を採用する（例: `.c-card`, `.c-card__title`）。
- **共通化**: デザイントークン（色、余白、フォントサイズ等）はCSS変数（Custom Properties）で一元管理する。
- **分割**: `app.css`（共通）と `pages/*.css`（ページ固有）の2層構造を基本とし、過度な分割を避ける。
- **社長変数ゾーン**: `app.css` の先頭に、非エンジニア（社長等）が調整しやすい変数定義セクションを設ける。

## 4. セキュリティと品質

### 4.1 SRI (Subresource Integrity)
- CDNから読み込む全ての資産には `integrity` および `crossorigin="anonymous"` を必須とする。
- SRIが提供されないCDNは利用せず、`public/assets/vendor/` に自己ホストする。

### 4.2 CSP (Content Security Policy)
- `script-src` は `'self'` および許可されたCDNドメインのみとする。
- `'unsafe-inline'` および `'unsafe-eval'` は原則として許可しない。

### 4.3 主要導線の不信任
- **予約・決済** などの主要導線は、JSが無効（またはCDN障害時）でも動作するよう、標準的なHTML/サーバーサイド処理で完結させる。JSはあくまでUX向上のための補助（Progressive Enhancement）として扱う。
- 金額、予約枠、ステータス遷移などのビジネスロジックは、フロントエンドを一切信用せず、すべてサーバー側で再検証する。

## 5. デプロイと整合性

- リリースおよびロールバックは、サーバーサイドコードと `public/assets/` 内の静的アセットを一つの原子単位（Atomic Unit）として扱う。
- 片方のみの更新や戻しによるキャッシュ不整合を避けるため、リリースごとに `ASSET_VERSION` を更新することを必須とする。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/v24kuon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
