## kakeibo-app

> iPhone/Android両対応の家計簿アプリ。家族で収支を共有し、SVGチャートで支出推移・カテゴリ分析を可視化する。

# 家計簿アプリ（kakeibo-app）

## プロジェクト概要
iPhone/Android両対応の家計簿アプリ。家族で収支を共有し、SVGチャートで支出推移・カテゴリ分析を可視化する。
デザインテーマは「藍染（Aizome）」— 深藍色（indigo）× 和紙クリーム（washi）× 真鍮（brass）の統一パレット。

## 技術スタック
- Frontend: React Native + Expo SDK 54 + TypeScript
- State管理: Zustand
- バックエンド/DB: Supabase (PostgreSQL + Auth + RLS + Edge Functions)
- AI: Claude API via Supabase Edge Function（APIキーはサーバー側で管理）
  - レシートOCR: claude-haiku-4-5-20251001
- チャート: react-native-svg（手描きSVGドーナツ・折れ線グラフ）
- カレンダー: react-native-calendars
- 通知: Expo Notifications（Expo Goでは無効化）
- 画像処理: expo-image-manipulator（リサイズ・圧縮）
- 課金: react-native-purchases（RevenueCat — プレミアム判定）
- 分析: posthog-react-native（画面遷移・イベント計測）
- 広告: react-native-google-mobile-ads（バナー + リワード実装済み。Expo Go ではリワードはモック表示）

## デザインテーマ: 藍染（Aizome）
テーマカラーは `src/theme/aizome.ts` で一元管理：
- `indigo: '#15243F'` — 主要背景・テキスト
- `indigo2: '#1f3358'` — 紺青
- `indigoSoft: '#384d75'` — タブ非活性など
- `washi: '#F1E8D3'` — 画面背景
- `washi2: '#E8DCC0'` — カード背景
- `ink: '#0E1729'` — 最暗色
- `text: '#15243F'` — テキスト（= indigo）
- `textSoft: '#5a6378'` — 補助テキスト
- `brass: '#C9A55C'` — アクセント・アクティブ状態
- `brassSoft: '#D9BC85'` — サブアクセント
- `rule: '#cdb98e'` — ボーダー
- `income: '#384d75'` — 収入表示色
- `expense: '#a44231'` — 支出表示色（赤褐色）
- `danger: '#c05050'` — 警告・削除

共通UIルール：
- ボタン: indigo背景 + brass文字
- カードUI: washi2背景 + rule罫線（影なし）
- アイコン: borderRadius: 8（角丸正方形）
- ヒーローカード: indigo背景 + AsanohaBg（麻の葉文様SVGオーバーレイ）

## ディレクトリ構成
```
src/
  screens/      # 画面コンポーネント（13画面 + MainNavigator + MainPlaceholder※未使用）
  components/   # 共通UI（ScopeSelector, CategoryIcon, AizomeCategoryIcons, MemberAvatar, AsanohaBg, AdBanner, RewardedAdModal）
  hooks/        # useTransactionFilter（ファイル名: useActiveGroupId.ts）
  stores/       # Zustand ストア（authStore, transactionStore, groupStore, viewStore）
  services/
    claudeApi.ts    # Claude API共通ヘルパー（Edge Function経由のみ）
    receiptOcr.ts   # レシート画像解析
    supabase.ts     # Supabaseクライアント
    transactions.ts # 取引CRUD + カテゴリ管理
    auth.ts         # 認証（匿名 + メール）
    family.ts       # グループ作成・参加・メンバー管理
    purchases.ts    # RevenueCat（IAP / プレミアム判定）
    analytics.ts    # PostHog（画面追跡・イベント計測）
    notification.ts # プッシュ通知
  theme/        # 藍染テーマ定義
  types/        # TypeScript型定義
  utils/        # format（日付・金額）, categoryMapping, categoryVisibility, transactionScope
supabase/
  migrations/   # SQLマイグレーション（0001〜0015）
  functions/
    claude-proxy/   # Claude API中継（認証・レート制限付き）
    delete-account/ # アカウント削除（関連データ一括削除）
    _shared/        # CORS設定など共通モジュール
  seeds/        # シードデータ
```

## 主なコマンド
- 起動: `npx expo start --lan --clear`（環境変数変更時は --clear 必須）
- Android: `npm run android`
- iOS: `npm run ios`
- 型チェック: `npm run type-check` または `npx tsc --noEmit`
- Lint: `npm run lint`
- Edge Function デプロイ: `npx supabase functions deploy claude-proxy --no-verify-jwt`
- Supabase Secrets 設定: `npx supabase secrets set ANTHROPIC_API_KEY=sk-ant-...`

