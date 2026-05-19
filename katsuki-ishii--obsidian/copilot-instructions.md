## obsidian-page-creation

> Obsidianナレッジベースの作業ルール（フロントマター・文体・ディレクトリ構造）


# Obsidian ナレッジベース 作業ルール

## 基本方針

- **常体**で記述する（敬語は使わない）
- **絵文字は使用しない**
- **図解を含める**（必要に応じて）
- **具体例を含める**（理解を助けるため）

## 1. フロントマターを必ず付ける

新規 `.md` ファイル作成時、先頭に YAML を置く。

```yaml
---
base: "[[ディレクトリ名.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - [カテゴリ名]
作成日時: YYYY-MM-DDTHH:mm:ss
aliases: [エイリアス1, エイリアス2]
---
```

### baseプロパティのルール

`base`プロパティは、そのノートが属するディレクトリ名（数字プレフィックスを除く）に設定する。

- ディレクトリ名から数字プレフィックス（`000_`, `010_`, `015_`, `020_`, `031_`, `050_`など3桁）を除いた名前を使用
- 例：`000_Home/` → `[[Home.base]]`
- 例：`010_CS/04_Network/` → `[[CS.Network.base]]` または `[[CS.base]]`
- 例：`015_Web/Browser/` → `[[Web.Browser.base]]` または `[[Web.base]]`
- 例：`020_AWS/` → `[[AWS.base]]`
- 例：`030_ECMAScript/` → `[[ECMAScript.base]]`
- 例：`031_Browser_runtime/` → `[[Browser_runtime.base]]`
- 例：`040_Vue.js/` → `[[Vue.js.base]]`
- 例：`050_Development_Environment/` → `[[Development_Environment.base]]`
- 例：`200_Life/Psychology/` → `[[Life.Psychology.base]]`
- 例：`200_Life/Health/` → `[[Life.Health.base]]`
- 例：`200_Life/Ideas/` → `[[Life.Ideas.base]]`

この設定により、ObsidianのDataviewプラグインでディレクトリごとにノートを分類・集約できる。

### エイリアス（aliases）のルール

ObsidianのAutomatic linkerプラグインは完全一致のみ対応のため、以下のルールで設定：

1. **タイトル名そのまま**を含める
2. **主要キーワードを分解**して含める（複合語の場合）
3. **英語の場合は大文字小文字のバリエーション**を含める（例：OAuth/oauth, REST/rest）
4. **スペースの有無のバリエーション**を含める（例：v-for 数字/v-for数字）
5. **技術用語の別表記**を含める（例：v-for/vfor/v for, v-on/von/v on）
6. **関連する概念や別名**も含める（例：setup構文に`<script setup>`, `script setup`）
7. **日本語と英語の両方**を含める（該当する場合）
8. **バージョンや別表記**を含める（例：OAuth2, OAuth 2.0）

例：
- `巻き上げとTDZ.md` → `[巻き上げとTDZ, 巻き上げ, TDZ, 巻き上げとtdz, 巻き上げとTdz]`
- `OAuth.md` → `[OAuth, oauth, OAuth2, OAuth 2.0]`
- `v-for 配列.md` → `[v-for 配列, v-for, vfor, v for, key]`

## 2. 本文の約束

- **常体**で書く（敬語なし）。**絵文字は使わない**
- セクション区切りは `---`
- 見出しや重要語は Obsidian の `[[リンク名]]` でリンクする
- 必要に応じて図解・具体例を入れる

## 3. ディレクトリ構造（3桁番号）

- `000_Home/`: ホーム
- `010_CS/`: コンピュータサイエンス系。サブフォルダ（番号順は低レイヤーから高レイヤー）：
  - **01_Computation_Theory**: 計算論、オートマトン、計算量理論、形式言語
  - **02_Hardware**: CPU、メモリ階層、ハードウェアアーキテクチャ
  - **03_OS**: オペレーティングシステム、プロセス、スレッド、仮想メモリ、スケジューリング、並行・並列
  - **04_Network**: ネットワーク層・トランスポート層（レイヤー4まで）、TCP/IP、UDP、ルーティング、DNS
  - **05_Database**: 永続化、ストレージ、トランザクション、Session/Cookieの「状態をどこに持つか」など
  - **06_Security**: 認証・認可、暗号理論、OAuth、境界、攻撃・防御など
  - **07_Programming_Languages**: 言語の理論、パラダイム、型、コンパイル・インタプリタ（具体言語は030_ECMAScript等で）
  - **08_Algorithm**: アルゴリズム、データ構造、計算量、オーダー
  - **09_AI**: 機械学習、ディープラーニング、NLP、コンピュータビジョン、強化学習
  - **10_Software_Engineering**: 設計・設計パターン、テスト、ビルド・CI、開発手法
- `015_Web/`: Web技術。サブフォルダ：
  - **HTTP**: HTTP、HTTPS、HTTP/2、HTTP/3、URI/URL
  - **WebSocket**: WebSocketプロトコル、双方向通信
  - **REST**: RESTアーキテクチャスタイル、REST API設計
  - **API**: API設計パターン、GraphQL、バージョニング、OpenAPI/Swagger
- `020_AWS/`: AWS関連
- `030_ECMAScript/`: ECMAScript（JavaScript言語仕様）関連。npm/Vite等のツール系ノートも暫定的にここに含む
- `031_Browser_runtime/`: ブラウザランタイム関連（JavaScriptエンジン、レンダリングエンジン、DOM、Web API（Fetch/Storage 等））
- `040_Vue.js/`: Vue.js関連
- `050_Development_Environment/`: 開発環境・ツール（ESLint/Prettier、Docker、再現性など）
- `200_Life/`: ライフ系。サブフォルダ：**Psychology**（心理学・UX）、**Health**（健康）、**Ideas**（アイデアや思考の整理）
- `300_Journal/`: 日記やログ（将来用）

## チェックリスト（新規作成時）

- [ ] 先頭に上記形式の YAML フロントマターがある
- [ ] `base` が配置ディレクトリに対応している
- [ ] `aliases` にタイトル・検索されそうな表記を含めている
- [ ] 常体・絵文字なし・セクションは `---`・リンクは `[[...]]`
- [ ] 適切な 3桁フォルダ・サブフォルダに保存している

## 注意事項

- 既存のファイル構造とスタイルを尊重
- すべての応答は日本語で行う
- ユーザーの指示に対して疑問や不明点がある場合は、必ず確認を取る
- ただし、明らかな誤変換やタイポ（例：「寿命と廊下」→「寿命と老化」）は、確認せずに適切に修正する

---
> Source: [katsuki-ishii/obsidian](https://github.com/katsuki-ishii/obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
