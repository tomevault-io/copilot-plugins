## comfyui-promptpalette

> ComfyUI-PromptPaletteは、マウス操作で素早いプロンプト編集を可能にするComfyUI向けのカスタムノードです。

# ComfyUI PromptPalette

## プロジェクト概要

ComfyUI-PromptPaletteは、マウス操作で素早いプロンプト編集を可能にするComfyUI向けのカスタムノードです。
チェックボックスによるフレーズの有効/無効切り替えや、ウェイト調整ボタンといった、インタラクティブなプロンプト編集機能を提供します。

## 使用言語

- Python
- JavaScript

## コーディング規約

- コードコメントは英語
- ログ出力は英語

## コミット規約

- コミットメッセージは英語

## インストール方法

- 標準的な ComfyUI カスタムノードの導入手順に従い、`custom_nodes` ディレクトリへ配置する。
- 追加のセットアップや依存関係は不要。

## ビルド

- 不要

## プロジェクト構成

ComfyUIのカスタムノード構成に従う。

- `__init__.py`: ノードのマッピングと Web ディレクトリをインポート/エクスポートするエントリポイント
- `nodes.py`: テキスト入力を処理するバックエンド（Python）
- `web/`
  - `index.js`: フロントエンドのエントリポイント（JavaScript）
  - `canvas_ui.js`: Nodes 1.0用のフロントエンド（Canvas描画）
  - `dom_ui.js`: Nodes 2.0用のフロントエンド（DOM描画）
  - `ui_utils.js`: UI関連の共通ロジック
  - `line.js`: 各行のテキスト処理の共通ロジック
- `pyproject.toml`: Comfy Registry 公開のためのメタデータ

## 機能

### ノードの入力スロット

- `prefix` slot (Optional)
  - type: Multiline STRING

### ノードのウィジェット

- `text` widget: メインテキストエリア
  - type: Multiline STRING 
  - default: ""
- `delimiter` widget: 区切り文字の設定
  - type: COMBO
  - options:
    - "comma"
    - "space"
    - "none"
  - default: "comma"
- `line_break` widget: 改行有無の設定
  - type: BOOLEAN
  - default: true

### ノードの出力

メインテキストエリアのテキストを以下の通り整形した文字列を出力する。

- 空行を削除
- コメントを削除
  - `//` で始まるコメント行を削除
  - 行内の `//` 以降の文字列は後続コメントとして削除
- `delimiter` の設定に応じて行の末尾に文字列を追加
  - "comma": ", " を追加
  - "space": " " を追加
  - "none": 何も追加しない
- `line_break` の設定に応じて出力文字列の改行を変更
  - true: 改行をキープ
  - false: 改行を削除
- `prefix` slot の入力がある場合、prefixを出力文字列の先頭に追加
  - `line_break` の設定に応じて結合時の改行有無を変更
    - true: 改行ありで結合
    - false: 改行なしで結合
  - 出力文字列が空の場合は結合せずprefixを出力文字列とする

### 編集モードと表示モード

- 編集モード (Edit mode) と表示モード (Display mode) という2種類のモードがある
- ノード下部の `Edit` / `Save` ボタンで編集モードと表示モードをトグル
- デフォルトは表示モード

### 編集モード時のUI

- 以下の要素を上から順に並べる
  - `text` widget
  - `delimiter` widget
  - `line_break` widget
  - `Save` ボタン
- レイアウト
  - `delimiter`, `line_break`, `Save` をノード下端に寄せる
  - 残りの上部領域に `text` widgetを表示

### 表示モード時のUI

- Nodes 1.0の場合、Canvasで UI を描画
  - クリック処理は座標ベースの `clickableAreas` により判定
- Nodes 2.0の場合、DOMでUIを描画
- 表示モードでは以下のウィジェットは非表示
  - `text` widget (メインテキストエリア) 
  - `delimiter` widget
  - `line_break` widget
- メインテキストエリアにテキストが書かれている場合:
  - メインテキストエリアの各行のテキストを元に、後述する「表示モード行」を表示
  - ノード下部に `Edit` ボタンを下寄せして表示
- メインテキストエリアが空の場合:
  - ノードの中央に "No Text" と表示
  - ノード下部に `Edit` ボタンを下寄せして表示
- ノードが折りたたまれている場合:
  - 何も描画しない

#### 用語定義

- `表示用テキスト`: 各行のテキストから、行頭コメントプレフィックスおよびウェイト値、ウェイトの括弧、ウェイトのコロンを除去したテキストを表示用テキストとする。後続コメントは表示用テキストに含まれる。
- `フレーズテキスト`: 各行のテキストから、行頭コメントプレフィックスおよび後続コメント、ウェイト値、ウェイトの括弧、ウェイトのコロンを除去したテキストをフレーズテキストとする。

#### 表示モード行

- 行ごとに「チェックボックス」「表示用テキスト」「ウェイト値」「-ボタン」「+ボタン」を描画
  - 「チェックボックス」「表示用テキスト」は左寄せ
  - 「ウェイト値」「-ボタン」「+ボタン」は右寄せ
- フレーズテキストが空文字や空白文字になる場合、その行は空行として表示する。
  - 例えば行頭コメントプレフィックスのみの行や、行頭コメントプレフィックスと後続コメントのみの行などがこれに該当

#### チェックボックスによる行頭コメントの切り替え

- チェックボックスがオフの場合、その行に行頭コメント `//` を付ける
- チェックボックスがオンの場合、その行の行頭コメントを削除する

