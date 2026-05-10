## calm-enter

> AIチャットサイトで Enter による誤送信を防ぐ Chrome 拡張機能。

# Calm Enter

AIチャットサイトで Enter による誤送信を防ぐ Chrome 拡張機能。
Enter = 改行、Cmd+Enter = 送信に変更する。

## 技術スタック

- Chrome Extension Manifest V3
- Vanilla JavaScript（バンドラ不要）
- Chrome APIs: storage, action

## ファイル構成

```
calm-enter/
├── manifest.json       # Manifest V3
├── content.js          # キーイベント制御（capture phase）
├── background.js       # ON/OFF状態管理（Service Worker）
├── icons/              # ツールバーアイコン（ON/OFF 各2サイズ）
└── docs/plans/         # 設計・実装計画
```

## 対象サイト

- claude.ai
- chatgpt.com
- gemini.google.com
- www.perplexity.ai

## 設計上の重要ポイント

- **capture phase** で keydown を登録（サイト側より先にイベントを捕まえる）
- **`e.isComposing`** チェック必須（日本語IME変換確定のEnterをブロックしない）
- **Shift+Enter** はスルー（既存の改行挙動を壊さない）
- **デフォルト ON**（インストール直後から有効）

## 開発方法

1. `chrome://extensions/` でデベロッパーモード ON
2. 「パッケージ化されていない拡張機能を読み込む」でこのディレクトリを選択
3. コード変更後は拡張機能ページでリロード

## Git運用

- Conventional Commits 形式、日本語
- main 直接コミット

## 実装計画

`docs/plans/2026-02-25-calm-enter-implementation.md` を参照。
`superpowers:executing-plans` スキルで実行すること。

---
> Source: [ashunar0/calm-enter](https://github.com/ashunar0/calm-enter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
