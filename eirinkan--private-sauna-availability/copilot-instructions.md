## private-sauna-availability

> - **プロジェクト名:** private-sauna-availability

# プロジェクト固有ルール

## Google Cloudプロジェクト
- **プロジェクト名:** private-sauna-availability
- **プロジェクトID:** gen-lang-client-0618658411
- **用途:** サウナアプリ（Cloud Run）

## 共通ルール

### 部屋名の重複禁止
- 同じ施設内で部屋名が重複しないようにすること
- 同じ部屋の異なるプラン（午前/午後/ナイト等）は、グループ化して表示するか、プラン名で区別すること
- データ取得時に部屋名の一意性を確認すること

### 表示必須項目（部屋ごと）
各部屋には以下の情報を必ず表示すること:
1. **部屋名** - 一意であること
2. **空き時間帯** - 利用可能な時間枠
3. **人数** - 定員（例: 2名、4名）
4. **分数** - 利用時間（例: 90分、120分）
5. **価格** - 料金（例: ¥6,600〜）

### 必須リンク（施設ごと）
各施設には以下のリンクを必ず設定すること:
1. **公式ホームページ** (`hpUrl`) - 施設の公式サイト
2. **Googleマップ** (`mapUrl`) - 店舗名で検索したGoogleマップリンク
   - 形式: `https://www.google.com/maps/search/?api=1&query=店舗名+地域名`
3. **予約ページ** (`url`) - 予約サイトへの直接リンク

これにより、地域や店舗が増えても統一された形式で対応可能

### ブラウザ自動化ツールの使い分け
データ検証やサイト確認時のブラウザツール選択:

1. **Claude in Chrome を優先使用**
   - Cloudflareで保護されたサイト（reserva.be等）の確認
   - ユーザーのブラウザセッションを使うため、Cloudflareをバイパスできる
   - 認証済みサイトの操作
   - 実際のブラウザ画面を見ながらのデバッグ

2. **Puppeteer MCP を使用する場面**
   - localhost/ローカル開発サーバーのテスト
   - Cloudflare保護のないサイト
   - ヘッドレスでの自動処理

3. **判断基準**
   - Cloudflare保護あり → **Claude in Chrome**
   - Cloudflare保護なし → Puppeteer MCP（軽量・安定）

### デプロイ方法
- **mainブランチへのコミット = 自動デプロイ**
- GitHubとCloud Buildが連携しており、pushすると自動的にCloud Runにデプロイされる
- デプロイ完了まで約3〜5分
- デプロイ後、`/api/refresh`でスクレイピングを再実行すると新コードが反映される
- 本番URL: `https://private-sauna-availability-526007709848.asia-northeast1.run.app`

### ナイトパックの表示形式
日をまたぐナイトパックの時間帯は、翌日の日付を**先頭**に付与する:
- 形式: `M/D HH:MM〜HH:MM`
- 例: `1/14 01:00〜08:30`
- 全施設で統一すること（GIRAFFE形式）

---

## 脈 MYAKU (spot-ly.jp) のスクレイピング

### 重要なルール

1. **カレンダーの◯✕マークは使用禁止**
   - 概要カレンダーの◯✕は日単位の空き状況のみで、具体的な時間帯がわからない
   - 必ずモーダルを開いて時間帯ボタンから取得すること

2. **ボタンのインデックスで直接指定**
   - ページ上のボタン順序は固定（0〜6）
   - PLANSのpageIndexでボタンを直接特定する

### スクレイピング手順

1. URLに日付パラメータを付与してアクセス
   ```
   https://spot-ly.jp/ja/hotels/176?checkinDatetime=YYYY-MM-DD+00%3A00%3A00&checkoutDatetime=YYYY-MM-DD+00%3A00%3A00
   ```

2. 各プランの「予約する」ボタン（`button.bg-black`）をクリックしてモーダルを開く

3. モーダル内で:
   - プラン名を取得してマッチング
   - 日付を選択
   - 時間帯ボタンの`disabled`属性で空き判定

### 空き判定方法

- **空いている時間帯**: `<button>` に `disabled` 属性がない
- **埋まっている時間帯**: `<button disabled="">`