## コーディング規約
- TypeScriptのanyは原則禁止。型は必ず明示する
- コンポーネントはfunctional componentのみ（classは不可）
- Supabaseクエリはservices/層にのみ書く。画面に直接書かない
- 金額はすべて円（整数）で扱い、表示時のみ円換算する（例: 1000 = ¥1,000）
- 日付はISO8601文字列で保持し、date-fnsで操作する
- 色は直接ハードコードせず `AI.*` テーマ定数を使う（`#F5F7FA`, `#4CAF50` 等の旧色は禁止）

## デバッグ記録の運用ルール
Claudeは作業中に遭遇したバグ・エラーとその解決過程を「過去のデバッグ記録」セクションに追記すること。
- **記録タイミング**: 原因特定に試行錯誤を要したバグ、または再発しやすいパターンを解決した直後
- **記録フォーマット**: `症状の要約 → 解決策`（1行で簡潔に）
- **作業開始時**: 必ず「過去のデバッグ記録」セクションを参照し、同じ問題を繰り返さない
- **蓄積の価値**: この記録はセッションをまたいで保持される。過去の自分が残した知見を信頼し、活用すること

## 重要な注意事項
- APIキーは Supabase Edge Function の環境変数（secrets）で管理する。クライアントに露出させない
- `EXPO_PUBLIC_` プレフィックスの環境変数はビルドに埋め込まれるため、秘密キーには絶対使わない
- Supabaseのanon keyはクライアントに公開可能だが、service roleキーは絶対に公開しない
- Row Level Security (RLS)は必ず有効にする
- Expo Goでは広告SDK / 通知は動作しない。Constants.executionEnvironment でスキップする

