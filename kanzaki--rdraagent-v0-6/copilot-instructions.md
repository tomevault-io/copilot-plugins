## rdrarules

> フォルダーを作成するときは既にフォルダーが存在するか確認し、存在する時はそのフォルダーを使用する

# Agentに対するルール
フォルダーを作成するときは既にフォルダーが存在するか確認し、存在する時はそのフォルダーを使用する
出力するファイルと同じ名前のファイルが存在する場合は、そのファイルを上書きする
【重要】が付けられたものは必ず実行する

# program
作成するプログラムは以下の３種類に限る
- ReScript
- javascript
- html
- css:cssはhtmlに含める
 
# GUIライブラリはshadcnを利用
https://ui.shadcn.com/docs/components

## テキストの文字列コードは「UTF8」を使う

# tsvファイルに出力するときの決まり事
- タブ区切り文字に変換して出力する
- tsvファイルは１行目をヘッダー行としてカラム名を出力する

# folder file design intent
## 「0_RDRAZeroOne/phase1」「0_RDRAZeroOne/phase2」「0_RDRAZeroOne/phase3」「0_RDRAZeroOne/phase4」
- 生成物を保持するフォルダー
- Phase1-Phase4:フェーズ毎の成果物を保持するフォルダー
## 「1_RDRA」フォルダー
- 要件の生成物を保持するフォルダー
- フェーズ３とフェーズ４の生成物を保持するフォルダー
- 関連データ.txt:RDRAGraphのためのデータ
## 「2_RDRASpec」フォルダー
- 仕様の生成物を保持するフォルダー
- 仕様ファイルを保持するフォルダー
## 「RDRA_Knowledge/0_RDRAZeroOne」フォルダー
- Phase1-4:初期要望から４フェーズでRDRASheetを生成する手順を示す

## 「RDRA_Knowledge/1_RDRA」フォルダー
要件定義手法の「RDRA」の考え方をまとめたファイルを含む
- RDRA.md:RDRAの考え方をまとめたファイル
- RDRASheet.md:RDRA定義をSpreadsheetで行う各シートのフォーマットの説明、各シート内のモデルは「RDRA.md」で定義したモデルに対応する
- RDRAGraph.md:RDRAGraphのデータ構造「関連データ.txt」の説明
## 「RDRA_Knowledge/2_RDRASpec」フォルダー
- 21_論理データ生成.md:RDRA定義から論理データを生成する方法を説明したファイル
- 22_画面生成.md:ユースケースと画面の関係から画面のUI定義を生成する方法を記述したファイル
- 22_ビジネスルール生成.md:ビジネスルールモジュールを定義する方法を記述したファイル
- specValidation.md:仕様の妥当性検証方法を記述したファイル
## 「2_RDRASpec/Validation」フォルダー
妥当性検証などの結果を保持する

## 「helper_tools」フォルダー
ファイルの削除やコピーなどのツールを保持する

# このプロジェクトの使い方
- RDRA定義を生成するために「menu.js」でメニューを表示する
- メニューの番号を順番に入力しRDRA定義を生成する
- 再作成するファイルは、そのファイルを削除してから実行するので、AIAgentは生成対象のひな型を生成しないでください
- 

---
> Source: [kanzaki/RDRAAgent_v0.6](https://github.com/kanzaki/RDRAAgent_v0.6) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
