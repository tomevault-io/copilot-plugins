## ndlocrlite-web

> ONNXモデルを使用するOCRツールNDLOCR-Lite（https://github.com/ndl-lab/ndlocr-lite）を ONNX WebRuntimeを使用してWebアプリケーションにポートしたもの

# ツール名

NDLOCR-Lite Web

## 概要

ONNXモデルを使用するOCRツールNDLOCR-Lite（https://github.com/ndl-lab/ndlocr-lite）を ONNX WebRuntimeを使用してWebアプリケーションにポートしたもの

## 機能

- jpg/png/PDFなどの複数画像一括処理モード
- ディレクトリ内の画像の一括処理モード
- 作業中の画像のレイアウト認識結果確認モード
  - マウスで任意のリージョンを選択すると、その範囲のOCR結果を表示する機能
- 作業の進捗状況のゲージ表示
- 作業結果のテキストダウンロード、コピー機能
- ダウンロードしたONNXモデルはIndexedDBにキャッシュして次回以降高速起動
- 過去の作業結果100件もIndexedDBにキャッシュして参照可能にする
  - キャッシュクリア機能も
- ユーザーへの注意書き
  - 本アプリケーションの動作はWebブラウザで完結しており、選択した画像ファイルとそのOCR結果は外部に送信されないことを明記する
  - 作成者：橋本雄太（国立歴史民俗博物館、国立国会図書館非常勤調査員）
  - GitHubレポジトリへのリンク

## 技術スタック

- ONNX Web Runtie
- React
- Google Analytics

## デプロイ先

GitHub + Netlify
GitHubにパブリックレポとして公開し、これをNetlifyに自動デプロイする

---
> Source: [yuta1984/ndlocrlite-web](https://github.com/yuta1984/ndlocrlite-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
