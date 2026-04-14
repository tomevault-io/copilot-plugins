## codexrules

> この `AGENTS.md` は `/home/myokoym/.codex/rules` を編集するときだけ適用する。通常利用時のフォールバック方針は `README.md` にある `~/.codex/AGENTS.md` 追記テンプレートを使う。

# rules リポジトリ編集ルール

この `AGENTS.md` は `/home/myokoym/.codex/rules` を編集するときだけ適用する。通常利用時のフォールバック方針は `README.md` にある `~/.codex/AGENTS.md` 追記テンプレートを使う。

## 構成ルール
- `default.rules` は言語非依存の共通コア専用に保つ。
- 言語依存コマンドは `node.rules`、`ruby.rules` のような別の `*.rules` に分ける。
- 一時回避コマンド、個人環境依存コマンド、長い shell wrapper は汎用 rules に入れない。

## 記述ルール
- 各 `prefix_rule` には `match` と `not_match` を付ける。
- 前方一致で広がりすぎるコマンドは、引数まで含めて限定する。
- 危険操作は `allow` にせず、`prompt` か未マッチに寄せる。
- `bash -c`、`sh -c`、`zsh -lc`、長い環境変数付きラッパーは汎用 rules に入れない。

## 確認ルール
- 変更後は `codex execpolicy check --pretty --rules <file> -- <command>` で境界を確認する。
- 許可例だけでなく、非許可にしたい代表例も確認する。
- `git diff` を見て、意図しない広い許可が入っていないか確認する。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/myokoym) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
