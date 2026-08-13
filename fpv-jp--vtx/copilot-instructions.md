## vtx

> GStreamer + libnice を使い、WebRTC sendonly で映像・音声・テレメトリを vrx (受信側) へ送信する C プログラム。

# vtx — Codebase Guide

GStreamer + libnice を使い、WebRTC sendonly で映像・音声・テレメトリを vrx (受信側) へ送信する C プログラム。

## アーキテクチャ概要

```
main.c
 └─ signaling.c          WebSocket シグナリング (libsoup)
      ├─ webrtc.c         webrtcbin 制御・SDP offer 生成・ICE ネゴシエーション
      │   ├─ pipeline_factory.c  GStreamer パイプライン文字列の組み立て・起動
      │   └─ ice.c        カスタム ICE エージェント (network_interface 指定時)
      ├─ datachannel.c           DataChannel (テレメトリ送信)
      │   ├─ datachannel_command.c
      │   ├─ datachannel_flight_controller.c  MSP 経由でフライトコントローラから受信
      │   └─ datachannel_wpa_supplicant.c     Wi-Fi 状態通知
      ├─ msp.c / msp_common.c    MSP (MultiWii Serial Protocol) 実装
      ├─ nic.c / nic_parser.c    Wi-Fi NIC 検出・情報収集
      ├─ device.c                カメラ・マイクデバイス列挙
      ├─ codec.c                 エンコーダ対応状況の検査
      ├─ rtp.c                   RTP ユーティリティ
      └─ utils.c                 共通ユーティリティ・クリーンアップ
```

## セッションフロー

```
vtx                          signaling server                    vrx
 |                                |                               |
 |--- WebSocket connect (sender) ->|                               |
 |<-- SENDER_SESSION_ID_ISSUANCE (100) --|                        |
 |--- SENDER_PLATFORM_INFO (110) ------->|                        |
 |                                |<-- SENDER_MEDIA_DEVICE_LIST_REQUEST (101) --|
 |<-- SENDER_MEDIA_DEVICE_LIST_REQUEST --|                        |
 |--- RECEIVER_MEDIA_DEVICE_LIST_RESPONSE (202) ->|-- response -->|
 |                                |<-- SENDER_MEDIA_STREAM_START (102) --------|
 |  [パイプライン起動・SDP offer 生成]                              |
 |--- RECEIVER_SDP_OFFER (203) ---------->|---------- offer ----->|
 |<-- SENDER_SDP_ANSWER (103) -----------|<--------- answer -----|
 |<-- SENDER_ICE (104) ------------------|<----------- ICE ------|
 |--- RECEIVER_ICE (204) ---------------->|------------ ICE ----->|
 |                [映像・音声・DataChannel ストリーミング開始]
```

## シグナリングメッセージ番号 (common.h)

| 定数 | 値 | 方向 | 意味 |
|---|---|---|---|
| SENDER_SESSION_ID_ISSUANCE | 100 | server→vtx | セッション ID 払い出し |
| SENDER_MEDIA_DEVICE_LIST_REQUEST | 101 | vrx→vtx | デバイス一覧要求 |
| SENDER_MEDIA_STREAM_START | 102 | vrx→vtx | ストリーム開始要求 |
| SENDER_SDP_ANSWER | 103 | vrx→vtx | SDP Answer |
| SENDER_ICE | 104 | vrx→vtx | ICE candidate |
| SENDER_RECEIVER_CLOSE | 108 | vrx→vtx | 切断通知 |
| SENDER_PLATFORM_INFO | 110 | vtx→server | プラットフォーム情報送信 |
| RECEIVER_MEDIA_DEVICE_LIST_RESPONSE | 202 | vtx→vrx | デバイス一覧応答 |
| RECEIVER_SDP_OFFER | 203 | vtx→vrx | SDP Offer |
| RECEIVER_ICE | 204 | vtx→vrx | ICE candidate |
| RECEIVER_SYSTEM_ERROR | 209 | vtx→vrx | エラー通知 |

## プラットフォーム検出 (common.h / device.c)

`vtx_detect_platform()` が `/proc/device-tree/model` を読んで `g_platform` (PlatformType) をセット。
`vtx_detect_gpu_vendor()` は LINUX_X86 のみ GPU ベンダーを検出して `g_gpu_vendor` をセット。
`pipeline_factory.c` の `vtx_pipeline_build()` でプラットフォーム別エンコーダパイプラインを選択する。

対応プラットフォーム: `APPLE_MAC`, `LINUX_X86`, `RASPBERRY_PI_4B`, `RASPBERRY_PI_4CM`, `RASPBERRY_PI_5`, `RASPBERRY_PI_5CM`, `JETSON_NANO_2GB`, `JETSON_ORIN_NANO_SUPER`, `RADXA_ROCK_5B`, `RADXA_ROCK_5T`

## 主要グローバル変数 (common.h で extern 宣言)

