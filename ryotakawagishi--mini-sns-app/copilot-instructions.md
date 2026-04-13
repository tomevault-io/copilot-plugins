## mini-sns-app

> - アプリケーションのドメインロジックを扱う処理は、Modelに実装する


# Modelの実装規約

## 基本方針

- アプリケーションのドメインロジックを扱う処理は、Modelに実装する
- Modelは `app/models` に配置する
- ファイル名はクラス名のスネークケースとする
- データベースアクセスするテーブルがある場合は、RailsのActiveRecord(実際にはApplicationRecord)を継承したクラスとする
- テーブルがない場合は、PORO（Pure Old Ruby Object）かイベント型モデルを選択する
- 複数のモデルに関連した一連のイベントをまとめる場合は、イベント型モデルとする
- イベントではない場合は、PORO（特に継承をしないクラス）とする
- validationが不要な場合で、イベント型モデルにする理由がない場合はPOROを採用する。
  - I/Fが簡素化でき、メモリなどのコストも低いため。

## イベント型モデル

- `ApplicationEventModel` を継承して作成する
- クラス名は、イベント名の名詞とし、末尾に `Event` をつける
  - 例： `MoveEvent` 
- I/FはRailsのApplicationModelに準ずる（https://railsguides.jp/active_model_basics.html）
  - 例： `valid?`, `save`

## validation

- カスタムvalidatorの実装はなるべく控え、既存のvalidator(numericalityなど)を使う

## トランザクション

1. データの整合性を保つために、関連するデータの更新は必ずトランザクション内で行う

   ```ruby
   ApplicationRecord.transaction do
     # データの更新処理
   end
   ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RyotaKawagishi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
