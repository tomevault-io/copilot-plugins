## jimakuchan

> - セッション開始時に共通ルールである、AGENTS.mdを必ず読み込むこと。

# CLAUDE.md

- セッション開始時に共通ルールである、AGENTS.mdを必ず読み込むこと。
- 読み込んだことを最初に報告すること
- 以下は Claude Code固有の差分のみ記載する

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

jimakuChan (音声認識字幕ちゃん) is a real-time speech recognition and translation subtitle application for streamers and content creators. The application provides live subtitles with multi-language translation support and customizable visual styling.

## Architecture

The application consists of three main components:

1. **Configuration Interface** (`index.html`) - A comprehensive settings panel where users configure:
   - Speech recognition language settings
   - Translation target languages (up to 3 simultaneous translations)
   - Visual styling (fonts, colors, borders, sizes)
   - Google Translate API integration
   - Bouyomi-chan TTS integration

2. **Subtitle Display** (`main.html`) - The actual subtitle overlay that:
   - Performs real-time speech recognition using Web Speech API
   - Displays recognized text with visual effects (stroke, colors, positioning)
   - Handles translation via Google Translate API
   - Supports multiple simultaneous translation outputs
   - Manages text timing and cleanup

3. **Development Server** (`run_server.py`) - HTTPS localhost server for testing

## Key Features

- **Multi-language Recognition**: Supports 20+ languages for speech recognition
- **Multi-translation Support**: Up to 3 simultaneous translation outputs
- **Visual Customization**: Extensive font, color, size, and positioning controls
- **Layered Text Rendering**: Uses background/foreground/invisible layers for stroke effects
- **Content Filtering**: Optional profanity filtering with allow/block word lists
- **Bouyomi-chan Integration**: WebSocket client for Japanese TTS software
- **Settings Persistence**: Local storage with import/export functionality

## Development Commands

### Local HTTPS Server
```bash
python run_server.py
```
Starts HTTPS server on localhost:4443 (required for Web Speech API)

## 🚨 CRITICAL DEVELOPMENT RULES 🚨

### **NEVER COMMIT WITHOUT TESTING FIRST**
- **絶対にテスト前にコミットしてはいけません**
- 必ず以下の手順を守ること:
  1. コード修正完了
  2. ユーザーがテスト実施（サーバー起動は**ユーザーが行う**）
  3. 実際の動作確認完了
  4. 問題がないことを確認後にのみコミット
- この手順を飛ばすことは**絶対禁止**

### **NEVER START SERVER AUTOMATICALLY**
- **`python run_server.py` などのサーバー起動コマンドを実行してはいけません**
- サーバー起動は**必ずユーザーが手動で行う**
- Bashツールでサーバー起動コマンドを実行することは**絶対禁止**

## File Structure
- `index.html` - Configuration interface (loads `main.html` in iframe)
- `main.html` - Subtitle display with speech recognition engine
- `js/bouyomichan_client.js` - WebSocket client for TTS integration
- `font/` - Custom font files for visual styling
- `font.css` - Font face definitions

## Configuration Flow

1. User configures settings in `index.html`
2. Settings are stored in localStorage and URL parameters
3. `main.html` is loaded in iframe with configuration as URL parameters
4. Speech recognition starts automatically and displays real-time results

## Translation Integration

Uses Google Apps Script as a proxy for Google Translate API:
- Requires `gas_key` parameter for API access
- Supports source language detection and target language specification
- Handles JSON responses with translation count tracking

## Text Rendering System

The application uses a three-layer text rendering approach:
- `*-bg`: Background stroke layer
- `*-fg`: Foreground text layer  
- `*-imb`: Invisible positioning layer (same color as background)

This creates customizable text outlines and ensures proper text positioning across different fonts and sizes.

## Beta Version

The repository includes beta versions (`beta_index.html`, `beta_main.html`) which may contain experimental features or modifications under development.

## SSL Certificates

The development server uses `localhost.pem` and `localhost-key.pem` for HTTPS support. These are required because the Web Speech API only works over secure connections.

---
> Source: [sayonari/jimakuChan](https://github.com/sayonari/jimakuChan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