```html
<!-- 空いている -->
<button><span>13:00</span><span>-</span><span>14:30</span></button>

<!-- 埋まっている -->
<button disabled=""><span>21:00</span><span>-</span><span>22:30</span></button>
```

### 注意事項

- **ナイトパック**: 日をまたぐため、カレンダー上の日付は翌日の予約を意味する
  - 例: カレンダーで1/14の◯は、1/13夜〜1/14朝のナイトパックが空いていることを示す
- モーダルのクリーンアップ: 次のプランを開く前に既存モーダルを閉じる

---

## GIRAFFE (reserva.be) のスクレイピング

### 重要なルール

1. **時間文字列の分割は正規表現で**
   - RESERVAのtimebox要素は`data-time`に時間を持つ（例: `09:40～11:40`）
   - `～`（全角チルダ U+FF5E）と`〜`（波ダッシュ U+301C）の両方が使われる可能性がある
   - 分割には正規表現 `/[～〜]/` を使用すること

2. **ページ再アクセス禁止**
   - 複数日のデータを取得する際、ページを再アクセス（`page.goto()`）しない
   - 同じページ内で日付をクリックして切り替えること
   - 再アクセスするとCloudflareチェックがトリガーされ、本番環境で失敗する

### スクレイピング手順

1. FlareSolverrでCloudflare Cookieを取得
2. 各部屋のURLにアクセス（1部屋1URL）
3. `input.timebox[data-vacancy="1"]`要素から空き枠を抽出
4. `data-targetgroup`（日付）と`data-time`（時間）を使用

### データ抽出コード

```javascript
// 正しい分割方法
const timeParts = time.split(/[～〜]/);
const timeRange = timeParts[0].replace(/^0/, '') + '〜' + timeParts[1].replace(/^0/, '');
```

---

## サウナヨーガン (reserva.be) のスクレイピング

### 重要なルール

1. **ページ再アクセス禁止（最重要）**
   - 7日分のデータを取得する際、2日目以降でページを再アクセスしない
   - 1回のアクセスで、日付ラベルをクリックして全日程を取得すること
   - 再アクセスすると本番環境でCloudflareに弾かれる

### スクレイピング手順

1. FlareSolverrでCloudflare Cookieを取得
2. 予約ページに1回だけアクセス
3. 各日付の`label[for="YYYY-MM-DD"]`をクリック
4. `input.timebox[data-vacancy="1"]`から空き枠を抽出
5. 次の日付をクリック（ページ再アクセスしない）

---

## サイト別メモ

### spot-ly.jp (脈)
- 認証: **ログイン不要**
- **ブラウザ不要**: 予約ページが内部で使うJSON APIを直接呼ぶ（2026-07-07に実証）
  - 部屋・プラン一覧: `https://api.spot-ly.jp/api/v2/spotly/hotels/176/room_types?checkinDatetime=X+00:00:00&checkoutDatetime=Y+00:00:00&roomTypeCategory=`
    - checkin < checkout が必須（同一日は422エラー）
  - 空き時間帯: `https://api.spot-ly.jp/api/v2/spotly/room_types/{部屋ID}/fixed_plans/{プランID}/available_times?startDate=YYYY-MM-DD&endDate=YYYY-MM-DD`
- 必要ヘッダーは `User-Agent` と `Accept: application/json` のみ（Cookie・認証・FlareSolverr不要）
- 時刻はUTCで返るため、JSTへ+9時間変換が必要
- 空き判定: `isAvailable: true`
- 旧方式（Puppeteer + FlareSolverr + react-select操作）は本番でreact-select描画待ちの失敗、spot-ly側の504エラー、プラン並び順のインデックス依存など複数の脆弱性があり、間欠失敗を起こしていたため使用禁止

#### ナイトパックの日付表示
- APIは深夜開始（例: `2026-07-08T00:30:00 JST`）で「7/7の夜」の枠として返す
- アプリ上は前日（=入店する夜の日付）にまとめ、時間帯を「翌M/D H:MM〜H:MM」形式で表示

### reserva.be (GIRAFFE, サウナヨーガン)
- 認証: **ログイン不要**
- Cloudflare保護あり → FlareSolverrでCookie取得必須
- 空き判定: `input.timebox[data-vacancy="1"]`
- 時間形式: `data-time="09:40～11:40"` → 全角チルダと波ダッシュの両方に対応
- **重要**: ページ再アクセス禁止（Cloudflareトリガー回避）