| 変数 | 型 | 役割 |
|---|---|---|
| `loop` | `GMainLoop *` | GLib イベントループ |
| `app_state` | `AppState` | 接続状態ステートマシン |
| `ws_conn` | `SoupWebsocketConnection *` | シグナリングサーバへの WebSocket 接続 |
| `ws1Id` | `gchar *` | vtx のセッション ID |
| `ws2Id` | `gchar *` | vrx のセッション ID |
| `pipeline` | `GstElement *` | GStreamer パイプライン |
| `webrtc` | `GstElement *` | webrtcbin 要素 |
| `g_platform` | `PlatformType` | 実行時プラットフォーム |
| `g_gpu_vendor` | `GpuVendor` | GPU ベンダー (LINUX_X86 のみ) |

## ビルド

```bash
make          # 本番ビルド (./vtx)
make test     # Unity テスト (./vtx_test)
make bear     # compile_commands.json 生成 (clangd 用)
make clean    # 成果物削除
```

依存: `gstreamer-1.0`, `gstreamer-webrtc-1.0`, `gstreamer-rtp-1.0`, `gstreamer-sdp-1.0`, `libsoup-3.0` (または `2.4`、自動検出), `json-glib-1.0`, `libnice`

## 実行時環境変数 (service/vtx.env)

| 変数 | デフォルト | 説明 |
|---|---|---|
| `SIGNALING_ENDPOINT` | `wss://fpv/signaling` | シグナリングサーバ URL |
| `SERVER_CERTIFICATE_AUTHORITY` | `./server-ca-cert.pem` | TLS CA 証明書パス (空文字でシステム証明書を使用) |

公開サーバでテストする場合: `SIGNALING_ENDPOINT=wss://fpv.jp/signaling SERVER_CERTIFICATE_AUTHORITY= ./vtx`

## パイプライン構造

vrx から受け取った `video_pipeline` / `audio_pipeline` 文字列 (GStreamer パイプライン記述) を `vtx_pipeline_build()` で組み立てて `webrtcbin` に接続する。
`network_interface` が指定された場合はカスタム ICE エージェント (`ice.c`) を使い、特定の NIC を ICE に使用させる。

## DataChannel 一覧 (data_channel.h / datachannel.c)

WebRTC DataChannel でテレメトリをバイナリ送信する。チャンネルは常に作成され、FC 未接続時はゼロ埋めのダミーデータを返す。

| チャンネルラベル | MSP コマンド | 送信頻度 | データ内容 | バイト数 |
|---|---|---|---|---|
| `CMD` | — | on-demand | コマンド (HANG_UP/PING/PONG/ERROR) | 可変 |
| `MSP_RAW_IMU` | `MSP_RAW_IMU` (102) | **50 Hz** | acc[3] + gyro[3] + mag[3] (各 int16) | 18 |
| `MSP_RAW_GPS` | `MSP_RAW_GPS` (106) | **1 Hz** | fix(u8) + numsat(u8) + lat(i32) + lon(i32) + alt(u16) + speed(u16) + course(u16) | 16 |
| `MSP_COMP_GPS` | `MSP_COMP_GPS` (107) | **1 Hz** | distanceToHome(u16) + directionToHome(u16) + heartbeat(u8) | 5 |
| `MSP_ATTITUDE` | `MSP_ATTITUDE` (108) | **30 Hz** | roll(i16) + pitch(i16) + heading(i16) | 6 |
| `MSP_ALTITUDE` | `MSP_ALTITUDE` (109) | **10 Hz** | estimatedAltitude(i32) | 4 |
| `MSP_ANALOG` | `MSP_ANALOG` (110) | **2 Hz** | vbat(u8) + mAhDrawn(u16) + rssi(u16) + amperage(i16) + voltage(u16) | 9 |
| `MSP_SONAR` | `MSP_SONAR_ALTITUDE` (58) | **10 Hz** | altitude(i32) [cm] | 4 |
| `MSP_BATTERY_STATE` | `MSP_BATTERY_STATE` (130) | **2 Hz** | cellCount(u8) + capacity(u16) + voltage(u8) + mAhDrawn(u16) + amperage(u16) + batteryState(u8) + milliVoltage(u16) | 10 |
| `WPA_SUPPLICANT` | — | **1 Hz** | Wi-Fi 接続状態 (JSON) | 可変 |

**GPS 関連チャンネルの詳細:**
- `MSP_RAW_GPS`: 緯度・経度 (度×10^7)・高度 (m)・速度 (cm/s)・進行方向 (度×10) ← GPS 座標取得はここ
- `MSP_COMP_GPS`: ホームポイントまでの距離 (m) と方位 (度) ← コンパス情報はここ

**DataChannel の信頼性設定:**
- 高頻度 (IMU/ATTITUDE): `ordered=false`, `max-packet-lifetime` — 低遅延優先、パケットロス許容
- 低頻度 (GPS/BATTERY/ANALOG): `ordered=true`, `max-retransmits=5` — 信頼性優先

## テスト

Unity フレームワークを使用。テストファイルは `test/` 以下。
`make test` でビルド・実行まで行う。

---
> Source: [fpv-jp/vtx](https://github.com/fpv-jp/vtx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
