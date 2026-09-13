## broot0

> Raspberry Pi Zero Wに接続されたカメラの映像をHDMI出力するだけのシンプルなイメージをBuildRootで作成するプロジェクト。

# Raspberry Pi Zero W カメラ→HDMI出力システム開発記録

## プロジェクト概要
Raspberry Pi Zero Wに接続されたカメラの映像をHDMI出力するだけのシンプルなイメージをBuildRootで作成するプロジェクト。

## 開発環境
- macOS (Darwin 24.6.0)
- BuildRoot 2025.08-rc3
- 開発期間: 2025-09-04

## 実装内容

### 1. カスタム設定ファイル作成
**ファイル**: `raspberrypi0w_camera_defconfig`
- ベース: Raspberry Pi Zero W標準設定
- 追加パッケージ:
  - `BR2_PACKAGE_RPI_USERLAND=y` (raspividコマンド用)
  - `BR2_PACKAGE_V4L_UTILS=y` (Video4Linux2ユーティリティ)
  - `BR2_PACKAGE_FFMPEG=y` (動画処理)
- ファイルシステム: ext4 256MB

### 2. ボード設定ディレクトリ作成
**ディレクトリ**: `board/raspberrypi0w-camera/`

#### a) Raspberry Pi設定 (`config_0w.txt`)
```
gpu_mem=128                 # GPU メモリ 128MB
start_x=1                   # カメラ有効化
disable_camera_led=1        # カメラLED無効化
hdmi_force_hotplug=1        # HDMI強制検出
hdmi_drive=2               # HDMI標準モード
hdmi_group=1               # CEA group
hdmi_mode=16               # 1920x1080 60Hz
```

#### b) 起動スクリプト (`start_raspivid.sh`)
```bash
#!/bin/sh
tvservice -e "CEA 16 HDMI"  # HDMI出力設定
sleep 2
raspivid -f -t 0 -p 0,0,1920,1080 -fps 30  # フルスクリーン連続出力
```

#### c) ビルド後処理 (`post-build.sh`)
- GPU設定をconfig.txtに追加
- 起動スクリプトをシステムに配置
- inittabに自動起動エントリ追加

### 3. システム仕様
- **アーキテクチャ**: ARMv6 (arm1176jzf-s)
- **ツールチェーン**: Bootlin External Toolchain
- **カーネル**: Raspberry Pi カスタム
- **起動時間**: 約30秒～1分
- **出力解像度**: 1920x1080 30fps
- **自動起動**: システム起動時にraspivid自動実行

## 技術課題と解決

### 課題1: macOSでのBuildRootビルド
**問題**: 
- gccコンパイラの認識問題
- PATH環境変数の特殊文字問題

**解決策**: 
- Linux環境での実行を推奨
- Docker/VM環境の利用

### 課題2: カメラとHDMI同時出力
**解決策**:
- GPU メモリを128MBに設定
- raspividのプレビューモードでフルスクリーン出力
- HDMI設定を明示的に指定

## ファイル構成
```
rpi_camera_configs/
├── BUILD_GUIDE.md                           # 完全なビルド手順書
├── raspberrypi0w_camera_defconfig          # BuildRoot設定
└── raspberrypi0w-camera/                   # ボード設定
    ├── config_0w.txt                       # Pi設定
    ├── genimage-raspberrypi0w-camera.cfg   # イメージ生成設定
    ├── post-build.sh                       # ビルド後処理
    ├── post-image.sh                       # イメージ後処理
    └── start_raspivid.sh                   # 起動スクリプト
```

## Linux環境でのビルド手順
```bash
# 1. BuildRootクローン
git clone https://github.com/buildroot/buildroot.git
cd buildroot

# 2. 設定ファイル配置
cp ../rpi_camera_configs/raspberrypi0w_camera_defconfig configs/
cp -r ../rpi_camera_configs/raspberrypi0w-camera board/

# 3. 設定ロードとビルド
make raspberrypi0w_camera_defconfig
make -j$(nproc)

# 4. 生成物
# output/images/sdcard.img が作成される
```

## カスタマイズ例

### 解像度変更
`start_raspivid.sh`で解像度を変更:
```bash
# 1280x720
raspivid -f -t 0 -p 0,0,1280,720 -fps 30

# 640x480  
raspivid -f -t 0 -p 0,0,640,480 -fps 30
```

### フレームレート変更
```bash
# 60fps
raspivid -f -t 0 -p 0,0,1920,1080 -fps 60

# 15fps
raspivid -f -t 0 -p 0,0,1920,1080 -fps 15
```

## 動作確認手順
1. SDカードに`sdcard.img`を書き込み
2. Raspberry Pi Zero Wにカメラモジュール接続
3. HDMI接続
4. 電源投入
5. 約30秒後にカメラ映像がHDMI出力される

## トラブルシューティング
- **カメラ認識しない**: ケーブル接続、カメラ有効化設定確認
- **HDMI出力されない**: ケーブル、モニター対応解像度確認  
- **映像が表示されない**: カメラレンズフォーカス、照明条件確認

## 開発完了状態
- ✅ 全設定ファイル作成済み
- ✅ 完全なビルド手順書作成済み
- ✅ カスタマイズ方法記載済み
- ❌ 実際のsdcard.imgビルド (Linux環境必須)

## 次のステップ
1. Linux環境でのビルド実行
2. Raspberry Pi実機での動作テスト
3. 必要に応じた設定調整

---
> Source: [ytx/broot0](https://github.com/ytx/broot0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
