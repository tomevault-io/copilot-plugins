## mazelingo

> Sentence-level bilingual mix translator。Webページのテキストを文単位で外国語とミックス表示する語学学習向けChrome拡張(Manifest V3)。

# Mazelingo

Sentence-level bilingual mix translator。Webページのテキストを文単位で外国語とミックス表示する語学学習向けChrome拡張(Manifest V3)。

Chrome Web Store: https://chromewebstore.google.com/detail/mazelingo/bhdngeocokoeblnnlhjibojcadefimpi

## ビルド・開発

```bash
npm install        # 依存: dotenv, terser (devDependenciesのみ)
npm run build      # terserで圧縮 → dist/ 生成 + mazelingo.zip (Chrome Web Store配布用)
```

- フレームワーク不使用の Vanilla JS (ESM)。ビルドなしで `chrome://extensions` から開発版(このフォルダ)を直接ロード可能
- test/ に LLM モデル動作検証スクリプト (test_llm.js, test_models.js, test_deepl.js)。`.env` にAPIキーを置いて実行(`.env.example` 参照)

## 構造

- `manifest.json` — MV3 定義
- `background.js` — service worker。翻訳バッチ・2層キャッシュ・長ユニット再分割
- `content_script.js` + `content_style.css` — ページ注入(リーフブロック検出・改行境界での断片化)・翻訳表示UI(トグル/ツールチップ)
- `popup.html` / `popup.js` — サイドパネルUI(設定/出力タブ、最大規模ファイル)
- `options.html` / `options.js` — 設定ページ
- `llm.js` — OpenAI / Anthropic / Gemini / GLM / DeepSeek / OpenRouter のリクエストビルダーと呼び分け (callLLMChain)
- `config.js` — デフォルト設定とマージ処理
- `vocab_data.json` / `situations.json` — 学習コンテンツ (web_accessible_resources)
- `docs/` — privacy-policy.md(ストア申請用)・スクリーンショット

## ルール

- APIキーはユーザーが設定画面から入力し Chrome Storage に保存する設計。**ソースにハードコードしない**
- `.env` はローカル検証用(gitignore済み)。内容を転記・コミットしない
- 翻訳の文分割は「読みの単位」: 改行(文末記号のあとは決定的、それ以外はLLM判断)・句読点・長文は節で分割。プロンプトは `background.js` の translateBatch を参照

---
> Source: [Yeq6X/mazelingo](https://github.com/Yeq6X/mazelingo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
