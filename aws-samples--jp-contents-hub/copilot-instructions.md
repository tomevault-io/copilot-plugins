## jp-contents-hub

> JP Contents Hub に掲載されているハンズオン・ワークショップの URL が正常にアクセスできるかを確認するツール。

# URL チェックツール - エージェント仕様書

## 概要

JP Contents Hub に掲載されているハンズオン・ワークショップの URL が正常にアクセスできるかを確認するツール。

## 背景

- JP Contents Hub は AWS 日本語ハンズオン一覧を掲載する GitHub Pages サイト
- リポジトリ: `aws-samples/jp-contents-hub`（Public, MIT-0 ライセンス）
- ハンズオン URL は時間経過で削除・移動されることがあり、定期的なリンク切れチェックが必要

## ツール構成

```
tools/url-check/
├── agent.md              # この仕様書
├── check_links.py        # HTTPステータスチェックスクリプト
├── errors.tsv            # チェック結果（エラー一覧）
└── ok_urls.txt           # HTTP 200 を返したURL一覧
```

## チェック方法（2段階）

### Phase 1: HTTP ステータスチェック（スクリプト）

`check_links.py` が `content/index.md` から全 URL を抽出し、HTTP ステータスコードを確認する。

- **200 以外** → 即エラー判定（DNS解決不可、タイムアウト、4xx/5xx）
- **200** → `ok_urls.txt` に出力（Phase 2 で内容確認）

### Phase 2: コンテンツ確認（Playwright MCP）

HTTP 200 を返す URL の中には、ページ自体は存在するがコンテンツが「Page not found」になっているケースがある（特に Workshop Studio 系）。

Playwright MCP を使ってブラウザでアクセスし、以下を確認：
- ページタイトルが正常か
- `<h2>Page not found</h2>` が存在しないか
- コンテンツが実際に表示されているか

#### Playwright MCP 設定

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--headless", "--browser", "chromium"],
      "env": {}
    }
  }
}
```

#### バッチチェック用コード例

```javascript
async (page) => {
  const urls = [/* チェック対象URL */];
  const results = [];
  for (const url of urls) {
    try {
      await page.goto(url, { waitUntil: 'domcontentloaded', timeout: 15000 });
      await page.waitForTimeout(800);
      const notFound = await page.$('h2:has-text("Page not found")');
      if (notFound) results.push(url);
    } catch (e) {
      results.push(url + ' [ERROR]');
    }
  }
  return results;
}
```

## 検出されるエラーパターン

| パターン | 原因 | 検出方法 |
|---|---|---|
| DNS_ERROR (HTTP 0) | CloudFront ディストリビューション削除 | Phase 1 |
| HTTP 301/403/404 | リソース移動・削除 | Phase 1 |
| S3 PermanentRedirect | バケット設定変更 | Phase 1 + Phase 2 |
| PAGE_NOT_FOUND (HTTP 200) | Workshop Studio のソフト404 | Phase 2 のみ |

## 実行方法

```bash
# Phase 1: HTTPステータスチェック
cd /path/to/jp-contents-hub
python3 tools/url-check/check_links.py

# Phase 2: Playwright MCP でコンテンツ確認
# Kiro CLI チャットで ok_urls.txt の URL を Playwright で確認
```

## LLM 判定（代替案）

Phase 2 の代替として、Amazon Bedrock Claude Haiku 4.5 を使ったコンテンツ判定も検討した。

- モデル: `us.anthropic.claude-haiku-4-5-20251001-v1:0`（inference profile 経由）
- 呼び出し方法: `aws bedrock-runtime invoke-model --body fileb://req.json`
- メリット: 自動化しやすい
- デメリット: コスト、速度、判定精度のトレードオフ

現時点では Playwright MCP による目視確認の方が確実。

## 2026-05-09 チェック結果

- 全 URL 数: 242
- 正常: 186
- リンク切れ: 56 件
  - DNS 解決不可: 25 件（`dcj71ciaiav4i.cloudfront.net` ドメイン消滅）
  - S3 エラー: 1 件
  - Page not found (HTTP 200): 30 件（Workshop Studio ソフト404）

詳細は `errors.tsv` を参照。

---
> Source: [aws-samples/jp-contents-hub](https://github.com/aws-samples/jp-contents-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