## 過去のデバッグ記録（繰り返し防止）
- `anthropic-dangerous-direct-browser-access` ヘッダーはEdge Function経由なら不要
- `CREATE TABLE IF NOT EXISTS` は既存テーブルがあると何もしない → スキーマ変更は DROP + CREATE が必要
- expo-image-manipulator の npm install で ERESOLVE エラー → `--legacy-peer-deps` で解決
- 環境変数やサービスファイルの変更後は `npx expo start --lan --clear` で再起動必須
- Expo Metro bundler のデフォルトポート 8081 が競合する場合は `-- --port 8082` で回避
- StyleSheet.create でキー名が重複すると後勝ちで上書きされる（例: `input` が2つ → 片方を `fieldInput` にリネーム）
- SVGドーナツで360°の arc は始点=終点となりパスが消える → `Math.min(span, 359.99)` でクランプ
- react-native-svg の strokeDasharray は配列 `[2, 2]` または文字列 `"2,2"` どちらでも可
- iOS の `<Modal>` 閉じ中に ImagePicker を起動すると開かない → Modal を閉じずにピッカーを起動し、完了後に閉じる
- posthog-react-native v4 が `@posthog/core/surveys` 等のサブパスを使う → metro.config.js に手動リゾルバーを追加（unstable_enablePackageExports=false のため）
- RevenueCat は Expo Go で動作しない → `Constants.executionEnvironment === 'storeClient'` でスキップ
- receiptOcr の max_tokens: 512 では商品数が多いレシートで JSON が途中切れ → 2048 に設定
- 新カラム追加マイグレーションは、それを使うクライアントより先にDBへ適用する。未適用だと PostgREST が「column ... does not exist」を返す（例: 0012 hidden_category_ids）
- Supabase の `update().eq()` は対象行が無くても error=null で 0 行更新になる（沈黙の失敗）→ 更新を保証したい箇所は `.select('id')` で affected 行を確認する
- Zustand の楽観更新で非同期保存する際、ハンドラ内で閉包の値を使うと全配列上書き時に競合する → `useStore.getState()` で最新値を読み、保存中は操作を直列化する
- レシートOCRの精度低下（金額誤読・読み取り失敗）→ 原因はJPEG圧縮率の上げすぎ（compress 0.5）。Claude Visionは~1.15MPに内部縮小するので解像度より圧縮ノイズが効く。品質0.8に上げると改善（トークン課金は画像寸法依存なので品質を上げてもコストはほぼ増えない）
- グループ表示で相手の記入者アイコンだけ「?」になる → 相手の users 行が取れていない（users_select_same_group RLS の適用漏れ or 相手がプロフィール未設定）。対策: RLSをバイパスする SECURITY DEFINER 関数 get_member_profiles(uuid[]) で同グループメンバー＋自分に限定して取得（0014）。関数未適用でも壊れないよう client は RPC 失敗時に直接クエリへフォールバック。名前・写真とも無い場合のフォールバックは「?」ではなく人型アイコン👤
- iOSで入力欄がキーボードに隠れて見えない → 素のScrollViewはキーボードを自動回避しない。ScrollViewに `automaticallyAdjustKeyboardInsets`（iOS向け・Androidは無害、RN0.71+）を付けるか、KeyboardAvoidingViewで包む。Androidは app.json の softwareKeyboardLayoutMode 既定（resize）で概ねOK。新規に入力欄を追加する画面では必ずどちらかを適用する
- 絵文字が白黒の線画（テキスト表示）で「変な」見た目になる → そのコードポイントが Emoji_Presentation=No（例: 👁 EYE U+1F441, ❤ U+2764, ✏ U+270F 等）。末尾に異体字セレクタ U+FE0F（`️`）を付けてカラー絵文字を強制する（`'👁'`→`'👁️'`）
- react-native-svg で `<Svg width="100%" viewBox=...>` だけ（height省略）だと高さが 0 に潰れてグラフが描画されない → 親Viewの `onLayout` で実幅を測り Svg に width/height を数値で渡す。固定座標系なら viewBox はそのまま `height={実幅*H/W}`、可変なら width=実幅・viewBox幅も実幅にして1:1。ReportScreen の折れ線・棒グラフ両方がこれで非表示になっていた
- 画面のデータ取得範囲(from/to)と集計側のフィルタ範囲のズレに注意: 週は月末をまたぎ「今週」は今日基準なので、取得toを月末で切ると実データがあるのに¥0/データなしになる → toは「月末を含む週の末尾」と「今日を含む週の末尾」の遅い方まで広げる（ReportScreen）
- 前月同日比較で累計配列を daysElapsed でインデックスすると前月が短い月で範囲外→`?? 0`で当月全額が前月比になる → `Math.min(daysElapsed, 配列長)` でクランプ
- 電卓UIで演算子直後に「=」を押すと第2オペランドが0扱いになり ×0=0 で入力金額が消える → 未入力('0')のままの確定は演算を取り消して accumulator を採用する
- 適用済みマイグレーションのin-place編集は適用済みDBには再実行されず届かない（schema_migrationsに記録済みのため）→ 修正は必ず新規マイグレーションとして追加する（例: 0013のpgcrypto修飾修正を0016として切り出し）
- クライアント側のfetchタイムアウト（AbortController）はサーバ側のコミットを取り消せない → DB書き込みに短いタイムアウトを掛けるとエラー表示後のリトライで二重登録になる。abortは読み取り専用とし、書き込みはストール保険の長いタイムアウトのみにする
- PostgRESTで存在しない列に `.eq()` すると42703エラー（例: budgets.user_id は存在しない— budgetsはgroup_id/category_idのみ。連鎖削除はFKのON DELETE CASCADEに任せる）
- プロフィール取得失敗時にuser=nullでAuthScreenへ落とすと、「登録なしで始める」タップで signInAnonymously() が新規アカウントを作り既存データが永久に孤立する → セッションが有効なら最小プロフィールでアプリへ進め、signInAnonymously自体にも既存セッション再利用ガードを入れる
- レシートOCRで「お預かり金額」を合計と誤判定 → 否定指示だけでは不十分。プロンプトに具体例（「合計1,580/お預かり2,000/お釣り420 → totalは1580」）を入れ、「金額の隣のラベルを読み『合計』ラベルの額だけ採用」と明示すると確実に改善。「最終合計」という表現は最下部のお預かり/お釣りを拾う誘因になるので使わない。あわせて未使用だった rawText（画像テキスト全文）を出力JSONから削除し出力トークンを節約（精度向上とコスト削減を同時に達成）
- 列DEFAULTのスキーマ修飾を `information_schema.columns.column_default` で判定してはいけない → 保存された式は**現在の search_path を基準に逆生成**されるため、`extensions` が search_path にあると `extensions.gen_random_bytes(...)` が `gen_random_bytes(...)` と修飾なしで表示される。さらに列DEFAULTはテキストではなく**関数OID解決済みのパースツリー**として保存されるのでDDLの修飾は保存時点で消える。ALTERがエラーなく通れば適用は確定。決定的に確認するなら `pg_attrdef` → `pg_depend` → `pg_proc` → `pg_namespace` を辿って参照先関数のスキーマを見る
- `npx supabase migration list` で全マイグレーションが `remote: ""` になる → スキーマをダッシュボードのSQLエディタから手動適用してきた場合、`supabase_migrations.schema_migrations` が空のままになる。この状態で `supabase db push` すると**0001から全部流そうとして DROP/CREATE で本番データを壊す**。手動運用を続ける限り push は禁止。適用状況の判断も migration list ではなくスキーマの直接確認で行う

