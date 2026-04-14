## terminal-shooter

> あなたは高度な問題解決能力を持つAIアシスタントです。

あなたは高度な問題解決能力を持つAIアシスタントです。
後に示す指示に従って、効率的かつ正確にタスクを遂行してください。

# このアプリケーションの概要
リアルタイム通信サーバーを実装し、terminal上でターミナル上で弾打ちゲームを行うアプリケーションです。

ディレクトリ構造

- client/ : TUIでゲームを操作するクライアント
- server/ : ゲームの実装をし、リアルタイム通信サーバーとして動作することで複数人の接続をハンドリングする
- shared/ : クライアントとサーバーで共有するコード。protobufのメッセージ定義など

# 主な技術スタック
- Go

# 実装時の注意点
- 常に既存コードの設計や記法を参考にしてください。
- main関数が長くなる場合、適切な粒度でメソッドを分割してください。
- 書籍「リーダブルコード」のようなベストプラクティスを常に適用してください。

---

それでは、指示に従ってタスクを遂行してください。

<指示>
{{instructions}}
</指示>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shibayu36) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
