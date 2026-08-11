## pico-boot

> Raspberry Pi Pico W (RP2040 + CYW43439) 用 WiFi ブートローダ。TDD + 形式検証で開発する。

# pico-boot 開発メモ

Raspberry Pi Pico W (RP2040 + CYW43439) 用 WiFi ブートローダ。TDD + 形式検証で開発する。
アーキテクチャと設計判断は `DESIGN.md`、使い方は README、詳細は各 `src` の doc コメント。
ここは作業手順と落とし穴のみ。

## ワークスペース構成

- **ルート = ブートローダ crate**（`thumbv6m`）。root で `cargo build` / `cargo run` = ファーム。
  build.rs は `vendor/cyw43-firmware/`(cyw43 blob）・root 直下の `keys/`・`.env` を読む。
- **`host/`** = ホストワークスペース（dir `core` = crate `pico-boot-core`、dir `pbtool` = crate
  `pico-boot-tool`（binary は `pbtool`））。root の thumbv6m 継承を打ち消すため
  `host/.cargo/config.toml` で target をホストに戻す（トリプルはハードコード、別 OS では要変更）。
- `examples/`（`heartbeat`・`self-update`・`bare-metal`）・`guest/`（crate `pico-boot-guest`）は
  別ワークスペース（thumbv6m）。`host/core` を path 依存。

## コマンド

```sh
cargo build --release                        # ブートローダビルド（root = firmware）
cargo run --release                          # 書き込み + defmt ログ
cargo test --test hil --release              # 実機テスト（アプリスロット破壊注意）
cd host && cargo test                        # ホストテスト（core + pbtool E2E）
cd host && cargo run -p pico-boot-tool -- ...        # pbtool（or cargo install --path host/pbtool）
cd examples/heartbeat && cargo build --release      # サンプルアプリ
cd host/core && cargo kani                 # 形式検証（要 kani-verifier）
```

## 落とし穴

- ロジックは `core`(`pico-boot-core`)に置き、テストを先に書く。bootloader は薄いグルーに保つ。
- フラッシュレイアウトは `layout.rs` が唯一の情報源。変更時は `memory.x`・各 example の `build.rs` のスロット定数を同期。
- cyw43 は `vendor/cyw43` に patch 済み（`PICO_BOOT_PATCH.md`）。更新時は再取得＋再適用。
- embassy / cyw43 の API はリリース版と main で乖離する。シグネチャは `~/.cargo/registry/src/` の
  該当バージョンで確認（GitHub の examples は main 向け）。
- AB アプリはスロット別リンクが必要（`PICO_BOOT_SLOT=a|b`）。Cortex-M0+ は非 PIC。
- production ビルドは `--features require-auth`（鍵欠如＝ビルドエラー）。
- アプリの反復開発は `cd examples/heartbeat && cargo run`（= `probe-rs run`）で完結する。ヘッダを書かなくても
  ブートローダがベクタ健全性で起動する（header-optional gate）。ブートローダは一度書いておけばよい。
  `pbtool pack` + `probe-rs download` はヘッダ付き一括投入用に残るが通常は不要。
  ゲート（header-optional + vectors-last + latch）の設計・不変条件・証明義務は `DESIGN.md` §5。
- ハング検出 watchdog（`DESIGN.md` §5.5）: **常時有効**（opt-in ではない）。**header 付き trial なら
  常に** arm、headerless（`probe-rs run`）のみ除外（H7）。timeout は `hang_watchdog_ms`（`0`=既定
  `DEFAULT_HANG_WATCHDOG_MS`、`pbtool hang <ms>` / guest `set_hang_watchdog` で調整）。`main` 入口で
  **無条件 disarm**（I1、消すと upload mode で brick）。ゲスト `confirm()` が feed→確定→disable で
  止める。ロジックは `boot.rs`（`plan_watchdog`/`effective_hang_watchdog_ms`/`hang_watchdog_load`）に
  置き Kani で証明（W1/W2/W5/W6）。
  実機 HIL は pause_dbg でカウント観測不可なのでレジスタ配線のみ検証、発火→revert 一巡は手動 HIL。
- WiFi の初期認証情報はビルド時に `.env`（リポジトリトップ、gitignore・Read 拒否済み）から焼き込む。
- プローブが無応答になったら USBDEVFS_RESET で復旧（device node は world-writable）。`probe-rs run`
  は XIAO プローブを wedge させやすい。`probe-rs info`/HIL 前に一度リセットすると通りやすい。

## ハードウェア

- デバッグプローブ: CMSIS-DAP、VID:PID 6666:4444（runner に指定済み）。
- ターゲット: Pico W。probe-rs のチップ名は `RP2040`。

---
> Source: [sksat/pico-boot](https://github.com/sksat/pico-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
