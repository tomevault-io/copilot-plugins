## marp-nvim

> Marp用Neovimプラグイン。Markdownからプレゼンテーション作成・リアルタイムプレビュー・各種エクスポートを提供。

# marp.nvim

Marp用Neovimプラグイン。Markdownからプレゼンテーション作成・リアルタイムプレビュー・各種エクスポートを提供。

## Structure

- `lua/marp.lua` — メイン実装（1040行）
- `plugin/marp.vim` — Vimコマンド定義（48行）

## Quality

```bash
make check   # luacheck + stylua（コミット前に必須）
make format  # stylua 自動修正
```

## Commands

| Command | Description |
|---------|-------------|
| `:MarpWatch` | ライブプレビュー開始 |
| `:MarpStop` / `:MarpStopAll` | プレビュー停止 |
| `:MarpExport [format]` | エクスポート（html/pdf/pptx/png/jpeg/notes） |
| `:MarpThumbnail [format]` | 最初のスライドのみ画像化（png/jpeg） |
| `:MarpTheme` / `:MarpSnippet` | テーマ設定・スニペット挿入 |
| `:MarpInfo` / `:MarpDebug` | 情報表示・診断 |

## Config Defaults

```lua
require('marp').setup({
  marp_command = "marp",        -- PATH上のmarpを使用
  server_mode = false,          -- false=watch, true=server
  html_option = true,           -- watchで--html使用
  allow_local_files = true,     -- ローカルファイルアクセス
  pdf_notes = false,            -- PDFノート付与
  pdf_outlines = false,         -- PDFブックマーク
  pptx_editable = false,        -- 編集可能PPTX
  image_scale = 1,              -- 画像倍率
  jpeg_quality = 85,            -- JPEG品質
  theme_set = {},               -- カスタムテーマCSS
})
```

## Troubleshooting

- `:MarpDebug` でCLI環境を診断
- Node.js v25+ ESMエラー → `marp_command = "marp"` でshebang経由実行
- プロセス残留 → `:MarpStopAll`

---
> Source: [nwiizo/marp.nvim](https://github.com/nwiizo/marp.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