## 認証フロー
- 初回起動: AuthScreen を表示（ログイン / 新規登録 / 「登録なしで始める」の3択）
- ゲスト開始: 「登録なしで始める」で匿名サインイン（Supabase Anonymous Auth）→ すぐに使い始められる
- メール登録: AuthScreen の新規登録、または ProfileScreen から後付け登録（`linkEmail()` でアカウント昇格）
- データ引き継ぎ: 匿名→メール登録後もデータは維持される

## 設計メモ

### ナビゲーション（5タブ + スタック画面）
タブは漢字アイコンで表示（KanjiIcon コンポーネント）：
1. 家（ホーム） — HomeScreen
2. 暦（暦） — CalendarScreen
3. 記（記入） — AddTransactionScreen（真鍮アクセント、中央に大きめ配置）
4. 歴（履歴） — TransactionListScreen
5. 析（分析） — ReportScreen

スタック画面: ReceiptScan, ReceiptConfirm, Family, CategoryManage, EditTransaction（AddTransactionScreen を編集モードで再利用）, Menu, Profile, Premium

未ログイン時（user==null）は AuthScreen を表示。それ以外は MainNavigator。

### 入力・表示スコープ
- 入力は常に個人（user_id）に紐付ける。AddTransactionScreenにグループ選択なし
- 表示スコープ（個人/グループ）は stores/viewStore.ts の selectedScope で管理
- ScopeSelector コンポーネントはホーム・履歴・カレンダー・レポートで共通利用

### カテゴリ管理
- デフォルトカテゴリ（is_default=true）は名前・削除の変更不可（全ユーザー共通レコードのため）。ただし各ユーザーが記入画面で「非表示」にできる（users.hidden_category_ids 配列に保持、0012マイグレーション）。CategoryManageScreen でタップ切替、AddTransactionScreen の候補から除外
- カスタムカテゴリ（is_default=false, user_id付き）はユーザーが作成・編集・削除可能
- アイコンは絵文字 or 写真（data:image/jpeg;base64形式、80x80にリサイズ）
- CategoryIcon コンポーネントで絵文字/画像/和の線画を自動判別して表示
- 藍染カテゴリアイコン（AizomeCategoryIcons.tsx）: デフォルトカテゴリ名（食費/外食/交通費/医療/サブスク/その他/娯楽/日用品/衣類/税金/給与/副業/お年玉）に一致すると、絵文字の代わりに indigo+brass の和の家紋風 react-native-svg 線画を描画（claude.ai/design「カテゴリの紋」由来）。色はカテゴリ色に依存せず固定。写真アイコンは常に優先（線画で上書きしない）。線画表示時はアイコン台座を明るいクリーム `AI.chip` にする（indigo 線が暗背景で消えるのを防ぐ）
- デフォルトカテゴリ刷新（0015）: 支出に「サブスク」「税金」、収入に「お年玉」を追加。収入の汎用「その他収入」は削除（既存取引があれば「給与」へ付け替えてから削除。FK が ON DELETE CASCADE でないため付け替え必須）。収入デフォルトは 給与/副業/お年玉 の3種

### 課金（RevenueCat）
- authStore.isPremium でプレミアム状態を管理
- プレミアムユーザーには広告を非表示（AdBanner）、レシートスキャン前のリワード広告もスキップ
- API Key: iOS は実キー設定済み。Android はプレースホルダー（`goog_xxx`）— ストア申請前に差し替え
- プレースホルダーキー / Expo Go 環境では configure せず非プレミアムへ安全にフォールバック。purchase/restore は明示エラーを投げる
- 月額価格は `getMonthlyPriceString()` でストア設定値（priceString）を取得して表示（取得不可時は ¥480 フォールバック）
- 動作確認の前提: RevenueCat ダッシュボードに Entitlement `premium` と Offerings（default → monthly）、ストアに月額商品の登録が必要。実購入テストは TestFlight + Sandbox のみ（Expo Go 不可）
- 課金画面（PremiumScreen）はストアの審査要件（App Store Guideline 3.1.2 / Google Play）を満たす必要がある。プラン名・期間・価格・自動更新の開示文・購入復元・利用規約（EULA）/プライバシーポリシーへのリンクを消さないこと。URLは `src/utils/links.ts` で一元管理（MenuScreen と共用）

