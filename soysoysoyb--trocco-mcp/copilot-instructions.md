## trocco-mcp

> このドキュメントは、TROCCO MCP サーバーの開発・保守において守るべきルールをまとめています。

# TROCCO MCP サーバー開発ルール

このドキュメントは、TROCCO MCP サーバーの開発・保守において守るべきルールをまとめています。

## 1. 基本方針

- **外部 API への依存を最小限に**: TROCCO API の機能拡充に対して柔軟に対応できる実装を心がける
- **input は必ず検証する**: Zod スキーマを使用して、入力パラメータの検証を行う
- **エラーハンドリングの充実**: デバッグに必要な情報（URL、メソッド、ステータス）を含める

## 2. プロジェクト構成

### ファイル構成

```
src/
├── index.ts          # MCPサーバーのエントリポイント
├── constants.ts      # 定数定義
├── tools/            # 各ツールの実装
│   └── __tests__/    # ツールのテスト
└── utils/            # ユーティリティ関数
    ├── createErrorResponse.ts
    ├── requestTROCCO.ts
    ├── validateApiKey.ts
    └── __tests__/    # ユーティリティのテスト
```

### 環境変数

- `TROCCO_API_KEY`: TROCCO API キー（必須）
- 他の環境変数は追加しない（シンプルに保つ）

### 定数定義

`constants.ts`で管理される定数：

- `BASE_URL`: TROCCO API のベース URL
- `MCP_NAME`: MCP サーバー名
- `CONNECTION_TYPES`: 接続タイプの一覧（athena, bigquery, gcs 等）
- `NOTIFICATION_TYPES`: 通知タイプの一覧（email, slack_channel）
- `DEFAULT_API_LIMIT`: APIのデフォルトリミット値（200）

新しい定数を追加する場合は、複数のツールで使用される可能性があるものは`constants.ts`に集約する。

## 3. 開発ガイドライン

### コーディング規約

#### ツールの実装

新しいツールを追加する場合は、以下の構造に従う：

##### 一覧取得系ツールの場合

```typescript
const GetXxxsInputSchema = z
  .object({
    // パラメータの順序は必ず以下の通りにする
    fetch_all: z
      .boolean()
      .default(false)
      .optional()
      .describe("全件取得フラグ: trueの場合、countは指定不可"),
    count: z
      .number()
      .min(1)
      .optional()
      .describe("取得したいアイテム数: fetch_allがtrueの場合は指定不可"),
    // パスパラメータがある場合
    path_params: z.object({
      resource_type: z.enum(RESOURCE_TYPES).describe("リソースタイプ"),
    }),
    // クエリパラメータがある場合
    query_params: z
      .object({
        name: z.string().optional().describe("名前でフィルタリング"),
        start_time: z
          .string()
          .optional()
          .describe("開始時刻: YYYY-MM-DD HH:MM:SS形式"),
      })
      .optional(),
  })
  .refine(
    (data) => {
      return !(data.fetch_all === true && data.count !== undefined);
    },
    {
      message:
        "fetch_allがtrueの場合、countを同時に指定することはできません。いずれか一方を指定してください。",
    },
  );
```

##### 詳細取得系ツールの場合

```typescript
const GetXxxDetailInputSchema = z.object({
  // パスパラメータ（詳細取得では必須）
  path_params: z.object({
    resource_id: z.number().describe("リソースID"),
    // 複数のパスパラメータが必要な場合
    resource_type: z.enum(RESOURCE_TYPES).optional(),
  }),
});

export class XxxTool implements IMCPTool {
  /**
   * ツール名
   */
  readonly name = "trocco_xxx";

  /**
   * ツールの説明
   */
  readonly description = "XXX ref https://documents.trocco.io/apidocs/xxx";

  /**
   * パラメータの定義
   */
  readonly parameters = XxxInputSchema;

  /**
   * ツールを実行する
   * @param input 入力値
   * @returns 実行結果
   */
  async execute(input: z.infer<typeof XxxInputSchema>): Promise<{
    content: TextContent[];
    isError?: boolean;
  }> {
    // 実装
  }
}
```

**パラメータ設計の原則**:

- **path_params**: URLパスに含まれるパラメータ（例: `/api/users/{user_id}` の `user_id`）
  - 必須パラメータとして定義
  - リソースの特定に使用（ID、タイプなど）
- **query_params**: URLクエリパラメータ（例: `?name=test&limit=10`）
  - オプショナルパラメータとして定義（`.optional()`を付ける）
  - フィルタリングや検索条件に使用
- **describe**: パラメータの説明は、自明でない場合のみ付ける
  - 良い例: `start_time: z.string().describe("開始時刻: YYYY-MM-DD HH:MM:SS形式")`
  - 悪い例: `user_id: z.number().describe("ユーザーID")` （自明なので不要）
- **cursorパラメータ**: 含めない（内部的にページネーション処理で使用されるため）

**重要な注意事項**:

- `IMCPTool` の後に型パラメータを付けない（`implements IMCPTool` とする）
- コメントは必ず日本語で記載し、削除しない
- API ドキュメントの URL は必ず実際にアクセスして確認する
- 既存のツールと重複していないか確認する

