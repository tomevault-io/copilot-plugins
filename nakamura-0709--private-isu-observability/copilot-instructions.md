## private-isu-observability

> このプロジェクトは、書籍『実践オブザーバビリティ & パフォーマンスチューニング』に則り、`private-isu`を題材にしてオブザーバビリティ基盤の構築とパフォーマンスチューニングを行うためのハンズオンリポジトリです。

# 実践オブザーバビリティ & パフォーマンスチューニング ハンズオン環境 (isu-perf-lab)

このプロジェクトは、書籍『実践オブザーバビリティ & パフォーマンスチューニング』に則り、`private-isu`を題材にしてオブザーバビリティ基盤の構築とパフォーマンスチューニングを行うためのハンズオンリポジトリです。

## Core Principle

「推測するな、計測せよ (Don't guess, measure)」

- 改善提案は必ず計測データ（alp, pt-query-digest, Grafana, Pyroscope, OTelトレース）に基づくこと。
- 根拠のない最適化提案は行わず、先に計測データの提示を求めること。
- 1つの変更ごとにベンチマークを実行し、スコアとメトリクスの変化で効果を検証すること。

## Phase Strategy

1. **Setup Phase**: AWS/Docker/OTelの基盤構築
2. **Analysis Phase**: alp・pt-query-digest・Grafanaによるボトルネック特定
3. **Tuning Phase**: 実装改善 → ベンチマーク → 効果検証のサイクルを繰り返す

## 技術スタック

- **アプリケーション**: Go
- **Webサーバー**: Nginx
- **データベース**: MySQL
- **オブザーバビリティ基盤**: OpenTelemetry (otelcol-contrib), ClickHouse, Grafana, Pyroscope
- **ログ分析ツール**: alp, pt-query-digest
- **インフラ**: AWS (EC2, CloudFormation)
- **タスクランナー**: Makefile

## 必須ツール

以下のツールがインストールされている必要があります。

- Go
- jq
- direnv
- Docker Desktop
- AWS CLI

ツールの確認コマンド:

```bash
make tool.check-required
```

## 初期セットアップ

1. **環境変数のロード**
   ディレクトリに移動し、direnvを許可します。

   ```bash
   direnv allow
   ```

2. **AWS認証情報の設定**
   AWS CLIの認証情報が未設定の場合は設定します。

   ```bash
   aws configure
   ```

   特定のプロファイルを使用する場合は、`.envrc.override`ファイルを作成し、
   `export AWS_PROFILE=〇〇`を設定してください。

3. **SSH鍵の設定**
   GitHubに登録されている公開鍵と対応する秘密鍵が必要です。
   必要に応じて`.envrc`または`.envrc.override`の`SSH_PRIVATE_KEY_PATH`や
   `GITHUB_USERNAME`を設定してください。

## よく使うコマンド一覧 (Makefile)

### AWS環境操作

- `make aws.create-cfn` : CFnスタックを作成し、VPCやEC2等のAWS環境を構築します。
- `make aws.setup-ssh-config` : SSH設定ファイルを生成し、接続可能な状態にします。
- `make aws.up` : 中断していたEC2インスタンスを再起動し、IPアドレスの許可設定を更新します。
- `make aws.down` : 作業を中断する際にEC2インスタンスを停止し、課金を抑えます。
- `make aws.delete-cfn` : CFnスタックを削除し、すべてのAWSリソースを破棄します。

### 開発・チューニングサイクル

- `make deploy` : ローカルで変更したGoアプリ、Nginx設定、MySQL設定・スキーマをWebインスタンスにデプロイします。
- `make bench` : ベンチマークを実行し、終了後に結果とログをローカルにダウンロードします。
- `make analyze` : ダウンロードしたアクセスログ(alp)とスロークエリログ(pt-query-digest)を分析し、結果をClickHouseに保存します。

### 分析・可視化画面

- `make perf.open` : ClickHouseのWeb UIとGrafana、Pyroscopeのダッシュボードをブラウザで開きます。

## 開発の進め方

1. `make aws.up` で環境を起動する。
2. ローカルでアプリケーションコード(`private_isu/webapp/golang/`)や設定ファイルを修正する。
3. `make deploy` で変更をサーバーに反映させる。
4. `make bench` でベンチマークを実行し、スコアを計測する。
5. `make analyze` で各種ログを分析し、データベースへ保存する。
6. `make perf.open` でGrafanaなどのダッシュボードを確認し、次のボトルネックを特定して仮説を立てる。
7. 作業を終了する時は `make aws.down` でインスタンスを停止する。

## セッション開始時の動作

会話を開始したら必ず `notes/` ディレクトリを確認し、現状を把握すること。
タスクが完了したら、対応する `notes/` 内の記述を削除すること。
すべてのタスクが完了したらファイル自体を削除すること。

## 注意事項

- **AWSの課金について**: AWSリソースを稼働させている間は課金が発生します。
  ハンズオン中断時は `make aws.down` で停止し、完全に終了した場合は
  `make aws.delete-cfn` でリソースを忘れずに削除してください。
- **EC2停止中の課金**: `make aws.down` でEC2を停止している間も、EBSボリューム分の課金は継続して発生します。

---
> Source: [Nakamura-0709/private-isu-observability](https://github.com/Nakamura-0709/private-isu-observability) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
