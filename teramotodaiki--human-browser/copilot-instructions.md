## human-browser

> このファイルは、このリポジトリで作業するエージェント向けの実務ルールです。

# AGENTS.md

このファイルは、このリポジトリで作業するエージェント向けの実務ルールです。

## 1. 目的

- 目的は、`human-browser` を通して「ログイン済み Chrome」を安定して操作できること。
- 中核は `snapshot -> ref -> action` の決定論的フローと、切断時の復旧性。
- まず観測可能性と信頼性を優先し、機能追加は最小限にする。

## 2. リポジトリ構成

- `src/cli/human-browser.ts`: CLI エントリポイント
- `src/daemon/*`: daemon 本体（HTTP + extension bridge）
- `src/shared/*`: 共通型・設定・エラー・snapshot整形
- `extension/*`: Chrome Extension (Manifest V3)
- `vendor/agent-browser/*`: snapshot/ref の vendor ロジック
- `docs/*`: CLI仕様と protocol
- `test/*`: unit / integration

## 3. 開発・実行コマンド

- 依存導入: `npm install`
- CLI をローカルコマンドとして使う: `npm link`
- 設定初期化: `human-browser init`
- daemon 起動: `human-browser daemon`
- WS/Token 確認: `human-browser ws`
- 状態確認: `human-browser --json status`
- テスト: `npm test`

## 4. 作業原則

- YAGNI を守る。必要になるまで抽象化・拡張を増やさない。
- 解決策の前に根本原因を特定する（観測 -> 切り分け -> 修正）。
- このセッションで作成していない無関係ファイルは触らない。
- 失敗を隠すフォールバックを入れない。
  - 環境変数未設定時の暗黙デフォルト
  - サーバーエラー時のモック返却
  - E2E例外時のスキップ
- 仕様変更を伴う変更では、`README.md` / `docs/*` を必要最小限で同期する。

## 5. CLI と API の扱い

- 互換性を壊す変更（コマンド名、引数、エラーコード、JSON 形状）は慎重に扱う。
- `status` は接続状態確認用。機密情報は原則 `ws` で明示的に取得する。
- `click` / `fill` は次の2系統をサポートする前提:
  - selector 指定（例: `#email`）
  - ref 指定（`@e1` / `ref=e1` / `e1` + `--snapshot <id>`）

## 6. 変更時の検証チェックリスト

最低限、以下を通すこと:

1. `npm test`
2. `human-browser --json status`
3. 変更対象に応じたスモークテスト
   - CLI/daemon変更: `tabs`, `snapshot` の往復
   - extension変更: popup で接続確認、必要に応じて `click/fill`

## 7. daemon 再起動ルール（重要）

`src/daemon/*` または daemon 起動経路（`src/cli/human-browser.ts` の daemon 関連）を変更した場合は、作業完了時に daemon を再起動すること。

- 再起動時は、起動中プロセスを停止し、同じ起動引数（特に `--config`）で起動する。
- `launchd` 管理の場合は `kickstart -k` を使って再起動する。
- 再起動後は `human-browser --json status` で疎通を確認する。

### 参考コマンド（launchd 管理時）

```bash
launchctl kickstart -k gui/$(id -u)/com.teramotodaiki.human-browser
human-browser --json status
```

## 8. extension 変更時の運用

- `extension/*` を変更したら、`chrome://extensions` で対象拡張を Reload する。
- 接続状態が落ちたら popup の `Reconnect` を押す。
- アイコン/表示状態を変える変更では、connected/disconnected の両方を確認する。

## 9. セキュリティ

- daemon はローカル bind（`127.0.0.1`）前提。
- token は機密として扱い、ログやIssue本文に貼らない。
- `~/.human-browser/config.json` は上書き前提を明示し、既存運用を壊さない。

## 10. PR / コミット方針

- 1コミット1意図を基本に、メッセージは変更意図が分かる形にする。
- 大きな変更は「何を変えたか」だけでなく「なぜ必要か」を説明可能にする。
- 無関係差分を混ぜない。

## 11. セキュリティ方針（公開運用）

- daemon の bind は `127.0.0.1` 固定を維持し、外部公開アドレスを許可しない。
- token はデフォルトで表示しない（必要時のみ明示フラグで表示）。
- 拡張の daemon 接続先は `ws://127.0.0.1:<port>/bridge` のみ許可する。
- snapshot で入力値（`input.value` などの実データ）を収集しない。
- 拡張権限は最小化を維持し、不要な `host_permissions` / `permissions` を追加しない。

---
> Source: [teramotodaiki/human-browser](https://github.com/teramotodaiki/human-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