#### エラーハンドリング

- `validateApiKey`の結果を必ず確認する
- API エラーは`createErrorResponse`を使用して統一的に処理する
- エラーメッセージは日本語で、ユーザーが理解しやすい内容にする

#### API リクエスト

- 単一のリクエストの場合は`troccoRequest`を使用
- ページネーションが必要な場合は`troccoRequestWithPagination`を使用
- レスポンスは JSON として返却し、型変換は行わない

#### テスト

各ツールのテストを作成する際は、以下の方針に従ってください：

##### 1. 実装の振る舞いをテストする

- 実装の詳細（内部で使用する関数など）ではなく、ツールの入出力の振る舞いをテストする
- ツールの`execute`メソッドが期待通りの結果を返すことを確認する

##### 2. 実装の重要な分岐点を網羅する

- 各ツールの実装を分析し、条件分岐やエラーハンドリングのパスを特定する
- すべての分岐が少なくとも 1 つのテストケースでカバーされるようにする
- **オプショナルパラメータも含めて、すべてのパラメータをテストする**
  - 特に`fetch_all`、各種 ID パラメータなど、実装に存在するすべてのオプションを網羅

##### 3. 外部依存を適切にモック化する

- API リクエストなどの外部依存は、実装で使用しているユーティリティ関数のレベルでモックする
- グローバルオブジェクト（fetch など）の直接モックは避け、抽象化されたレイヤーでモックする
- **プロジェクト全体で統一されたモック方法を使用する**
  - `validateApiKey`、`troccoRequestWithPagination`、`createErrorResponse`をモジュールレベルでモック

##### 4. エラーケースを必ずテストする

- API キーが無効な場合
- API からエラーレスポンスが返る場合（404、500 など）
- ネットワークエラーが発生する場合
- 入力パラメータが不正な場合（Zod バリデーションエラー）

##### 5. テストの独立性を保つ

- 各テストケースは他のテストケースに依存しない
- テスト実行順序に関わらず、同じ結果が得られるようにする
- `beforeEach`でモックをクリアし、クリーンな状態から各テストを開始する

##### 6. テストケースの記述を明確にする

- 日本語で「正常系」「異常系」を明記する
- 何をテストしているかが一目でわかるような説明的なテスト名を付ける
- **コメントは最小限にし、自明なものは書かない**
  - `expect(fn).toHaveBeenCalled()` のような自明な確認にコメントは不要

##### 7. 特殊なテストケースの実装方法

- **ページネーションのテスト（`fetch_all`）**
  - `fetch_all: true`を指定して全件取得が動作することを確認する
  - `fetch_all: true`と`count`を同時指定して、バリデーションエラーが発生することを確認する

### 新機能追加時のチェックリスト

- [ ] [API ドキュメント](https://documents.trocco.io/trocco-api/apidocs/overview)で正確なエンドポイントを確認
- [ ] エンドポイントに対応するドキュメントで仕様の詳細を確認
- [ ] 既存のツールと重複していないか確認（特に似た名前のツールに注意）
- [ ] ツールクラスを作成し、`IMCPTool`インターフェースを実装（型パラメータなし）
- [ ] 入力パラメータの Zod スキーマを定義
  - 一覧系: fetch_all → count → path_params → query_params の順序
  - 詳細系: path_params → query_params の順序
  - cursorパラメータは含めない
- [ ] 一覧系の場合: fetch_allとcountの相互排他バリデーションをrefineで実装
- [ ] 既存の実装を参考にして、正常系・異常系のテストを作成
- [ ] `src/index.ts`の TOOLS 配列に追加
- [ ] `npm test`でテストが通ることを確認
- [ ] `npm run build`でビルドが通ることを確認
- [ ] README.md の「利用可能なツール」セクションを更新
- [ ] 更新すべきことがあれば CLAUDE.md を加筆修正

## 4. 運用

### バージョン管理

バージョンは package.json で管理され、必要な場所（index.ts、requestTROCCO.ts）で直接読み込まれます。

以下のコマンドでバージョンを更新：

```bash
npm run version:patch  # 0.1.0 → 0.1.1
npm run version:minor  # 0.1.0 → 0.2.0
npm run version:major  # 0.1.0 → 1.0.0
```

### ドキュメント整理

#### README.md のツール一覧

ツール一覧は以下のリソース種別の順番で整理する：

1. ユーザー
2. チーム
3. リソースグループ
4. ラベル
5. 通知先
6. 接続情報
7. 転送設定
8. データマート定義
9. ワークフロー定義

各リソース種別内では以下の順番で記載：

1. 設定一覧（例：`trocco_get_users`）
2. 設定詳細（例：`trocco_get_user_detail`）
3. ジョブ一覧（例：`trocco_get_jobs`）
4. ジョブ詳細（例：`trocco_get_job_detail`）

各リソース種別は見出し（###）として分離し、その下に表形式でツールを記載する。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SoySoySoyB) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
