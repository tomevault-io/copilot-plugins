## peerclock

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
swift build                                    # ビルド
swift test                                     # 全テスト実行（37テスト、7スイート）
swift test --filter TypesTests                 # スイート単位で実行
swift test --filter NTPSyncEngineTests/offsetCalculation  # 単一テスト実行
```

- Swift Package Manager のみ（Xcode プロジェクトなし）
- Swift tools version 6.0（strict concurrency 有効）
- テストフレームワーク: Swift Testing（`import Testing`, `@Suite`, `@Test`, `#expect`）
- リンター・CI は未設定

## プロジェクト概要

PeerClock は Apple デバイス間の**対等ピアツーピア連携基盤**。クロック同期（±2ms）・汎用コマンドチャネル・ステータス共有を提供する。[1Take](https://github.com/hakaru/1Take)（マルチデバイス同時録音アプリ）から抽出。

対象: iOS 17.0+ / macOS 14+。外部依存なし。

**差別化**: iOS/macOS エコシステムに「対等ピア・クロック同期 + コマンド/ステータス統合基盤」は存在しない（2026-04 調査済み）。

## アーキテクチャ（対等ピア、Role なし）

**全ノード対等。** 公開 API に Role の概念はない。クロック同期に必要な coordinator は最小 PeerID で自動選出され、アプリからは不可視。

設計仕様: `docs/superpowers/specs/2026-04-07-peerclock-v2-design.md`

```
PeerClock (Facade — role-free)
├── Protocols/        Transport, SyncEngine, CommandHandler（境界に Protocol、即座に実装）
├── Transport/        Discovery (Bonjour) + WiFiTransport (UDP/TCP) + MockTransport
├── Coordination/     CoordinatorElection（最小 PeerID 自動選出）
├── ClockSync/        NTPSyncEngine (4-timestamp + best-half) + DriftMonitor
├── Command/          CommandRouter（汎用コマンド send/broadcast）
└── Wire/             MessageCodec（ワイヤプロトコル エンコード/デコード）
```

### 主要 Protocol

- `Transport`: `sendUnreliable`/`sendReliable`/`broadcastReliable` + `connectionEvents`。WiFiTransport（実機）と MockTransport（テスト）が準拠
- `SyncEngine`: `currentOffset`/`start(coordinator:)`/`stop()`。NTPSyncEngine が準拠
- `CommandHandler`: `send`/`broadcast`/`incomingCommands`。CommandRouter が準拠

### テスト戦略

MockTransport（インメモリ）で全ロジックをユニットテスト。WiFiTransport は実機テストのみ。テスト用には `PeerClock(transportFactory:)` で MockTransport を注入:

```swift
let network = MockNetwork()
let clock = PeerClock(configuration: config, transportFactory: { peerID in
    network.createTransport(for: peerID)
})
```

## ワイヤプロトコル

ヘッダ 5 bytes: `Version(1) + Category(1) + Flags(1) + Length(2, big-endian)`

- Unreliable チャネル: `SYNC_REQUEST`(0x01) / `SYNC_RESPONSE`(0x02) — 方向非依存
- Reliable チャネル: `HEARTBEAT`(0x10) / `DISCONNECT`(0x11) / `ELECTION`(0x12) / `APP_COMMAND`(0x20) / `STATUS_*`(0x30-32)
- 整数: 全て big-endian。文字列: UTF-8。

## 同期アルゴリズム

- NTP 風 4-timestamp 交換: `offset = ((t1-t0) + (t2-t3)) / 2`
- 30ms 間隔で 40 回測定（`Configuration` で調整可能）
- best-half filtering（RTT 上位 50%）
- 5 秒周期再同期
- `DriftMonitor`: オフセット差 >10ms でジャンプ検出 → 完全再同期

## 実装ロードマップ

- **Phase 1 (完了)**: Transport + ClockSync + CommandChannel + Coordinator 選出 + Facade
- **Phase 2**: ステータス共有（プッシュ+プル、世代番号、debounce）+ EventScheduler
- **Phase 3**: MultipeerConnectivity フォールバック、自動トランスポート切替、再接続
- **Phase 4**: 合議ベース同期、対等ピア選出の洗練、超音波同期、watchOS

## 注意事項

- Swift 6 strict concurrency 有効。公開型は `Sendable` 準拠必須。可変状態を持つクラスは `@unchecked Sendable` + `NSLock`
- アプリターゲット追加時に `NSLocalNetworkUsageDescription` と `NSBonjourServices: ["_peerclock._udp"]` が Info.plist に必要
- `docs/archive/DESIGN-v1.md` は初期設計（master/slave）で参考情報。正の設計仕様は `docs/superpowers/specs/` の v2 spec

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hakaru) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
