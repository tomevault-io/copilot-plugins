## fino-filing

> - Do not invent requirements

# AGENTS.md

## Core Principles

- Do not invent requirements
- Prefer existing docs over assumptions
- docs is exist in ./docs/\*\*
  there are scenario samples which this repository implement as,
  architecture design model,
  basical design rule etc.

## Authority Order

1. AGENTS.md
2. docs/
3. User input

## Allowed Actions

- Modify code under src/
- Update docs under docs/
  don't write too much keep simple order
  separet by interest of coding or design
- Add tasks when plan next actions under .tasks and numbaring is required
  ex. .tasks/1_create_domain_model/1_hoge.md

## Forbidden Actions

- Changing public API without approval

# コーディングルール

## コメントルール
- packageのコメントは不要
- 公開用クラスのコメントは英語で記載する。内部ロジックや制限等の記載はuserには不要の情報のため載せない
- テストのコメントはテストクラスのコメントは英語で"Class_name.Method_name Test"として、テストケースのコメントは日本語で、そのケースの観点を記載してください

## 型ヒント

- 全関数・全メソッドに型ヒントを必須とする
- public APIは特に厳密に型を記述する

---

## テスト

- pytest を前提とする
- 新しいユースケースを追加する場合、対応するテストを追加すること
- adapters層のテストは統合テスト寄りでも可
- 同一ファイルでは対象クラスを最外周の `Test*` クラスに寄せ、そのメソッド群ごとにネストした内部 `Test*` クラスで区切ると見通しがよい

---

## ロギング

- logging モジュールを使用する
- printは禁止
- loggerは各モジュールで以下の形式で定義する

```python
import logging
logger = logging.getLogger(__name__)
```

Agent駆動開発ルール

Agentによる開発では以下を厳守する。

実装単位
• クラス単位
• ユースケース単位

開発フロー 1. 設計を文章で明確化 2. 実装 3. テスト

この順序を必ず守る。

⸻

クラス設計ルール
• 各クラスには「責務を示すコメント」を必ず記述する
• publicメソッドには簡潔なdocstringを書く
• parameterやresponseの説明はコメントに含めない

公開範囲ルール
• 基本的なクラスは内部利用前提とする
• 必要なもののみ公開するために、`src/fino_filing/__init__.py`で指定してpackage rootから呼び出し可能な状態にする
• **all** を用いて公開対象を明示する

---
> Source: [fino-dev/fino-filing](https://github.com/fino-dev/fino-filing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
