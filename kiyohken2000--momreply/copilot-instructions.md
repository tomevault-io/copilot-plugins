## momreply

> 母から届く iMessage に AI が返信を生成し、自動送信または確認のうえ送信する macOS メニューバーアプリ。

# MomReply

母から届く iMessage に AI が返信を生成し、自動送信または確認のうえ送信する macOS メニューバーアプリ。

仕様書: `docs/spec.md` — 実装前に必ず該当セクションを読むこと。

## 絶対に守るルール

1. **chat.db には絶対に書き込まない。** 接続は必ず `SQLITE_OPEN_READ_ONLY`。
   書き込み可能な接続を開くコードを書いたら、それは即座にバグ。
2. **テスト送信は自分の Apple ID 宛にのみ行う。** 母のハンドル宛に
   テストメッセージを送るコードを書かない・実行しない。
3. **本番の chat.db を破壊的に扱わない。** コピーして実験する場合も
   `chat.db-wal` と `chat.db-shm` を必ず同時にコピーする。
4. **API キー・メッセージ本文を、ログ・コミット・設定ファイルに書かない。**
   API キーは Keychain のみ。本文は app.db のみ。
5. **フェーズのスコープを超えない。** 現在のフェーズで求められていない
   ファイルを先回りして作らない。

## 実装の前提

- `message.text` はほぼ常に NULL。本文は `attributedBody`（typedstream）から取る。
  これを忘れると「メッセージが1件も取れない」状態になる。
- 送信メッセージは `handle_id = 0` になることがある。`handle` テーブル経由の
  JOIN ではなく `chat_message_join` → `chat` 経由で取得する。
- AppleScript は送信失敗を報告しないことがある。終了コード 0 を信じず、
  chat.db を見て送信結果を検証する。
- 外部クレートの API は記憶で書かない。`cargo doc` や実際のソースで
  現行シグネチャを確認してから使う。特に `imessage-database`。

## 進め方

- 不明点は推測で埋めず、質問する。
- 大きな変更の前に方針を1〜3行で述べて、承認を得てから書く。
- 動作確認は実際に実行して行う。「動くはず」で完了報告しない。
- 各フェーズの受け入れ基準（仕様書 13章）を、実行結果とともに報告する。

## コマンド

```bash
cargo run -p momreply-cli      # Phase 0 の検証 CLI
cargo test
cargo tauri dev                # Phase 2 以降
```

---
> Source: [kiyohken2000/momreply](https://github.com/kiyohken2000/momreply) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
