## testbedui-plugins

> このプロジェクトでは、`testbed-core` が提供する MCP サーバー（`localhost:11452`）と通信して、デバイス情報を取得したり操作したりします。

# エージェント実行のためのプロジェクト知見

## 1. TestBed Core (MCP サーバー) の起動と停止
このプロジェクトでは、`testbed-core` が提供する MCP サーバー（`localhost:11452`）と通信して、デバイス情報を取得したり操作したりします。
エージェントがツールを正常に利用するためには、事前に `testbed-core` デスクトップアプリが起動している必要があります。

### 起動方法
別ターミナルで `testbed-core` リポジトリのディレクトリに移動し、以下のコマンドを実行して起動します。

```bash
cd ../testbed-core
./gradlew :composeApp:run
```

### 停止（キル）方法
二重起動を防ぐため、またはポート競合（`Address already in use` 等）が発生した場合は、11452 番ポートを使用しているプロセスを特定して強制終了します。
エージェントが自動作業する前にも、このコマンドでクリーンアップしてから起動すると安全です。

```bash
# 11452ポートを使っているプロセスをキル
lsof -t -i :11452 | xargs kill -9
```

## 2. ドキュメント (MDFPP-CC) の参照
`testbed-core` と同様に、このリポジトリにも `docs`（実体は `KVVat/mdfpp-docs.git`）が Git サブモジュールとして追加されています。
CC (Common Criteria) などのセキュリティドキュメントを参照して実装を進める場合は、`docs/MDFPP-CC/` 以下の Markdown ファイルを直接読み込んでください。

サブモジュールが空の場合や最新化したい場合は以下を実行します。
```bash

## 3. ドキュメント駆動のテスト開発（至上命題）
このプロジェクト（`testbedui-plugins`）における最大の指針は、**セキュリティ要件ドキュメントに厳格に従ったテスト実装**です。
- LLMエージェントは自律的にテストを記述する際、必ず事前に `docs/MDFPP-CC/` 以下のMarkdownファイルから該当する機能要件や評価基準（Test Case）を読み込んでください。
- 実装するJUnitテストは、ドキュメントに定義されたPass/Fail判定基準に合致している必要があります。

## 4. テスト開発・実行ワークフロー
LLMエージェントがテストを開発・実行する際は、以下のワークフローに従います。詳細は [reference/TOOL_USAGE.md](reference/TOOL_USAGE.md) を参照してください。

1. **ドキュメント参照**: `docs/` から要件を取得
2. **コード生成**: `test-sample` 等にテストコードを追加。ログ出力は [reference/TEST_CONVENTIONS.md](reference/TEST_CONVENTIONS.md) の規約（`logi`, `logp` 等）に従う。
3. **ビルドとロード**:
   - `./gradlew :test-sample:jar` でビルド
   - `junit_test_reload` でテストを反映
4. **テスト実行と確認**:
   - `junit_test_execute` で実行開始
   - `junit_test_receive` を定期的に呼び出し、リアルタイムログと最終結果を取得

## 5. 実装済みツールと仕様
以下のツールが `testbed-core` 側で実装されており、自律的なテスト実行が可能です。

- **`junit_test_reload`**: テストJARを再読み込み。
- **`junit_test_list`**: ロードされたテストの一覧を返却。
- **`junit_test_execute`**: クラス名とメソッド名を指定して実行。
- **`junit_test_receive`**: 実行中のログ（`status: "Running"`）や完了後の結果（`status: "Finished"`, `Pass/Fail`, `stacktrace`）を構造化されたJSONで取得。

> **Note**: 実行中のリアルタイムログ取得に対応しているため、長時間（2分以上）のテストでも `junit_test_receive` をポーリングすることで進捗を監視できます。

## 6. リソース配置と配布パッケージング規約（厳守事項）
- **テスト用外部リソース（大容量ベクターデータ、テスト用バイナリ等）の配置場所**:
  - すべて **プロジェクトルート直下の `resources/`**（例: `resources/kernelacvp/`, `resources/revocation/`, `resources/x509extensions/`）に配置・管理すること。
  - **`test-sample/src/main/resources/` に大容量ファイルやテストデータを配置してはならない**（プラグイン JAR の不要な肥大化を防止し、プロジェクト構成を人間にとって明確にするため）。
- **配布用 ZIP パッケージの生成**:
  - 配布用パッケージ（`dist/test-sample-plugin.zip`）は、プロジェクトルート直下の `resources/` とビルドされたプラグイン JAR（`plugins/test-sample.jar`）から生成される。
  - リソースの追加・更新時は必ずルートの `resources/` に対して行い、`./gradlew :test-sample:packageZip` でパッケージングすること。

---
> Source: [KVVat/testbedui-plugins](https://github.com/KVVat/testbedui-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
