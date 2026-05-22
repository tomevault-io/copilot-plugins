## resobotgw

> 本プロジェクトにおけるエージェント開発・運用の最小規約。優先順位は以下の通り（衝突時の解決順）。

# AGENTS

本プロジェクトにおけるエージェント開発・運用の最小規約。優先順位は以下の通り（衝突時の解決順）。

## 優先順位（衝突解決）

- MUST: 静的安全・正確性 > ユビキタス言語に整合する命名（意図の可読性） > 変更容易性（小さく凝集・最小表面） > シンプルさ（YAGNI/DRY） > 一貫性（既存スタイル） > 後方互換性。
- SHOULD: 既存との一貫性よりも、意図の明確さと変更容易性を優先する。
- SHOULD: 互換性維持が複雑性を増す場合は、破壊的変更を選び、PR の Migration に記載する。

## 位置づけ

- この文書はプロジェクト規約。必要に応じて継続的に整理・改善する（自己適用）。
- 本規約の原則はコードだけでなくドキュメントにも等しく適用する。

## 原則（規範）

- MUST: 静的に安全で見通しの良いコード（型/不変条件/契約）
- MUST: 不要な Optional/nullable を禁止（Non‑Null 既定）
- MUST: 命名を最重要視（ユビキタス言語）
- MUST: YAGNI/DRY を徹底
- MUST: 設計ファースト（最初に短い設計ノート）
- MUST: 小さく凝集したコードベース
- MUST: 最新ツールセット・言語仕様を優先
- SHOULD: Evans DDD に親和
- SHOULD: 後方互換性よりシンプルさ
- MUST: 依存はコンストラクタ注入

### 本プロジェクト追加原則

- MUST: asyncio を徹底し、アプリケーションループは `runner.run()` を中心に据える。
- MUST: `openai` パッケージを直接利用しない（Agents SDK のアダプタ経由）。
- MUST: dynamic import と `try` で囲った import を禁止する（失敗は即時に顕在化）。
- MUST: Protocol の使用を禁止する（抽象基底クラス/具象型で表現）。
- MUST: 存在が確認できない API を推測で利用しない。必ず公式ドキュメント/コードで確認してから実装する。
- MUST: 薄すぎる抽象化層を設けない（YAGNI）。コードベース簡素化のため、必要ならライブラリへのロックインを許容する。
- MUST: 設定は Pydantic Settings を用いてパース/検証し、Non-Null を強制する（`.env` を含む）。

## 変更容易性ゲート（PR チェック）

- MUST: Public API/可視性は最小
- MUST: 破壊的変更は PR に `Migration`
- MUST: 非自明な変更は設計要旨 or ADR リンク
- MUST: 変更は単一の理由に収束（凝集）
- MUST: YAGNI/DRY を満たす
- MUST: Optional/nullable の導入理由と扱いを明記
- SHOULD: 依存方向を悪化させない
- SHOULD: 命名が意図・制約を十分に表現
- SHOULD: 小 PR で読みやすい差分
- SHOULD: テストは意図を保障し結合を過剰にしない
- SHOULD: ドキュメント更新を伴う

### Migration（Breaking）

- 目的: MCP は Agents SDK の機能を直接利用し、薄い独自抽象（`resobot_gw.mcp.Bridge`）を排除してシンプルさ/YAGNIを徹底する。
- 影響範囲: `src/resobot_gw/mcp/bridge.py` を削除。将来的に MCP 連携は Agents SDK の `HostedMCPTool`/`MCPServer` 等を直接利用する。
- ロールバック: 当該ファイルを復帰し、`AGENTS.md` の公開境界/配置ポリシーを元に戻す。

## 設計ファースト（プロセス）

- MUST: 着手前にミニ設計ノート（最大10分、PR 貼付）
- MUST: 実装で逸脱したら設計ノート更新
- SHOULD: 小変更でも設計要旨を数行で記す

## CI の最小基準 / Definition of Done（既定）

- MUST: すべての CI チェックがグリーン（`format`/`lint`/`typecheck`/`test`/`mdfmt-check`）。
- MUST: カバレッジのしきい値を満たす。満たさない場合は作業を止め、指示を求める（勝手にしきい値を下げない）。
- MUST: PR チェックリスト（含む `Migration`）に合格。
- MUST: テストをグリーンにするためにプロダクションコードを変更しない。必要な場合は作業を止め、指示を求める。
- SHOULD: テスト側の改善で解決可能な失敗は最小変更で対処。ただし、テストファイルの変更のみでグリーン化が困難な場合は、チェック完了前に指示を求める。
- SHOULD: あらかじめ別途の指示がある場合はそれに従い、独断でチェック完了（グリーン化）を進めない。