#### ⚠️ 継続的な観察が必要（2026-01-26更新）

時間経過でスクレイピングが失敗することがある。

**症状**: 全日程で空き枠0件になる（間欠的に発生）

**根本原因（2026-01-26特定）**:
1. **Cloud Runタイムアウト不足**: 540秒（9分）→ 8施設の処理に不足
2. **スクレイピング順序**: RESERVA系が後半にあるとタイムアウトの影響を受けやすい

**対策済み**:
| 日付 | コミット | 対策 |
|------|---------|------|
| 2026-01-22 | 10b8877 | Cookieキャッシュ90分TTL、チャレンジ検出時キャッシュ無効化 |
| 2026-01-26 | a4635f4 | Cloud Runタイムアウト900秒に延長、RESERVA系を最初に処理 |

**観察結果**:
- 2026-01-26 02:30頃: 修正デプロイ → 経過観察中

**スクレイピング順序（修正後）**:
1. GIRAFFE南天神（RESERVA）← 重点監視
2. GIRAFFE天神（RESERVA）← 重点監視
3. サウナヨーガン（RESERVA）← 重点監視
4. 脈 MYAKU（spot-ly）← 重点監視
5. KUDOCHI（hacomono）
6. SAKURADO
7. SAUNA OOO（gflow）
8. BASE（Coubic）

**確認コマンド**:
```bash
curl -s "本番URL/api/availability?date=$(date +%Y-%m-%d)" | jq '[.facilities[] | select(.name | contains("GIRAFFE") or contains("ヨーガン"))] | .[] | {name, slots: [.rooms[].availableSlots | length] | add}'
```

**今後の検討**: 並列実行によるさらなる高速化

---

## 実装ルール（地域拡大対応）

地域や店舗を追加する際は、以下のルールに従うこと。

### データ構造

#### スクレイパーの返り値（必須形式）
```javascript
{
  dates: {
    'YYYY-MM-DD': {
      '部屋名（時間/定員N名）¥価格': ['HH:MM〜HH:MM', ...]
    }
  }
}
```

#### 日付フォーマット
- 形式: `YYYY-MM-DD`（ISO 8601）
- 例: `2026-01-25`

#### 時間フォーマット
- 形式: `HH:MM〜HH:MM`
- 区切り文字: `〜`（波ダッシュ U+301C）を使用
- 注意: 全角チルダ `～`（U+FF5E）は使用しない（内部で統一）
- 例: `10:00〜12:00`, `9:40〜11:40`

#### 部屋名フォーマット
```
部屋名（時間/定員N名）¥価格
```

| パターン | 形式 | 例 |
|---------|------|-----|
| 固定価格 | `¥X,XXX` | `Silk（90分/定員2名）¥6,000` |
| 価格幅あり | `¥X,XXX-X,XXX` | `「陽」光の陽彩（120分/定員7名）¥6,600-11,000` |
| 平日/週末 | `¥X,XXX-X,XXX` | `プライベートサウナ（150分/定員3名）¥9,900-13,200` |

### スクレイパー実装ルール

#### 関数シグネチャ
```javascript
async function scrape(browser) {
  const page = await browser.newPage();
  try {
    const result = { dates: {} };
    // スクレイピング処理
    return result;
  } finally {
    await page.close();
  }
}
```

#### 必須設定
1. **User-Agent**: 最新のChrome/Safariを模倣
2. **Viewport**: `{ width: 1280, height: 800 }` 以上
3. **タイムアウト**: 60秒以上（Cloudflare待機用）
4. **タイムゾーン**: `page.emulateTimezone('Asia/Tokyo')` を設定

#### ボット検知対策
- `puppeteer-extra-plugin-stealth` を使用
- `navigator.webdriver` を偽装
- 適切な待機時間を設ける（2〜5秒）

### 施設登録ルール

#### scraper.js への登録

1. スクレイパーをimport
```javascript
const newsite = require('./sites/newsite');
```

2. scrapeAll() に追加
```javascript
console.log('  - 新店舗 スクレイピング中...');
try {
  data.facilities.newStore = await scrapeWithMonitoring('newStore', newsite.scrape, browser);
} catch (e) {
  console.error('    新店舗 エラー:', e.message);
  data.facilities.newStore = { error: e.message };
}
```

