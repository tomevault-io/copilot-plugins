## web-pong

> HTML/CSS/JavaScriptのみで構成されたシンプルなPongゲームです。

# プロジェクト概要

## プロジェクト名: Web Pong Game

## 目的:
HTML/CSS/JavaScriptのみで構成されたシンプルなPongゲームです。
ボールとパドルの衝突判定、ボールの移動、スコア管理、ゲームの状態管理（開始、終了など）が主な機能です。

## 技術スタック:
* **言語:** JavaScript (ES6+), HTML5, CSS3
* **フレームワーク/ライブラリ:** なし (Vanilla JavaScript)
* **テストフレームワーク (希望):** Jest (もしGeminiが他の選択肢を提案する場合は検討可)
* **テストランナー (希望):** Node.js環境で実行可能なもの (例: Jest, Puppeteerなど)

## プロジェクトのファイル構造:

* `index.html`: ゲームのHTML構造。キャンバス要素が含まれます。
* `style.css`: ゲームの見た目を定義するCSS。
* `main.js`: ゲームのロジック（ボールの移動、衝突判定、ゲーム状態など）を記述したJavaScriptファイル。

## テストに関する指示:

### 1. テストの最終目的:
* ゲームの主要なロジックが正しく機能することを確認する。
* CI/CDパイプラインで自動的に実行できること。

### 2. テストの対象:
主に `main.js` 内のゲームロジック関数を対象とします。
テストが存在しないため、まずは簡単な単体テストの実行を目標とします。
カバレッジについては環境が整備できてから検討します。

### 3. テストの粒度:
まずは最も簡単なテストを作成してテストコマンドが成功することを確認してください。

### 4. テスト環境:
* Node.js環境で実行可能なテストコードを想定しています。

### 5. コード生成のスタイル:
* テストコードは、読みやすく、理解しやすいものであること。

### 6. CI/CDへの組み込み:
* テストコードが動作するようになってからの検討とします。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/you-sk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/you-sk)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
