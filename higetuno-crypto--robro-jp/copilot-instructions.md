## robro-jp

> 1. [このドキュメントの使い方](#このドキュメントの使い方)

# Roblox Japan Ranking (robro.fortunep.net — 予定 / `roblo.fortunep.net` は商標リスク回避のため却下)

## 目次

1. [このドキュメントの使い方](#このドキュメントの使い方)
2. [プロジェクト概要](#プロジェクト概要)
3. [ビジョン](#ビジョン設計判断の基準)
4. [UI設計原則](#ui設計原則最優先--迷ったらここに戻る)
5. [運営コンプライアンス](#運営コンプライアンスroblox規約遵守--絶対守る)
6. [技術スタック](#技術スタック確定事項--逸脱しない)
7. [ディレクトリ構造](#ディレクトリ構造)
8. [DBスキーマ](#dbスキーマsupabase--postgresql)
9. [拡張設計ガイドライン](#拡張設計ガイドライン絶対原則)
10. [フェーズ分割](#フェーズ分割必ず順番に実装)
11. [コーディング規約](#コーディング規約)
12. [外部API注意事項](#外部api使用上の注意)
13. [やらないこと](#やらないことスコープ外)
14. [質問・確認ルール](#質問確認)
15. [現在のフェーズ](#現在のフェーズ)

---

## このドキュメントの使い方

- **タスク開始時**：目次から該当セクションに飛ぶ。全文読み直しは不要
- **迷った時**：UI設計原則 と 拡張設計ガイドライン を最優先で参照
- **スキーマ変更時**：必ず確認を取る（勝手に進めない）
- **新機能追加時**：拡張設計ガイドラインに沿っているか検証
- **SEO目的のサイト編集（メタタグ・タイトル・description・JSON-LD・llms.txt・OG画像・sitemap 等）は事前確認不要・事後報告でOK**（2026-05-24 Yuki 指示）。ただし Roblox 規約（運営コンプライアンス章）は厳守

## 関連リポジトリ

- **`higetuno-crypto/robro-jp-analytics`**（private）— SEO/GSC 週次自動収集とレポート生成。ローカル：`C:\Users\higes\claudecode\robro-jp-analytics\`
  - 「robro-jp の今週のSEOレポート作って」で `reports/YYYY-MM-DD.md` + `.html` を生成
  - 既知の SEO 課題は `robro-jp-analytics/known-issues.md` で管理（robro-jp 本体を直したら該当 KI を削除）
  - 仕組みは taisyoku（失業給付情報サイト）の同等システムの移植版

---

## プロジェクト概要

日本語圏向けのRobloxゲーム発見プラットフォーム。Roblox公式検索では埋もれがちな個人開発ゲームを含め、リアルタイムのCCU（同時接続数）ランキングを提供する。将来的には個人開発者の登録・プロモーション機能、宣伝ポイント経済を構築予定。

**MVPのゴール**：プレイヤー向けランキングサイトを最速でローンチし、トラフィックを蓄積する。

---

## ビジョン（設計判断の基準）

- **北極星（最終判断基準）**：「**僕は優秀なクリエイターが真っ当に評価されればそれでいいんだ**」（Yuki, 2026-04-25）。機能の採否で迷ったら「これは優秀なクリエイターを真っ当に評価することに寄与するか？」で判定する。詳細は `higesakusei/idea-evaluation-v3.md` §0
- **面白さの通訳（コア差別化）**：他サイト（Rolimons、RoMonitor、robloxgame.jp）は「人気ゲームの数値比較」で勝負している。robro-jp は**数字の翻訳ではなく、"何が面白いか・誰向けか・英語不要か・配信向きか"を日本語で翻訳するサイト**としてポジションを取る。ランキングサイトではなく「日本人向け Roblox 発見サイト」。
- **日本語ファースト**：UI・デフォルト並び順・コピーライティング全てを日本語ユーザー中心に据える。公式Chartsの `country=JP` フィルタは認証必須で匿名APIからは使えないため、**`is_japanese` 判定を一次シグナル**として「日本で人気」を構築
- **運営解説 + ユーザー集合知のハイブリッド**：運営が日本語一言ピッチを与え、その上に**ユーザー投票タグ**（選択式、自由入力は初期は不可）を重ねて意味付け。ニコニコのタグ文化を参考
- **配信者ファースト導線**：配信者／インフルエンサーが「紹介しやすい」「企画が作れる」「OG画像が映える」状態を作れば、外部SNSからの流入ハブ（ro-bro は紹介される側ではなく、紹介に使われる側）になれる
- **軽量・高速**：初回表示1秒以内。モバイル最優先
- **誠実な数字**：Rolimonsのコピーではなく、自前で時系列を貯めて独自の「急上昇」を出す
- **将来のJP実データ取得**：認証cookie運用（使い捨てアカウント）や、ブラウザヘッドレスでの取得は将来検討
- **拡張を前提**：**将来の機能（タグ職人ランキング、配信者アカウント、開発者登録、宣伝ポイント）を最初から構造に織り込む**

---

## UI設計原則（最優先 / 迷ったらここに戻る）

このサイトは**5種類**のページで構成される。それぞれUI思想が違う。**絶対に混ぜない**。

### A. ランキングページ（/, /trending, /categories, /new, /global）

**思想：事実を淡々と提示する。編集者の意図を一切混ぜない。ニコニコ動画のランキングページが手本。**

- 1位も100位も**同じ行の高さ・同じフォントサイズ・同じサムネサイズ**
- 順位バッジは1〜3位のみ色差（金・銀・銅）OK、ただし**サイズは同一**
- 1行のカラム構成固定：`[順位] [サムネ60×60] [タイトル+開発者] [CCU] [変動]`
- 装飾禁止：影、グラデーション、ホバーアニメーション、派手なアイコンNG
- フォントサイズは2種類まで：タイトル（14px）、数値（14px tabular-nums）
- 変動だけ色差OK：↑緑、↓赤、NEW青バッジ
- 更新時刻常時表示：「3分前更新」
- 参考UI：**Yahoo!リアルタイム検索、価格.comランキング、ニコニコ動画ランキング、Googleトレンド**

### B. ピックアップページ（/featured）

**思想：編集者（Yuki）の主観で推す。熱量を出していい場所。**

- カードベース、大きいサムネ、キャッチコピーOK
- 1枚ごとに一言コメント（推薦理由）を必ず入れる
- 装飾・装丁のバリエーションOK
- 手動更新（Supabaseダッシュボードから直接編集）
- ランキングページとは**明確に別物**に見える見た目

### C. 配信者ページ（/stream, /stream/[slot]）

**思想：配信者／インフルエンサーが「今夜の配信ネタ」を5秒で選べる専用ハブ。詳細ページとも事実ベースのランキングとも分離する。ヘッダーの「配信」タブで独立し、トップページには混ぜない（ランキングの淡々としたトーンを壊さない）。今後コンテンツを厚くしていく柱の1つ。**

- トップ `/stream` は用途別スロットのカードグリッド（collab / viewer / short / reaction / no-english / loud の6枠、DB駆動化は将来）
- `/stream/[slot]` は該当タグを持つゲーム一覧。カードには配信向けバッジ最大3つ、英語ハードルを一目で表示
- 並び順：`confidence_score DESC, editorial_score_stream DESC, vote_count DESC`
- 詳細ページ（`/game/[universeId]`）には `StreamMetaPanel` を**条件付きで**埋め込む（`game_streaming_meta` 行が存在するゲームだけ）。一言ピッチ、配信向けポイント3つ、FitMatrix（ソロ/コラボ/視聴者参加/切り抜きの4軸）、最初の10分、今なぜ配信向きか、注意点（英語依存度・BGM・年齢感・チャット露出）
- 各ゲームに **シェアカード（OG画像）** を `next/og` で動的生成：日本語一言ピッチ＋配信バッジ最大3＋英語ハードル。Xシェアの標準ツールに
- 文言ルール厳守：「安全に配信できます」「著作権的に問題ありません」「絶対バズる」「クソゲー」等は**禁止**。管理画面でワードチェッカを発動

### D. タグページ（/tags, /tags/[slug]）

**思想：ユーザー集合知を集約・閲覧するための場所。ランキングでもピックアップでもない第3の発見軸。**

- タグは3層構造：**公式タグ**（運営付与・骨格）／**ユーザー推薦タグ**（プールから選択式・空気感）／自由入力はMVPでは封印
- `/tags` はタグ一覧（人気／新着／カテゴリ別）
- `/tags/[slug]` はタグ詳細＋紐づくゲーム一覧（得票順）＋そのタグを多く付けた「タグ職人」枠＋関連タグ
- 詳細ページにも `TagCloud` を埋め込み：公式は塗り、ユーザーは枠線、得票数に応じて濃淡
- 「タグを付ける」ボタン → `TagPickerModal`（最大5件・選択式のみ）。未ログイン時はログイン誘導
- ネガティブ表現は辞書で置換：「クソゲー」→「好み分かれる」、「治安終わってる」→「治安に波あり」
- タグ職人文化：ユーザーの `tagContributionScore` を日次バッチで更新。採用バッジ（公式採用／急上昇／配信者支持）で動機づけ

### E. AdSense 広告枠（フェーズ12以降）

**思想：サイト運営費を Google AdSense 等の受動広告で賄う。ユーザーから直接お金が出ない・クリエイターへの直接送金もしない（個人運営でのリスク管理優先）。「広告主と表示順位は無関係」を明示することでランキング聖域を守る。**

- 課金フロー（Stripe・クリエイター広告主からの直接支払い）は **永続的に不実装**
- AdSense は枠買いに過ぎず特定ゲーム/クリエイターを上位表示できない仕様 → ランキング聖域は自動的に守られる
- 広告は「ファーストビュー圧迫しない」「ランキング行間に挟まない」「自動再生動画NG」等の節度（`higesakusei/feature-spec.md` §6.2）
- 各広告枠に「📢 広告」または「PR」ラベルを明示（ステマ規制 2023-10 順守）
- フッターに「サイト運営は広告収益で賄われています。広告主と表示順位は無関係です」を明示
- クリエイター支援は「robro-jp 上で発見される機会そのもの」で達成。直接送金は行わない

詳細は `higesakusei/idea-evaluation-v4.md` および `feature-spec.md` §6（AdSense 統合）。

### ナビゲーション階層（絶対に混ぜない）

```
ヘッダー：[ランキング] [配信] [タグ] [ピックアップ] [ご意見（控えめ）] [クリエイター]
             ↑フェーズ1-3  ↑フェーズ7  ↑フェーズ6  ↑フェーズ5  ↑フェーズ8.5      ↑フェーズ10以降
  ├─ ランキング配下タブ：
  │     [日本で人気]（/ デフォルト）
  │     [日本の急上昇]（/trending）
  │     [カテゴリ別]（/categories）
  │     [今週の新着]（/new）
  │     [全世界総合]（/global）
  ├─ 配信：
  │     /stream（ハブ：用途別スロットカード）
  │     /stream/[slot]（collab/viewer/short/reaction/no-english/loud）
  ├─ タグ：
  │     /tags（人気／新着／カテゴリ別）
  │     /tags/[slug]（タグ詳細＋ゲーム一覧＋タグ職人）
  ├─ ピックアップ：
  │     /featured（一覧）
  │     /featured/[slug]（今週の配信ネタ記事）← フェーズ7で拡張
  ├─ ご意見：
  │     /feedback（Fider風ボード：投稿・投票・ステータス表示）
  │     ヘッダーでは text-muted-foreground で控えめに表示。主動線はランキング/配信/タグ/ピックアップ
  │     右下の FeedbackFab（/admin と /feedback 自身では非表示、閉じられる）からも到達可
  └─ クリエイター：
        /creators（クリエイター一覧・検索／フェーズ10）
        /creators/[id]（クリエイター詳細／フェーズ10）
        /creators/register（自薦登録／フェーズ10）
        ※ /promotions/* は v4 で廃止（クリエイター広告主の累計広告費公開機構は不要）
```

**重要**：ルート `/` は「日本で人気」。「全世界総合」は `/global` に格下げ。日本ユーザーが最初に見るのは日本で人気のゲーム。

タブに異種を混ぜない。階層を上げて分離する。

---

## 運営コンプライアンス（Roblox規約遵守 / 絶対守る）

**参考資料**：`higesakusei/ToS確認/02_調査でわかった事実.md`・`03_robro-jpリスク分析.md`・`06_2026-04-30_ToS改定.md` を必ず読む。一次資料は同フォルダの `資料1.txt`（Roblox ToS 本体）〜 `資料6.txt`（Privacy Policy）。行番号は一次資料のもの。

**⚠️ 2026-04-30 ToS改定の主要差分**（詳細は `06_2026-04-30_ToS改定.md`）：
- **Creator Terms §7 新設**：「Only access an API by the means described in the documentation of that API」「refrain from misrepresenting or masking your identity」 → **公式文書化された方法でのみAPIアクセス・身元秘匿禁止**
- **Privacy 改定**：「We do not show personalized ads to users under the age of 18 on Roblox」 → **18歳未満には personalized ads を表示しない設計が必要**（AdSense導入時に対応）
- **AI 条項本体統合**：旧AI補助規約が ToU §8 / Creator Terms §6 に吸収。AI機能を載せる時は明示開示が条項要件に
- **Authorities共有強化**：Roblox側がDSA・UK Online Safety Act等の法執行に応じてデータ・通信内容を共有する範囲が明確化（robro-jp側の対処は不要だが認識）
- **Cookie-based API は production 非推奨と明言**：`.ROBLOSECURITY` 自動利用・X-CSRF再現は危険度上昇 → robro-jp の現方針（無認証GETのみ）は整合

**今すぐ実装（フェーズ9で完了）**：
1. 全ページフッターに **「当サイトは Roblox Corporation の公式サービスではありません。Robloxによる承認・提携・スポンサー提供を受けていません。」** を表示（資料2 L34 App Terms §1.1.3「may not … imply any affiliation with Roblox」 / §10「Roblox is not affiliated in any way with you or your App」）
2. `/privacy`（プライバシーポリシー）・`/terms`（利用規約）・`/contact`（削除申請・問い合わせ）ページ実装

**必須UI表記4種**（2026-04-30 ToS改定で必要性が再確認・詳細は `06_2026-04-30_ToS改定.md`）：
1. **非公式表記**（フッター固定）：上記の「公式サービスではない」文言
2. **投票UI近接表記**：「❤️⭐🔥 は robro-jp 独自のサイト内投票です。Roblox 公式の Discover、評価、推薦、ランキングを表すものではありません。」
3. **自薦登録フォーム近接表記**：「本人確認は、あなたが Roblox プロフィール bio に一時掲載した確認コードを、公開 users API で照合して行います。パスワード、`.ROBLOSECURITY` Cookie、アクセストークンの入力は求めません。」
4. **広告ラベル**：各広告枠に「📢 広告」または「PR」明示。About/Legal で「本サイトの広告は robro-jp が外部広告配信事業者を通じて掲載しているものであり、Roblox 公式広告ではありません」

**技術的遵守事項（出典明記）**：
1. **Roblox由来データの売買禁止**（資料1 L292(a) / 資料3 L62）— 有料宣伝機能は Roblox との個別ライセンス契約なしに実装しない（v4で永続的に不実装）
2. **Robloxユーザーのプロファイリング禁止**（資料3 L54 / Creator Third-Party App Policy「Do not build profiles for Roblox Users」）— fingerprint は**サイト訪問者の重複投票防止**用途のみ。Robloxアカウントとの紐付け禁止（OAuth連携しない方針）。**bio照合は one-shot verification のみ・bio本文は長期保存しない**（プロファイリング寄りになるため）
3. **AI/LLM訓練への投入禁止**（資料1 L292(d) / 資料3 L63 / 2026-04-30 ToU §8 / Creator Terms §6）— `games.description`・ゲーム名・サムネイル等を LLM の fine-tuning / RAG データセット / embeddings 訓練に使わない。Claude Code などの開発支援ツールに参照させるのは可（モデル重みを更新しない純粋な読み取りのため）。**将来 robro-jp にAI機能を載せる場合は「AI is not human」「AI output is artificially generated」のUI開示が条項要件**
4. **データ削除の即時実行体制**（資料3 L61）— `scripts/purge-roblox-data.ts` を用意。Roblox警告時に `games` / `game_snapshots` / `game_streaming_meta` を即削除。独自データ（タグ投票・featured記事）は維持
5. **投稿ログretention** — `game_tag_vote_logs` ・ `game_button_vote_logs` は**6ヶ月〜1年保存**（日本プロバイダ責任制限法の発信者情報開示請求に備える）
6. **description は要約表示** — 先頭200文字まで、「詳細は Robloxで見る」導線を必須添付（クリエーター規約 L423 の著作権配慮）
7. **サムネイルは Roblox CDN 直リンク**のみ（自前再配信禁止。クリエーター規約 L424 のサブライセンス権の射程内に留める）
8. **API アクセス方法の制約**（Creator Terms §7 / 2026-04-30）— **公式に文書化された方法のみ**でAPIアクセス。`.ROBLOSECURITY` Cookie・X-CSRF再現・ヘッドレスログイン・Cloudflareバイパス・rate limit 回避は禁止。**User-Agent に `robro-jp` と連絡先を含め、身元秘匿しない**
9. **API レート制限の扱い**（DevForum: non-Open Cloud は rate limit 不透明・縮退方針）— 429 を正常系として扱い、指数バックオフ＋jitter＋強キャッシュで運用。endpoint ごとに kill switch を用意

**商標・ブランド**（資料4 全体 / [Name and Logo Community Usage Guidelines](https://en.help.roblox.com/hc/en-us/articles/115001708126)）：
1. **Robloxロゴは使わない**（資料4 L17 / "Besides our official badge, we do not permit the general use of our logo"）
2. **Roblox Tilt（傾けロゴ）使用禁止**
3. **"Now on Roblox" バッジ画像は使わない**：
   - Roblox公式ガイドでは「**creators including brand partners can use the badge for all digital promotions off of the Roblox platform**」と明示。一見 OK に見えるが、これは **creator が自分の experience を宣伝する** 文脈の許諾
   - robro-jp は第三者リスティング（他人のゲームを並べる立場）であり、かつ AdSense による広告収益がある（"commercial content" 該当の余地あり）ため、**安全側でバッジ画像は使わない**
   - ただし **テキスト表現は OK**：「Robloxで見る」「Robloxで遊ぶ」「Robloxで開く」「Now on Roblox」（テキスト・ロゴ画像なし）はゲームへのリンクボタン文言として使用可（資料4 L17 "「Now on Roblox」 相当の文字表現のみOK"）
4. サイト名・ドメインに **"Roblox" または類似語（Blox, Bloxy, Roblo 等）を含めない**
   - 現状 `robro-jp` は OK（"ro-bro" の発音、Roblox 要素なし）
   - `roblo.fortunep.net` は **NG**（"Roblox" の部分文字列 "roblo" を含む）→ 確定：`robro.fortunep.net`
   - 資料4 L19 の Experience タイトル規制は Roblox 内の話で外部サイトに直接適用されないが、日本の商標法・不正競争防止法の「出所混同」回避として自発的に順守
5. **サイト名・ロゴ・favicon・OG画像・ランキングUI語彙で Roblox 公式UIに寄せない**（誤認防止）
6. 「公式」「公認」「パートナー」「認定」等の表現は**一切使わない**（資料2 L12 "No Misleading or Affiliation" / Name and Logo Guidelines「Imply official Roblox partnership or endorsement」禁止）

**ユーザー投稿モデレーション**（資料5 Community Standards）：
1. タグは**選択式のみ**（自由入力禁止 / MVP継続）
2. ネガティブ表現は辞書で置換済（クソゲー→好み分かれる 等。資料5 L29-45 の体型・特徴辱め禁止に相当）
3. 削除申請は**24時間以内に一次対応**。管理UIで特定タグをゲームから非表示にできる機能を用意
4. タグ投票は「サイト訪問者によるゲームへのタグ付与」であり、**Robloxユーザーを代理した投票ではない**旨を利用規約に明記（資料3 L38 との境界を明確化）

**13歳未満ユーザー**（資料6 L131・L381、COPPA・日本個人情報保護法）：
- 現状は匿名投票のみで個人情報収集なし
- プライバシーポリシーに「13歳未満は保護者の同意のもとでご利用」を明記
- ログイン機能導入時は年齢確認フローを併設
- fingerprint は個人関連情報として扱い、保存期間・用途・第三者提供なしを明記

**日本法の特則**（資料1 L562-578 付録B）：
- 日本ユーザーとの契約相手は **Roblox 合同会社** であることを認識（我々の規約で Roblox を参照する際の名称に反映）
- 利用規約の免責条項は「当方の故意または重過失の場合を除く」と明記（付録B L576 に整合）
- 日本の消費者の紛争は日本の裁判所でも提起されうる前提で運営

**重要なリスク（再掲）**：フェーズ8以降に当初予定していた「有料宣伝枠」は 資料1 L292(a) 販売禁止・資料3 L62 データ販売禁止・資料4 L20 商業利用ライセンス要件に抵触する可能性が高いため保留。`higesakusei/ToS確認/03_robro-jpリスク分析.md` 参照。

---

## 技術スタック（確定事項 / 逸脱しない）

| レイヤー | 採用 |
|---|---|
| フロントエンド | Next.js 16 (App Router, Turbopack) + React 19 + TypeScript |
| スタイル | Tailwind CSS + shadcn/ui |
| グラフ | Recharts |
| DB | Supabase (PostgreSQL) |
| データ取得ジョブ | Vercel Cron (初期) → 負荷増大で n8n 移行 |
| データソース（主） | Roblox公式 explore-api（`get-sort-content`）：`top-playing-now` / `top-trending` / `up-and-coming` / `top-revisited` などの全世界ソートをunionして上位500件を取得 |
| データソース（補） | Roblox公式 games/v1 API（詳細・CCU詳細）、Roblox公式 thumbnails API |
| 日本ゲーム抽出 | `japanese-detector.ts`（タイトル・説明の日本語スコア）。country=JP フィルタは認証必須で使えないため、この判定でJP向けゲームを近似 |
| ホスティング | Vercel (東京リージョン) |
| ドメイン | robro.fortunep.net（`roblo.fortunep.net` は商標リスク回避のため却下） |
| 認証 | Supabase Auth (Email / Magic Link)。**Roblox OAuth は不採用**（Third Party App Policy抵触回避） |
| クリエイター本人確認 | Robloxプロフィール bio に確認コードを24時間掲載 → robro-jp が `users.roblox.com/v1/users/{id}` の公式APIで description を取得して照合（フェーズ10） |
| 広告 | Google AdSense（フェーズ12以降）。**Stripe等の決済プロバイダは不採用**（v4方針：ユーザー/クリエイター間の金銭フロー全廃） |

**採用しない**：Redis、WebSocket、SSE、Prisma（Supabase client直で十分）

---

## ディレクトリ構造

```
roblo-jp/
├── CLAUDE.md
├── app/
│   ├── layout.tsx                    # ヘッダーナビ
│   ├── (ranking)/
│   │   ├── layout.tsx                # ランキング配下タブ
│   │   ├── page.tsx                  # / 日本で人気（デフォルト）
│   │   ├── trending/page.tsx         # /trending 日本の急上昇
│   │   ├── categories/
│   │   │   ├── page.tsx              # /categories カテゴリ一覧
│   │   │   └── [slug]/page.tsx       # /categories/roleplay など
│   │   ├── new/page.tsx              # /new 今週の新着
│   │   └── global/page.tsx           # /global 全世界総合
│   ├── featured/
│   │   ├── page.tsx                  # /featured ピックアップ一覧
│   │   └── [slug]/page.tsx           # /featured/[slug] 今週の配信ネタ記事（フェーズ7）
│   ├── stream/                       # フェーズ7：配信者導線
│   │   ├── page.tsx                  # /stream ハブ
│   │   └── [slot]/page.tsx           # /stream/collab など
│   ├── tags/                         # フェーズ6：タグ機能
│   │   ├── page.tsx                  # /tags 一覧
│   │   └── [slug]/page.tsx           # /tags/[slug] タグ詳細
│   ├── promoted/                     # フェーズ8以降
│   │   ├── layout.tsx
│   │   ├── page.tsx                  # /promoted 総合宣伝
│   │   ├── company/page.tsx          # /promoted/company 企業宣伝
│   │   └── user/page.tsx             # /promoted/user ユーザー宣伝
│   ├── game/[universeId]/page.tsx
│   └── api/
│       └── cron/
│           └── fetch-games/route.ts
├── lib/
│   ├── supabase.ts
│   ├── roblox-api.ts                 # games/v1 詳細取得
│   ├── roblox-charts.ts              # 公式Charts スクレイピング（country=JP / 全世界）
│   ├── rolimons-api.ts               # 全世界上位リスト補完用（従属）
│   ├── japanese-detector.ts
│   ├── tags.ts                       # フェーズ6：タグ集計・confidence_score計算
│   ├── streaming.ts                  # フェーズ7：stream-metaヘルパー・文言チェッカ
│   ├── og.ts                         # フェーズ7：OG画像生成ヘルパー
│   ├── validators/                   # zod スキーマ
│   │   ├── streamMeta.ts
│   │   └── tagVote.ts
│   └── promotion/                    # フェーズ8以降
│       ├── pricing.ts                # 料金計算
│       └── ranking.ts                # 宣伝ランキングのクエリ生成
├── components/
│   ├── RankingRow.tsx                # 全順位共通の1行
│   ├── FeaturedCard.tsx
│   ├── tag/                          # フェーズ6
│   │   ├── TagBadge.tsx              # variant: official | community
│   │   ├── TagCloud.tsx
│   │   └── TagPickerModal.tsx
│   ├── stream/                       # フェーズ7
│   │   ├── StreamSlotCard.tsx
│   │   ├── StreamBadge.tsx
│   │   ├── StreamMetaPanel.tsx       # 詳細ページ埋め込み
│   │   ├── StreamFitMatrix.tsx
│   │   ├── StreamCautionList.tsx
│   │   ├── FirstTenMinutesBox.tsx
│   │   ├── WhyNowPopularBox.tsx
│   │   └── ShareCardButton.tsx
│   ├── PromotedRow.tsx               # フェーズ8以降
│   └── TrendChart.tsx
├── types/
│   └── game.ts
└── scripts/
    └── seed-universe-ids.ts
```

---

## DBスキーマ（Supabase / PostgreSQL）

**必ずこのスキーマを守る。変更時は必ず確認を取る。**

### フェーズ1-5で使うテーブル

```sql
-- ゲームマスタ
CREATE TABLE games (
  universe_id BIGINT PRIMARY KEY,
  place_id BIGINT,
  name TEXT NOT NULL,
  description TEXT,
  creator_name TEXT,
  creator_type TEXT,          -- 'User' or 'Group'
  thumbnail_url TEXT,
  is_japanese BOOLEAN DEFAULT FALSE,
  japanese_score REAL DEFAULT 0,
  -- カテゴリ（Roblox 詳細API由来）
  genre_l1 TEXT,               -- 表示用：'Roleplay & Avatar Sim' など
  genre_slug TEXT,             -- URL用：'roleplay_and_avatar_sim' など
  -- 将来の拡張用：開発者登録機能で使う
  owner_account_id BIGINT,    -- accountsテーブルへのFK（フェーズ6以降）
  is_verified_by_us BOOLEAN DEFAULT FALSE,  -- 自前認証
  first_seen_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_games_is_japanese ON games(is_japanese);
CREATE INDEX idx_games_updated_at ON games(updated_at);
CREATE INDEX idx_games_owner ON games(owner_account_id);
CREATE INDEX idx_games_first_seen_at ON games(first_seen_at DESC);  -- 今週の新着用
CREATE INDEX idx_games_genre_slug ON games(genre_slug);              -- カテゴリ別ランキング用

-- 時系列スナップショット（CCU等の数値）
CREATE TABLE game_snapshots (
  universe_id BIGINT REFERENCES games(universe_id),
  captured_at TIMESTAMPTZ NOT NULL,
  playing INT NOT NULL,
  visits BIGINT,
  favorites BIGINT,
  PRIMARY KEY (universe_id, captured_at)
);
CREATE INDEX idx_snapshots_captured_at ON game_snapshots(captured_at DESC);

-- NOTE: 当初 chart_rankings テーブルで source 別に保存する設計だったが、
-- explore-api にカテゴリ別 sort がなく、代わりに games.genre_l1 / genre_slug（Roblox詳細API由来）
-- でカテゴリ分類できることが判明したため、chart_rankings テーブルは不要になった。
-- ランキングはすべて game_snapshots + games の WHERE 条件で表現する：
--   全世界総合  : games 全件、playing 降順
--   日本で人気  : games.is_japanese = true、playing 降順
--   急上昇      : 前スナップショットとの playing 比率
--   カテゴリ別  : games.genre_slug = '<slug>'、playing 降順
--   今週の新着  : games.first_seen_at >= now() - '7 days'、playing 降順

-- ピックアップ（編集者手動）
CREATE TABLE featured_games (
  id BIGSERIAL PRIMARY KEY,
  universe_id BIGINT REFERENCES games(universe_id),
  headline TEXT NOT NULL,
  comment TEXT NOT NULL,
  display_order INT DEFAULT 0,
  is_active BOOLEAN DEFAULT TRUE,
  featured_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_featured_active ON featured_games(is_active, display_order);
```

### フェーズ6：タグ機能で使うテーブル

**設計原則**：拡張設計ガイドライン#1（イベントログ型）・#3（TEXT + CHECK）を順守。自由入力タグはMVPでは受け付けず、`tag_master` からの選択式のみ。

```sql
-- タグマスタ（公式タグ・ユーザー選択式タグプール）
CREATE TABLE tag_master (
  tag_id TEXT PRIMARY KEY,              -- 'collab_good' など英小文字slug
  tag_name TEXT NOT NULL,               -- 表示名「コラボ向き」
  tag_type TEXT NOT NULL CHECK (tag_type IN ('official','user_selectable','free')),
  tag_group TEXT NOT NULL CHECK (tag_group IN ('format','reaction','participation','caution','difficulty','vibe','genre')),
  description TEXT,
  is_streaming_related BOOLEAN NOT NULL DEFAULT FALSE,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  sort_order INT NOT NULL DEFAULT 0
);
CREATE INDEX idx_tag_active ON tag_master(is_active, sort_order);
CREATE INDEX idx_tag_streaming ON tag_master(is_streaming_related) WHERE is_streaming_related = TRUE;

-- タグ投票集計（ゲーム×タグの一意、vote_countはトリガで更新）
CREATE TABLE game_tag_votes (
  universe_id BIGINT REFERENCES games(universe_id) ON DELETE CASCADE,
  tag_id TEXT REFERENCES tag_master(tag_id) ON DELETE CASCADE,
  vote_count INT NOT NULL DEFAULT 0,
  confidence_score REAL NOT NULL DEFAULT 0,  -- min(1, vote_count / (vote_count + K)), K=10
  last_voted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  PRIMARY KEY (universe_id, tag_id)
);
CREATE INDEX idx_gtv_universe ON game_tag_votes(universe_id);
CREATE INDEX idx_gtv_tag_conf ON game_tag_votes(tag_id, confidence_score DESC);

-- 投票生ログ（イベントログ型。荒らし検知・取り消し・職人スコア算定に使う）
CREATE TABLE game_tag_vote_logs (
  id BIGSERIAL PRIMARY KEY,
  universe_id BIGINT NOT NULL,
  tag_id TEXT NOT NULL,
  account_id BIGINT,                   -- ログインユーザー（phase6でaccounts稼働時）
  fingerprint TEXT NOT NULL,           -- IP hash + UA hash（匿名投票の重複防止）
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_gtvl_fp_time ON game_tag_vote_logs(fingerprint, created_at DESC);
CREATE INDEX idx_gtvl_account ON game_tag_vote_logs(account_id) WHERE account_id IS NOT NULL;

-- タグ職人スコア（日次バッチで再計算、拡張ガイドライン#5：RPC/Viewで集計）
-- accounts.tag_contribution_score カラムに書き戻す想定（accounts テーブル稼働後）
```

**レートリミット（アプリ層で実装）**：
- 1 fingerprint × 1 (game_id, tag_id) = 24h内1票
- 1 fingerprint = 60秒20票、1日50票
- 違反は 429

### フェーズ7：配信者導線で使うテーブル

```sql
-- ゲーム別・配信メタ情報（運営が手動で入れる。Supabase管理画面 or /admin 画面から）
CREATE TABLE game_streaming_meta (
  universe_id BIGINT PRIMARY KEY REFERENCES games(universe_id) ON DELETE CASCADE,
  short_pitch_ja TEXT NOT NULL,              -- 5〜60字：一言でいうと
  stream_summary_ja TEXT NOT NULL,           -- 10〜200字
  stream_points JSONB NOT NULL DEFAULT '[]', -- 最大3件の配信ポイント
  solo_fit TEXT NOT NULL CHECK (solo_fit IN ('high','mid','low')),
  collab_fit TEXT NOT NULL CHECK (collab_fit IN ('high','mid','low')),
  viewer_participation_fit TEXT NOT NULL CHECK (viewer_participation_fit IN ('high','mid','low')),
  clip_fit TEXT NOT NULL CHECK (clip_fit IN ('high','mid','low')),
  english_barrier TEXT NOT NULL CHECK (english_barrier IN ('low','mid','high')),
  learning_curve TEXT NOT NULL CHECK (learning_curve IN ('easy','normal','hard')),
  first_10min_guide TEXT NOT NULL DEFAULT '',
  why_now_popular TEXT NOT NULL DEFAULT '',
  stream_caution_notes JSONB NOT NULL DEFAULT '[]',  -- [{id,label,body,severity}]
  recommended_party_size TEXT NOT NULL DEFAULT '',
  average_session_length TEXT NOT NULL DEFAULT '',
  share_card_enabled BOOLEAN NOT NULL DEFAULT TRUE,
  editorial_score_stream INT NOT NULL DEFAULT 0 CHECK (editorial_score_stream BETWEEN 0 AND 100),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_gsm_editorial ON game_streaming_meta(editorial_score_stream DESC);

-- 配信用途スロット（/stream/[slot] のマスタ。固定でコードに持たず、ここに置く = 拡張ガイドライン#2）
CREATE TABLE stream_slots (
  slot_key TEXT PRIMARY KEY,           -- 'collab','viewer','short','reaction','no-english','loud'
  display_name TEXT NOT NULL,
  description TEXT NOT NULL,
  sort_order INT NOT NULL DEFAULT 0,
  is_active BOOLEAN NOT NULL DEFAULT TRUE
);

-- スロット→タグのマッピング（1対N）
CREATE TABLE stream_slot_tags (
  slot_key TEXT REFERENCES stream_slots(slot_key) ON DELETE CASCADE,
  tag_id TEXT REFERENCES tag_master(tag_id) ON DELETE CASCADE,
  PRIMARY KEY (slot_key, tag_id)
);

-- 特集記事（「今週の配信ネタ」。featured_games は単発ピックアップ、こちらは記事型）
CREATE TABLE stream_featured_articles (
  article_id TEXT PRIMARY KEY,
  slug TEXT UNIQUE NOT NULL,           -- /featured/[slug]
  title TEXT NOT NULL,
  summary TEXT NOT NULL,
  eyecatch_text TEXT,
  featured_universe_ids JSONB NOT NULL DEFAULT '[]',   -- [BIGINT...]
  status TEXT NOT NULL CHECK (status IN ('draft','published')) DEFAULT 'draft',
  published_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_sfa_status_pub ON stream_featured_articles(status, published_at DESC);

-- シェアカードキャッシュ（OG画像の生成結果をバージョン管理。任意）
CREATE TABLE game_share_assets (
  asset_id BIGSERIAL PRIMARY KEY,
  universe_id BIGINT NOT NULL REFERENCES games(universe_id) ON DELETE CASCADE,
  asset_type TEXT NOT NULL CHECK (asset_type IN ('og_image','share_card')),
  image_url TEXT NOT NULL,
  version INT NOT NULL DEFAULT 1,
  generated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**slot → tag 初期マッピング**（`stream_slot_tags` シード）：
```
collab      → collab_good, voice_chat_plus
viewer      → viewer_join, scale_up
short       → short_play, easy_rule
reaction    → reaction_good, loud_fun
no-english  → no_english, easy_rule
loud        → loud_fun, reaction_good
```

**公式配信タグ初期シード**（`tag_master` の `is_streaming_related=TRUE`）：
```
stream_good(配信映え), collab_good(コラボ向き), solo_ok(ソロでもいける),
viewer_join(視聴者参加向き), short_play(短時間向き), long_play(長時間向き),
reaction_good(初見リアクション), loud_fun(叫ぶ系), slow_burn(じわじわ沼る),
no_english(英語ほぼ不要), easy_rule(ルール簡単), voice_chat_plus(通話あり推奨),
scale_up(人数いると化ける)
```

**文言モデレーション**：管理画面での入力時、以下は保存前にブロック：
- 禁止：「安全に配信できます」「著作権的に問題ありません」「絶対バズる」「クソゲー」「ガキ向け」「治安終わってる」
- 置換推奨：クソゲー→好み分かれる／ガキ向け→年齢層低めの印象／絶対バズる→配信映えしやすい

### フェーズ8以降で使うテーブル（設計だけ先に固める）

**以下は実装しない。ただし、上記のフェーズ1-5スキーマに外部キー `owner_account_id` を予約しておくことで、後でこのテーブル群を追加しても既存データが壊れない。**

```sql
-- アカウント（将来の開発者/企業登録）
CREATE TABLE accounts (
  id BIGSERIAL PRIMARY KEY,
  account_type TEXT NOT NULL,         -- 'user' | 'company'
  display_name TEXT NOT NULL,
  email TEXT UNIQUE,
  roblox_user_id BIGINT,              -- 本人確認用
  points_balance BIGINT DEFAULT 0,    -- 保有宣伝ポイント
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 宣伝枠マスタ（種別・単価をテーブル化）
-- 新しい枠を追加したい時はコード修正ではなくこのテーブルにINSERTするだけ
CREATE TABLE promotion_slots (
  id BIGSERIAL PRIMARY KEY,
  slot_key TEXT UNIQUE NOT NULL,       -- 'top_banner' | 'sidebar' | 'promoted_list' など
  display_name TEXT NOT NULL,
  base_price_per_day INT NOT NULL,     -- ポイント/日
  max_concurrent INT DEFAULT 1,        -- 同時掲載可能数
  is_active BOOLEAN DEFAULT TRUE
);

-- 宣伝トランザクション（イベントログ型）
-- すべての宣伝行為を1行=1イベントで記録。集計はクエリ側で行う
CREATE TABLE promotion_transactions (
  id BIGSERIAL PRIMARY KEY,
  account_id BIGINT REFERENCES accounts(id),
  universe_id BIGINT REFERENCES games(universe_id),
  slot_id BIGINT REFERENCES promotion_slots(id),
  points_spent BIGINT NOT NULL,
  starts_at TIMESTAMPTZ NOT NULL,
  ends_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_promo_universe ON promotion_transactions(universe_id);
CREATE INDEX idx_promo_account ON promotion_transactions(account_id);
CREATE INDEX idx_promo_active ON promotion_transactions(starts_at, ends_at);

-- 料金・レート設定（ハードコード禁止、ここに集約）
CREATE TABLE pricing_config (
  key TEXT PRIMARY KEY,                -- 'jpy_per_point' | 'min_purchase' など
  value_numeric NUMERIC,
  value_text TEXT,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 宣伝ランキングは「View」で実現する

細分化ランキング（少額多数/高額少数/企業/ユーザー）はすべて `promotion_transactions` への**クエリの違い**で表現する。新ランキングを足したくなったらテーブル追加ではなくクエリ追加で済む。

```sql
-- 例：高額少数ランキング（1件あたりの平均単価が高い順）
-- 例：少額多数ランキング（トランザクション数が多い順）
-- 例：企業宣伝ランキング（account_type = 'company' のみ集計）
-- すべて promotion_transactions + accounts + games のJOINで出る
```

---

## 拡張設計ガイドライン（絶対原則）

**新機能を追加する時、以下の原則を逸脱しそうになったら必ず停止して確認。**

### 1. イベントログ型を基本とする

- ユーザー行動・宣伝行為・ポイント消費などは**1行=1イベント**で記録
- 集計値は保存しない（毎回クエリで計算）
- 理由：後から集計軸を増やしても既存データが使える

### 2. マスタデータのテーブル化

- 以下はコードに書かない：
  - 料金、ポイントレート、宣伝枠の種類、プラン名、カテゴリ
- すべて対応する `*_config` または `*_slots` テーブルに持たせる
- 理由：値の変更にデプロイ不要

### 3. 列挙型はTEXTで持つ

- `account_type`、`slot_key` などはENUMではなくTEXT + CHECK制約
- 理由：PostgreSQLのENUMは後から値を追加するのがDDL必要で面倒

### 4. 外部キーは予約しておく

- フェーズ1で作ったテーブルにも、将来のテーブルへのFK列を予約する
- `games.owner_account_id` は今は常にNULLだが、列だけ用意しておく
- 理由：後で列追加すると既存行の扱いが煩雑

### 5. 集計ランキングはViewまたはRPCで

- アプリ側で複雑なSQLを書かず、Supabase側にViewやRPC（Stored Procedure）として定義
- 理由：クエリロジックの差し替えがDB側で完結する

### 6. 機能フラグの活用

- 未リリース機能はコードを入れても `NEXT_PUBLIC_FEATURE_PROMOTION=false` でUIから隠す
- 理由：段階的リリースとA/Bテストが可能

### 7. URL構造の一貫性

- カテゴリ分類のあるページは `/{category}/{subcategory}/` 形式で統一
- 例：`/promoted/company`、`/promoted/user`、`/promoted/high-value`
- 理由：SEOとUXの両方で予測可能な構造になる

---

## フェーズ分割（必ず順番に実装）

各フェーズの完了時に git commit。

### フェーズ0：プロジェクト初期化
**ゴール**：Next.jsが起動し、Supabaseに接続できる。

- [ ] Next.js + TS + Tailwind + App Router
- [ ] shadcn/ui init
- [ ] `@supabase/supabase-js` 導入
- [ ] `.env.local` 作成
- [ ] `lib/supabase.ts` でブラウザ用・サーバー用分離
- [ ] `/` にダミーテキスト表示

**完了条件**：`pnpm dev` で localhost:3000 が表示される。

### フェーズ1：データ取得パイプライン
**ゴール**：5〜10分ごとに「日本Chart」「全世界Chart」「カテゴリ別Chart」のUniverseIdと順位、およびCCUが蓄積される。

- [x] `lib/roblox-explore-api.ts`：公式 explore-api を叩き、複数sort（top-playing-now / top-trending / up-and-coming / top-revisited / カテゴリ別sort）をunionしてUniverseIdを取得
  - **調査済み制約**：`country=JP` 相当のフィルタは匿名アクセスでは効かない（filter値は常に `all`）。日本ゲームは `is_japanese` 判定で近似
- [ ] `lib/roblox-api.ts`：games/v1 でCCU/詳細を100件ずつ取得、バッチ間500msディレイ
- [ ] `lib/rolimons-api.ts`：全世界上位リスト補完用（Charts取得が失敗した時のフォールバック）
- [ ] `lib/japanese-detector.ts`：ルールベース判定
- [ ] `app/api/cron/fetch-games/route.ts`：Charts取得 → 重複除去したUniverseIdでRoblox詳細取得 → `games` upsert + `game_snapshots` insert + `chart_rankings` insert
- [ ] `CRON_SECRET` でBearer認証
- [ ] `vercel.json` に `"*/10 * * * *"`

**完了条件**：手動叩きで `chart_rankings` に `source='official_jp'` と `'official_global'` の行が両方入る。2回目で重複エラーなし。

### フェーズ2：日本で人気ランキング（デフォルト）
**ゴール**：`/` に公式Charts(country=JP)の最新順位トップ100を表示。

- [ ] `(ranking)/layout.tsx`：タブナビ（日本で人気 / 急上昇 / カテゴリ / 新着 / 全世界）
- [ ] `(ranking)/page.tsx`：`chart_rankings` の最新 `source='official_jp'` をJOIN
- [ ] `components/RankingRow.tsx`：全順位で見た目同じ
- [ ] モバイル縦リスト、デスクトップテーブル
- [ ] 更新時刻表示
- [ ] `revalidate = 300`

**完了条件**：1位と100位の見た目が順位バッジ以外同じ。URLが `/` でJP Chartsが表示される。

### フェーズ3：急上昇・カテゴリ別・今週の新着・全世界
**ゴール**：5種類のランキング。**同じ `RankingRow` を使い回す**。

- [ ] `/trending`：日本Chartsの順位変動（前回captureからのrank差分、新規=NEWバッジ）
- [ ] `/categories`：カテゴリ一覧（公式Chartsのカテゴリslug列挙、テーブル化：`category_slots` などにマスタを置く）
- [ ] `/categories/[slug]`：`source='official_category:<slug>'` で絞り込み
- [ ] `/new`：`games.first_seen_at >= now() - interval '7 days'` を playing 降順で
- [ ] `/global`：`source='official_global'` の最新順位

**完了条件**：5URLで並び順が異なる。行の見た目は全ページ同一。

### フェーズ4：個別ゲームページ
**ゴール**：詳細+24時間CCU推移グラフ。

- [ ] `/game/[universeId]`
- [ ] Recharts LineChart（直近24時間、5分刻み）
- [ ] 公式Robloxへの遷移ボタン

**完了条件**：ランキングから遷移可、グラフ描画OK。

### フェーズ5：ピックアップページ
**ゴール**：Yukiが手動で選んだ推しを掲載。

- [ ] `/featured`：`featured_games` JOIN
- [ ] `components/FeaturedCard.tsx`：カード型
- [ ] ヘッダーに「ピックアップ」追加
- [ ] トップページ上部にミニセクション+導線

**完了条件**：Supabaseで行追加すれば即反映。

### フェーズ6：タグ機能（MVP1）
**ゴール**：公式タグが全ゲームに付き、ログインユーザーがプールから選んで投票できる。詳細ページでタグが見える。

**優先度**：宣伝より先にこれを実装する。ro-bro の差別化の中核は「日本語の通訳」であり、タグはその中核要素。

- [ ] スキーマ：`tag_master` / `game_tag_votes` / `game_tag_vote_logs` / `accounts`（最小＝ログインのみ）
- [ ] シード：公式タグ + ユーザー選択式タグプール（higesakusei/新しい方向性/タグ仕様書.txt のシード表に準拠）
- [ ] API：`GET /api/games/[id]/tags` / `POST /api/games/[id]/tags`（選択式・最大5件・レートリミット）
- [ ] コンポーネント：`TagBadge` / `TagCloud` / `TagPickerModal`
- [ ] 詳細ページ上部改修：一言ピッチ → 公式タグ → 人気ユーザータグ → 「タグを付ける」ボタン
- [ ] `/tags`（人気/新着/カテゴリ別）、`/tags/[slug]`
- [ ] トップ「目的別で選ぶ」セクション（初めてのRobloxならこれ／英語ほぼ不要／友達向け等）
- [ ] モデレーション：選択式のみ受け付け、レートリミット、ネガティブ表現辞書
- [ ] 管理画面：タグ管理・ゲーム別割り当て
- [ ] OPS：`tag_contribution_score` 日次バッチ（タグ職人）

**完了条件**：任意のゲーム詳細ページでタグ投票ができ、`/tags/collab_good` で該当ゲーム一覧が表示される。

### フェーズ7：配信者導線（MVP1）
**ゴール**：配信者が `/stream` から「今夜のネタ」を選び、詳細ページの StreamMetaPanel と OG シェア画像で紹介コピーに困らない。

- [ ] スキーマ：`game_streaming_meta` / `stream_slots` / `stream_slot_tags` / `stream_featured_articles`
- [ ] 手動入力：10本のゲームに `game_streaming_meta` を投入（Sprint 1相当）
- [ ] API：`GET /api/games/[id]/stream-meta`、`GET /api/stream/slots/[slot]`、`GET/POST /api/featured`
- [ ] 詳細ページ：`StreamMetaPanel`（存在する時だけ表示）。FitMatrix / CautionList / FirstTenMinutesBox / WhyNowPopularBox / ShareCardButton
- [ ] `/stream`（ハブ）、`/stream/[slot]`（6スロット）
- [ ] `/featured/[slug]`（今週の配信ネタ記事、1本公開）
- [ ] OG画像：`/api/og/game/[id]`、`/api/og/featured/[slug]`（`next/og` で1200x630）
- [ ] 計測：`stream_entry_click` 等の9イベント（higesakusei/配信者導線.txt §9.1）
- 配信導線はヘッダーの「配信」タブで独立させる（トップページに6スロットカードは置かない。ランキングの淡々としたトーンを崩さないため）

**完了条件**：`/stream/collab` で該当ゲームが並び、詳細ページの配信ブロックが出て、OG画像がX共有で映える。

### フェーズ8以降（v4方針：集合知ランキング＋AdSense運営）

**⚠️ 2026-04-30 v4 再ピボット**：v3 で予定していた「クリエイター広告（本人プロフィール広告）」は、個人運営で金銭フローを抱えるリスク管理が重い（規約・税務・決済代行・ステマ規制・トラブル対応）と判断。代わりに大規模無料サイト（ポケモン徹底攻略・gamewiz・ニコニコ大百科）と同じ **AdSense モデル**（受動広告で運営費）に移行。**ユーザー/クリエイター間の金銭フローは全廃**。詳細は `higesakusei/idea-evaluation-v4.md`。

ピボットの根拠：
- 推定型ランキング（`is_japanese` 判定）が原理的に機能していない → ユーザー投票による集合知ランキングへ（v3で確定）
- 北極星「**優秀なクリエイターが真っ当に評価されればそれでいい**」は **直接送金がなくても達成可能**（可視化・推薦・流入で十分）
- 個人がルールの穴を突いて金銭を流す仕組みは規制リスクが大きい → 受動広告のシンプルモデルが最適

#### v4 フェーズ計画

- **フェーズ8**：3ボタン投票（❤️好き / ⭐お気に入り / 🔥頼むから人来て）+ ベイズ平均ランキング ← **実装中**
  - DB：`accounts` / `game_button_votes` / `game_button_vote_logs` / `user_savings` / マテビュー `game_voting_scores`
  - 重み 0.5 / 1.0 / 2.1（行動コスト＝情報価値の原理）、時間減衰半減期7日、CCUとの合成
  - レートリミット：1 account × 1 ボタン = 24h / 1 account = 60秒20票・1日100票
- **フェーズ9**：法令・規約対応（`/privacy` `/terms` `/contact`、削除申請フロー、フッター「公式ではない」表記、`scripts/purge-roblox-data.ts` 準備）
- **フェーズ10**：クリエイター自薦登録（無料）
  - `creators` / `creator_games` テーブル
  - Robloxプロフィール bio に確認コード掲載 → `users.roblox.com/v1` 公式APIで照合
  - `/creators` 一覧、`/creators/[id]` 詳細、`/creators/register` 登録フロー
- **フェーズ11**：タグ職人バッジ／ランキング（タグ機能と統合）
- **フェーズ12（v4再定義）**：**AdSense 統合**
  - Google AdSense 審査・審査通過後に AdSlot コンポーネント設置
  - 配置：フッター直上・ゲーム詳細の説明文ーCCUグラフ間・/featured/[slug] 本文中段
  - ラベル：「📢 広告」または「PR」を明示
  - フッター透明性表記：「サイト運営は広告収益で賄われています。広告主と表示順位は無関係です」
  - **Stripe・creator_ad_slots 等は実装しない**（v3旧仕様の退避）
- **フェーズ13（v4軽量化）**：軽量モデレーション
  - 短時間大量🔥検出（H2）・ユーザー通報UI（H4）・BAN公開ログ（H5）のみ
  - v3 の「同一決済元検出（H1/H6）」「アカウント年齢×広告額閾値（H7）」は不要（決済が無いため）
- **フェーズ14**：Tier機能（個人Tier→集合知Tier、票が割れてるゲーム）
- **フェーズ15+**：必要に応じてスポンサー記事（運営編集記事の中で企業名を載せる形・別途規約整備）

---

### フェーズXX（永続的に不実装）：金銭フロー全般

**不実装の理由**：
- Robloxゲームを対価で露出させる行為は商業利用条項抵触（資料1 L292a / 資料3 L62 / 資料4 L20）
- クリエイター本人広告（v3案）も、個人運営での金銭フロー管理リスクが大きい（v4で廃止）
- 投げ銭・ユーザーからクリエイターへの送金は決済代行審査・税務処理が個人運営で重い

**永続的に実装しないもの**：
- ユーザー → クリエイターの直接送金
- クリエイター → robro-jp の出稿料金（Stripe 等の決済プロバイダ全般）
- Robloxゲームの有料露出枠
- ユーザーの投げ銭・サポーター制（フェーズXX保留）

**代替**：AdSense（フェーズ12）で運営費、クリエイター支援は「発見される機会の最大化」で達成。

---

## コーディング規約

- **TypeScript strict mode必須**
- Server Componentデフォルト、Client Componentは必要時のみ `'use client'`
- Supabase型は `supabase gen types typescript` で自動生成
- エラーは握りつぶさず `console.error`
- `any` 禁止、`unknown` で絞り込む
- 日本語コメント推奨（設計意図は日本語で残す）
- **各フェーズ完了時に git commit**
- **マジックナンバー禁止**：料金・閾値は `pricing_config` テーブルまたは定数ファイルに

---

## 外部API使用上の注意

- 公式 explore-api（主ソース）：`apis.roblox.com/explore-api/v1/get-sort-content`。1 sort あたり ~90件。複数sortをunionして500件を構築。10分に1回程度。User-Agent付与
  - 使うsortId：`top-playing-now` / `top-trending` / `up-and-coming` / `top-revisited`、および将来のカテゴリsort
  - `country_filter_v3` パラメータは匿名では反映されない（確認済）
- Rolimons API：1分に1〜2回まで（フォールバック用途）
- Roblox公式API（games/v1）：UniverseId 100件/req、バッチ間500ms
- プロキシ禁止（rprxy.xyz等）
- サムネはRoblox CDN直リンク

---

## やらないこと（スコープ外）

- フェーズ1-5の範囲では：認証、ユーザー登録、WebSocket/SSE、多言語対応、モバイルアプリ、サーバーごとCCU詳細取得
- **Robloxゲームを対価で宣伝する機能**（商業利用条項抵触のため、永続的に不実装）
- **クリエイター本人有料広告枠**（v3案。v4で廃止：個人運営での金銭フロー管理リスクが大きいため）
- **Stripe・PayPay 等の決済プロバイダの導入**（v4方針：ユーザー/クリエイター間の金銭フロー全廃）
- **ユーザー → クリエイターの直接送金（投げ銭等）**（同上）
- **クリエイター自薦プロフィールでの集客誘導文・ゲームサムネ大型表示**（事実上のゲーム宣伝化するため、自薦コンテンツのモデレーション基準として `feature-spec.md` §14 の OK/NG 境界線を適用）
- **Roblox OAuth連携** — Third Party App Policy の scope 制限・プロファイリング制限に抵触するリスク高。本人確認が必要になったら別手段（Robloxプロフィール URL 貼付による簡易確認）で
- **Robloxゲーム description の全文掲載** — 先頭200文字までに要約
- **Robloxロゴ・ブランドの使用** — 必要時のみ"Now on Roblox"バッジ相当の表現で代替
- **Robloxユーザーのクロスプラットフォーム追跡** — fingerprint は同一サイト内の投票重複防止のみに使用
- **AI/LLM へのRoblox由来データ投入** — `games.description` 等を外部AIサービスに送らない
- **blox.js** の導入：公式API直叩き実装（`lib/roblox-api.ts` / `lib/roblox-explore-api.ts`）で足りており、導入しない。`.ROBLOSECURITY` cookie運用が必要になった時点で再評価
- **Contentlayer / fumadocs** の導入：デフォルトはDB駆動（`stream_featured_articles`）で、Yukiが Supabase Dashboard から即時編集できる方針を優先。フェーズ7で記事本文のリッチさ・編集頻度を見てMDX採用を再評価する余地あり

---

## 質問・確認

設計判断に迷った時、スキーマ・スタックから逸脱しそうな時は、**勝手に進めず必ず確認を取る**。

特に：
- ランキングページで装飾やサイズ差を入れたくなったら停止
- 「1位を目立たせるべき」と思ったらUI設計原則を再読
- 「この数字をコードに書けばいい」と思ったら拡張設計ガイドライン#2を再読
- 新テーブル追加を考えたら拡張設計ガイドライン#1, #4を再読

---

## 現在のフェーズ

**→ フェーズ1〜7は実装・本番公開済み（`/`, `/trending`, `/categories`, `/new`, `/global`, `/game/[id]`, `/featured`, `/stream`, `/feedback` が稼働）。**

**→ 2026-04-25 大方針転換**：推定型ランキング（`is_japanese` 判定）が原理的に機能していないことが判明し、**ユーザー投票による集合知ランキング + クリエイター広告**にピボット。北極星「優秀なクリエイターが真っ当に評価されればそれでいい」を中核原理として全機能を再整理した。

**次の実装：フェーズ8（3ボタン投票）**

参考資料（必ず目を通す・優先度順）：
- `higesakusei/idea-evaluation-v4.md` — **採否確定の最新版**（v3のクリエイター広告→v4でAdSenseモデルに再ピボット、2026-04-30）
- `higesakusei/idea-evaluation-v3.md` — 参考（クリエイター広告版・v4で廃止された設計の参照用）
- `higesakusei/feature-spec.md` — **仕様確定書**（v4対応済：§6 AdSense統合、§7-9 軽量化）
- `higesakusei/新しい方向性/タグ仕様書.txt` — タグ機能の開発チケット一覧（フェーズ6・11で参照）
- `higesakusei/新しい方向性/配信者導線.txt` — 配信者導線のv1.0仕様書（フェーズ7実装済）
- `higesakusei/ToS確認/03_robro-jpリスク分析.md` — Roblox規約抵触リスク分析（v3ピボットの法的根拠）
- Notion「ro-bro.jp 企画メモ」「まず結論」 — プロダクト哲学「面白さの通訳」の原典

フェーズ8着手時は、`feature-spec.md` §2-4 のスキーマ・投票システム・ランキング合成式をそのまま落とし込む。実装着手前に Yuki 確認を取ること（DB変更は重大）。

---
> Source: [higetuno-crypto/robro-jp](https://github.com/higetuno-crypto/robro-jp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