3. facilityInfo に追加（必須フィールド）
```javascript
{
  key: 'newStore',           // data.facilities のキー（キャメルケース）
  name: '店舗名',             // 表示名
  url: 'https://...',        // 予約ページURL
  hpUrl: 'https://...',      // 公式サイトURL
  mapUrl: 'https://www.google.com/maps/search/?api=1&query=店舗名+地域名'
}
```

#### 表示順序
- ユーザーの利用頻度・重要度に応じて決定
- 新規施設は既存施設の後ろに追加
- 地域別にグルーピングする場合は地域ごとにまとめる

### 予約システム別テンプレート

#### RESERVA系 (`reserva.be`)
- Cloudflare保護あり → FlareSolverr必須
- 空き判定: `input.timebox[data-vacancy="1"]`
- 時間分割: `/[～〜]/` で分割（両方の文字に対応）
- **重要**: ページ再アクセス禁止（Cloudflareトリガー回避）

#### hacomono系 (`xxx.hacomono.jp`)
- Cloudflare保護なし
- カレンダーのDOM構造がシンプル
- 空き判定: 要素の色やクラスで判定

#### STORES予約系（旧Coubic） (`xxx.stores.jp/reserve/`)
- `coubic.com` のURLは `xxx.stores.jp/reserve/` にリダイレクトされる（2026年に移行）
- **ブラウザ不要**: 予約ページが内部で使うJSON APIを直接呼ぶ（BASEで2026-07-07に実証）
  - コース一覧: `/reserve/api/reservation_flow/merchants/{merchant}/course_scheme/resources/{id}/courses`
  - 空き状況: `.../courses/{canonical_id}/availability/board_dates`（selected_date省略で今日から7日分）
- **重要**: board_datesのIDはコース一覧の `id` ではなく `canonical_id`（`id`だと404）
- 必要ヘッダーは `User-Agent` と `Accept: application/json` のみ（Cookie・認証不要）
- 空き判定: `dates[].availabilities[].is_available`（時刻はJSTオフセット付きで返る＝変換不要）
- 祝日判定はAPI側に任せる（平日プランは土日祝が全枠false、逆も同様→両プラン統合でよい）
- 旧方式（Puppeteerでボタンクリック→画面遷移）は本番で間欠失敗したため使用禁止

#### gflow系 (`sw.gflow.cloud`)
- Vue.jsベースのリアクティブUI
- ラジオボタンのクリックは複数手法で試行
- iframeでカレンダー表示

#### spot-ly系 (`spot-ly.jp`)
- タイムゾーン設定必須: `page.emulateTimezone('Asia/Tokyo')`
- 正規表現は `[0-9]` を使用（`\d`はCloud Runで動作しない場合あり）
- モーダル内の`disabled`属性で空き判定

### 地域拡大時の注意点

1. **命名規則**
   - 施設キー: キャメルケース（例: `giraffeTenjin`）
   - 同一ブランド複数店舗: `ブランド名+地域名`（例: `giraffeMiamitenjin`, `giraffeTenjin`）

2. **地域別ファイル構成**（将来的な拡張）
   - 現在: `src/sites/` に予約システム別
   - 拡張案: `src/sites/fukuoka/`, `src/sites/tokyo/` など地域別も検討

3. **料金データ**
   - `public/index.html` の `guestPricing` に追加
   - 平日/週末、追加人数、夜間料金などのパターンに対応

### 新店舗追加チェックリスト

- [ ] 予約システムを特定した
- [ ] スクレイパーを作成/修正した（返り値形式を確認）
- [ ] `scraper.js` にimportと実行コードを追加した
- [ ] `scraper.js` の `facilityInfo` に店舗情報を追加した（url, hpUrl, mapUrl）
- [ ] `public/index.html` の `guestPricing` に料金を追加した
- [ ] 部屋名フォーマットが統一されている（時間/定員/価格）
- [ ] 時間フォーマットが統一されている（`〜`使用）
- [ ] ローカルで動作確認した
- [ ] コミット＆プッシュした

---
> Source: [eirinkan/private-sauna-availability](https://github.com/eirinkan/private-sauna-availability) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
