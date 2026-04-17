## raspi-hypervisor

> - このプロジェクトは Raspberry Pi 3 用の実験的なハイパーバイザ実装です

# Project overview
- このプロジェクトは Raspberry Pi 3 用の実験的なハイパーバイザ実装です

# Project structure
- `lib/src/`: 共通のソース
- `lib/include/`: 共通のヘッダ
- `sm/src/`: セキュアモニタのソース
- `sm/include/`: セキュアモニタのヘッダ
- `hv/build/`: セキュアモニタのビルド成果物
- `hv/src/`: ハイパーバイザのソース
- `hv/include/`: ハイパーバイザのヘッダ
- `hv/build/`: ハイパーバイザのビルド成果物
- `os/`: ゲスト OS のソースやヘッダ
    - `vmm`: VM を起動するための管理用 OS
    - `raspios`: 動作確認を行うための最小構成の OS で、複数コアに対応している

# その他の指示
- 最終的な出力はすべて日本語で行ってください。
- ビルドや実行は docker を使って行ってください。以下はサンプルです。
    - ビルド: `./run_command.sh make`
    - セキュアモニタの実行: `./run_command.sh ./run_sm.sh`
    - ハイパーバイザの実行: `./run_command.sh ./run_hv.sh`
- OS　やハイパーバイザやセキュアモニタを実行した場合、自動では終了しませんので、`timeout` コマンドをつけるか `Ctrl-a x` で終了させてください
    - ただし、`timeout` コマンドは mac 環境にはなく、docker 内にしかないので注意してください
    - つまり、`./run_command.sh timeout (command)` のように実行してください

# 実行およびプロセス管理に関する重要事項
- **プロセスの放置禁止**: コマンド実行（ビルド、QEMU、Docker等）の後は、必ずプロセスが終了したことを確認してください。
- **クリーンアップの徹底**: 動作確認やテストが終わったら、使用したバックグラウンドプロセス（QEMU、Dockerコンテナ等）を即座に停止・終了させてください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shimomura1004) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