#### 表示用テキスト

- 表示用テキストがウェイト値や±ボタンと重なる場合、重なる部分を省略表記する
  - Nodes 1.0 (Canvas) の場合: Canvasの `clip()` を使用し、表示用テキストをクリッピング
  - Nodes 2.0 (DOM) の場合: 三点リーダー `text-overflow: ellipsis` で省略表記
- ウェイトが 1.0 以外の場合、表示用テキストを太字にする

#### ウェイト値

- `(phrase:1.5)` のようなウェイト表記を解析し、ウェイト値を表示
- ウェイトが 1.0 の場合はウェイト値を非表示
- ウェイト値は小数第一位まで必ず表示
  - 小数第二位がある場合は小数第二位も表示
  - 小数第三位以降は四捨五入

#### ウェイト値調整用の-ボタンと+ボタン

- ± ボタンで 0.1 刻みの増減
- ウェイトの範囲は 0.1-2.0
- 手入力で範囲外の値（例: 2.5）が設定されている場合:
  - 表示: そのままの値を表示
  - ± ボタン押下後: 範囲内にクランプ
- ウェイトの値を1.0に戻すと、ウェイトの括弧やウェイト値を外す
- 行頭コメント行や後続コメント付き行でも増減可能
  - ウェイト変更後も行頭コメント行や後続コメントを残す

### ComfyUIテーマ連携

- 文字色やボタン色、チェックボックス色をComfyUIテーマ色に合わせる
  - ライト/ダーク両テーマに対応
- ComfyUI のCSS変数から色を取得
  - パフォーマンスのため色情報はキャッシュ
  - 3桁の16進カラーの場合、6桁の16進カラーに変換して使用

#### Nodes 1.0 (Canvas) の場合の配色

チェックボックスがオン状態:
- チェックボックスの枠線: --input-text
- チェックボックスの塗り: --input-text
- チェックボックスのチェック: --comfy-input-bg
- 表示用テキスト: --input-text

チェックボックスがオフ状態:
- チェックボックスの枠線: --input-text (透明度: 0.5)
- チェックボックスの塗り: なし
- 表示用テキスト: --input-text (透明度: 0.4)

その他:
- ウェイトボタンの塗り: --comfy-input-bg
- ウェイトボタンの+と-: --input-text (透明度: 0.6)
- "No Text" の文字: --input-text (透明度: 0.6)

#### Nodes 2.0 (DOM) の場合の配色

チェックボックスがオン状態:
- チェックボックスの枠線: --text-primary
- チェックボックスの塗り: --text-primary
- チェックボックスのチェック: --component-node-widget-background
- 表示用テキスト: --text-primary

チェックボックスがオフ状態:
- チェックボックスの枠線: --text-primary (透明度: 0.5)
- チェックボックスの塗り: なし
- 表示用テキスト: --text-primary (透明度: 0.4)

その他:
- ウェイトボタンの塗り: --component-node-widget-background
- ウェイトボタンの塗り (ホバー時): --component-node-widget-background-hovered
- ウェイトボタンの+と-: --text-primary (透明度: 0.6)
- トグルボタンの文字: --text-primary (透明度: 0.6)
- トグルボタンの塗り: --component-node-widget-background
- トグルボタンの塗り (ホバー時): --component-node-widget-background-hovered
- "No Text" の文字: --text-primary (透明度: 0.6)

### 拡張登録

- ComfyUI 拡張として登録し、PromptPalette ノードの生成/描画にフック
- `beforeRegisterNodeDef` で PromptPalette ノードの挙動を差し替え

## その他の実装方針

- UI の定数は `CONFIG` オブジェクトで定義

## ComfyUIの実行方法

ComfyUI portableのcustom_nodesフォルダ内にComfyUI-PromptPaletteフォルダを配置している。
ワーキングディレクトリがComfyUI-PromptPaletteフォルダとすると、以下のコマンドでComfyUIを起動する。

```
..\..\..\python_embeded\python.exe -u -s ..\..\..\ComfyUI\main.py --windows-standalone-build --disable-auto-launch
```

## テスト方法

1. ComfyUIを再起動
2. MCP PlaywrightでComfyUIを開く
3. 画面上部の+ボタンで新しいWorkflowを作成
4. 画面中央付近をダブルクリックしてノードを検索
5. "PromptPalette" と入力してEnter
6. 画面中央にPromptPaletteノードが追加されたことを確認
7. PromptPaletteノードのEditボタンを押す
8. PromptPaletteノードのテキストエリアに以下の行を追加
```
aaa
(bbb:0.9) // BBB
// (abc:1.2)
//
```
9. PromptPaletteノードのSaveボタンを押す
10. hogeの右横の+ボタンを2回押す
11. hogeの右横の数値が1.2になったことを確認

## コーディングエージェントのポリシー

- 要求にない変更はせず、指定された箇所のみ最小限の変更に留める
- 要求に不明な点や曖昧な点があれば確認し、勝手な推測をしない
- コードを編集する際は、その意図を説明する
- コマンドを実行する際は、その意図を説明する
- チャットでの回答は日本語

## 参考ページ

- [Comfy Objects - ComfyUI](https://docs.comfy.org/custom-nodes/js/javascript_objects_and_hijacking)
- [Comfy Hooks - ComfyUI](https://docs.comfy.org/custom-nodes/js/javascript_hooks)

---
> Source: [kambara/ComfyUI-PromptPalette](https://github.com/kambara/ComfyUI-PromptPalette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
