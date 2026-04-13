## cmd-go

> "parent": { "database_id": {{ database_id }} },

# NotionAPIのドキュメント自動生成のrequestのフォーマット
## JSON のフォーマット
```json
[{
  "parent": { "database_id": {{ database_id }} },
  "properties": {
    "エンドポイント": {
      "title": [
        {
          "text": {
            "content": {{ endpoint }}
          }
        }
      ]
    },
    "概要": {
      "rich_text": [
        {
          "text": {
            "content": {{ summary }}
          }
        }
      ]
    }
  }
}]
```

### 条件
- {{ endpoint }} はページのタイトル
- {{ summary }} はページの概要
- それぞれの変数に定義したものを入れて./api.jsonを作成する

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hiroky1983) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
