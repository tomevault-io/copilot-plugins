## unwrapped

> - 入力：PEファイル（`.exe` / `.dll`）

# unwrapped

## プロジェクト概要

- 入力：PEファイル（`.exe` / `.dll`）
- 言語：Rust
- `colored` クレート（カラー表示）

---

## 実装ルール

- ステップを1つ実装したら必ず止まり、ユーザーに確認してから次へ進む
- Rustのコードを修正したら `cargo fmt`, `cargo clippy` を実行してください
- 作業対象のファイルのみ Read する。他モジュールは必要な場合のみ参照する

---

## 実装ステップ

✅ ステップ 1〜15 完了済み

| # | ステップ名 | 実装内容 | 確認内容 |
|----|-----------|---------|---------|
| 16 | Section Entropy 計算 | エントロピー計算 / `[normal]` / `[^ elevated]` / `[!! HIGH]` 判定 | セクションごとのエントロピー値と判定結果をカラー表示 |
| 17 | CLIフラグの実装 | `-h` / `-S` / `-s` / `-r` / `-d` / `-n` / `-a` 実装 | 各フラグで対応する項目のみ表示される |

---

## 出力仕様

### 出力共通ルール

- 左端のOffsetは同じ数値が並ばないようにデザインする
- 左端のOffsetは上から降順で並ぶようにデザインする
  - 構造的にOffsetが飛ぶ場合は左端のOffsetは出力せず子要素でOffsetがわかるようにする
- ラベルやヘッダー名は基本的に、Microsoftの公式ドキュメント「PE Format」に記載のある名称を使う
  - ない場合はwinnt.hの構造体に記載のある名称を使う

### 出力例

出力例は `docs/output_example.txt` に記載してあるが、必要な場合のみ Read する。

あくまで、出力例なので必要な情報がある場合は追記してかまわない。

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ky0ta168) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
