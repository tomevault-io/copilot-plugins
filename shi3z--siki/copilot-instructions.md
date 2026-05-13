## siki

> コードを変更してビルドしたら、**必ず既存プロセスを kill して再起動すること**。

# siki 開発ルール

## ビルド & 再起動（必須）

コードを変更してビルドしたら、**必ず既存プロセスを kill して再起動すること**。
`go:embed` で HTML を埋め込んでいるため、ビルドしないと反映されない。
ビルドしても再起動しないと古いバイナリが動き続ける。

```bash
# ビルド
/usr/local/go/bin/go build -o siki .

# 既存プロセスを停止して再起動
pkill -f './siki web' && sleep 1 && ./siki web &
```

## プロジェクト構成

- 単一バイナリ Go アプリ（外部依存なし、標準ライブラリのみ）
- `main.go` — 全コード
- `web/index.html` — Web UI（go:embed で埋め込み）
- `~/.siki/` — データディレクトリ（設定、スレッド、キャッシュ等）

## Go ビルド

- Go パス: `/usr/local/go/bin/go`
- モジュール: `go 1.25.0`、外部依存なし

---
> Source: [shi3z/siki](https://github.com/shi3z/siki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