### 広告（AdMob）
- バナー（AdBanner）: ホーム・履歴・分析に配置。プレミアム時は非表示
- リワード（RewardedAdModal）: レシートスキャン前に表示。報酬獲得前に閉じたらスキャンさせない。ロード失敗（在庫なし等）はブロックせずスキャンへ進める。Expo Go では5秒カウントダウンのモック
- 本番広告ユニットIDを使うのは「production チャンネルのリリースビルド」のみ。それ以外（__DEV__ / staging / Expo Go）はテストID — テスターの閲覧・タップによる無効トラフィック（アカウント停止）防止のフェイルセーフ
- 初期化順序: ATT許諾（iOS）→ mobileAds().initialize()（App.tsx）

### 分析（PostHog）
- PostHogProvider は使わない（iOS Modal競合のため）。posthog クライアントを直接利用
- services/analytics.ts で型付きイベント関数を一元管理
- App.tsx の NavigationContainer onStateChange で画面遷移を手動追跡
- `EXPO_PUBLIC_POSTHOG_KEY` で制御（未設定時は disabled）

## Feature: 分析チャート（ReportScreen）

### カテゴリ分析タブ
- 期間タブ: 今週 / 今月 / 3ヶ月
- SVGドーナツチャート: arcPath()ヘルパー、rOuter/rInner で太さ制御
- パレット: brass系7色 `[AI.brass, '#D9BC85', '#A8845A', '#8a9d6a', '#7e8aa3', '#5a6b87', '#384d75']`
- ランキングリスト: 連番 + 色付き正方形 + カテゴリ名 + 金額 + パーセンテージ

### 支出推移タブ
- 月切り替え（← →）
- 累計折れ線チャート: 今月（brass実線 + 面積塗り）vs 前月（半透明破線）
- 統計行: 日平均 / 最大日 / 記録日
- ペース予測カード: 月末予測金額を表示

### 技術詳細
- react-native-svg で手描き（ライブラリ依存なし）
- viewBox="0 0 264 130" + width="100%" でレスポンシブ
- buildDailyExpense() で日別支出配列を構築 → 累積和で折れ線データ化

## Feature: OCR（レシート解析）
- 解析エンジン: Claude Vision API（claude-haiku-4-5-20251001）
- 呼び出し経路: receiptOcr.ts → callClaudeViaEdge() → Supabase Edge Function
- 画像前処理: 1000px幅にリサイズ、JPEG品質0.8。Claude Visionは~1.15MPに内部縮小するため解像度はこの程度で十分。圧縮率を上げすぎる（品質0.5等）と桁・小数点が潰れて誤読・読み取り失敗の原因になる。JPEG品質はトークン課金（画像寸法依存）にほぼ影響しないため品質優先で問題ない
- 全自動保存は禁止。必ず ReceiptConfirmScreen を経由させる
- 解析結果のJSONパース失敗時はnullを返し、アプリをクラッシュさせない
- 画像はBase64でEdge Functionに送信。ファイル保存は行わない（プライバシー）

## 公開準備ステータス
実装済み: Edge Function でのAPIキー管理 / 藍染テーマ全画面適用 / 匿名→メールのオンボーディング / PostHog分析 / RevenueCat課金（iOSキー設定済み・Androidはプレースホルダー）/ アプリ識別子（com.moriyaryoga.kakeibo）/ アイコン・スプラッシュ（generate_icons.py 生成）/ AdBanner（AdMob接続）/ リワード広告（実広告 + Expo Goモックフォールバック）/ プライバシーポリシー・利用規約（docs/ を GitHub Pages で公開、アプリ内リンク済み）/ Apple Developer Account・App Store Connect アプリレコード（ascAppId は eas.json）/ 課金画面のストア審査要件対応

残タスク（iOS先行リリース想定）:
- [ ] App Store Connect に月額サブスク商品を作成し、RevenueCat（Entitlement `premium` / Offerings default→monthly）と紐付け
- [ ] TestFlight + Sandbox で購入・復元・アカウント削除を実機確認
- [ ] production ビルドで本番広告（バナー / リワード）の表示確認、AdMob アカウントの支払い情報
- [ ] Anthropic Console で API 利用額の上限・アラート設定（claude-proxy のレート制限はインメモリで1ユーザー10回/分のみ）
- [ ] App Privacy（プライバシーラベル）入力・スクリーンショット等の掲載情報
- [ ] EAS Build → `eas submit` → 審査提出
- [ ] Android対応（後回し可）: RevenueCat Android APIキー / Play Console 登録 / Data safety フォーム / app.json の不要権限 RECORD_AUDIO 削除

---
> Source: [kopan0126/kakeibo_app](https://github.com/kopan0126/kakeibo_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