## コミットルール

- MUST: Conventional Commits + gitmoji
- 形式: `type(scope)?: gitmoji Message`（1行）

## 命名・言語

- MUST: ユビキタス言語を優先（用語集に追記）
- コード（識別子/コメント/Docstring）は英語。レポートやチャットは日本語。

## Optional/Null ポリシー（最小）

- MUST: 非Null 既定。導入は「不在に意味がある」場合のみ。
- MUST: 導入時は不在の意味と扱いを設計に明記。
- SHOULD: コレクションは空で返し、Null は使わない。

## 機密情報

- MUST: コード/ログ/PR に秘密を含めない。環境変数/シークレット管理を使用。

## 本プロジェクト固有の補足

- メインアーキテクチャ: OpenAI Agents SDK（依存導入は API 確認後に実施）。
- 言語/ツール: Python 3.13、Hatch、ruff、basedpyright、pytest。
- MCP は Agents SDK の MCPServerStdio を利用して StdioMcpd に接続する。Resonite 側は ResoBotMCP を MCP サーバとして公開し、GW は Agents SDK 経由で利用する。

### 公開境界（最小）

- CLI: `resobot-gw run [--dry-run] [--log-level ...]`
- 環境変数: `OPENAI_API_KEY`（必須・Non‑Null）
- MCP: Agents SDK の MCP 機能（`agents.mcp.MCPServerStdio`）で StdioMcpd に接続。独自の `resobot_gw.mcp.Bridge` 抽象は廃止。

### テスト方針（最小）

- ユニット: 純粋ロジック（config、bus、調停ポリシー）
- 結合: ブリッジのフェイクと調停の相互作用
- スモーク: CLI/ランタイム（dry-run、構成エラー）

### 非機能 / 制約（再掲）

- Non‑Null 既定。Optional はドメイン上の不在のみ許可。
- 構造化ログとレイテンシ計測を早期導入。
- asyncio を徹底し、ループの中心は `runner.run()`。
- dynamic import と try-wrapped import を禁止。
- Protocol は禁止（抽象基底クラスを使用）。

## OS / シェル指針（最小）

- MUST: PowerShell で文字化けが発生する場合、読取りに限り WSL2 の bash を用いて内容を確認してよい（破壊的操作は PowerShell 側で行う）。

## 用語集（英→日）

- Agent: エージェント（自律タスク実行主体）
- Gateway (GW): ゲートウェイ（連携と調停の中核）
- Resonite: Resonite（対象プラットフォーム）
- ResoBotMCP: Resonite Bot の MCP サーバ（Agents SDK `MCPServerStdio` から接続）

## ToDo

ここに現在進行中、着手予定のタスクをすべて記述する。追加、分割は直ちに反映。現コードベース起点を徹底して完了タスクは直ちに削除。

- [Now] MCP Integration (StdioMcpd): Agents SDK の `MCPServerStdio` を用いて StdioMcpd に接続。
  - 設定: `MCP_STDIO_COMMAND`（必須）、`MCP_STDIO_ARGS`（任意、空白区切り）。
  - 実装: Config に追加し、OpenAI ランナーで `Agent(mcp_servers=[MCPServerStdio(params=...)])` を構成。
- [Now] Observability: ランナー呼び出しのレイテンシ計測ポイントを追加。
- [Now] GW Boundary Mock: 並列エージェント連携を検証するための外部境界モックを確立。
  - `docs/GlobalWorkspace_BoundaryMock.md` に設計を記述し、テスト用の `MockWorld` を提供。
- [Next] Agents Orchestrator: 並列実行・調停の最小骨格（TaskGroup 等）。
- [Next] Tests: 結合（MCP Stdio フェイク）/スモークを GW Boundary Mock を用いて拡充。
- [Next] Docs: README に MCP 設定と実行例を追記。

【Done】

- Docs: OpenAI Agents ドキュメント確認（API/名前解決）—完了（`from agents import Agent, Runner`／`Runner.run()`）

---
> Source: [esnya/ResoBotGW](https://github.com/esnya/ResoBotGW) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
