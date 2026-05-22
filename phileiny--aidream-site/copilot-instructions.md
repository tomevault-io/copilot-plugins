## aidream-site

> 當使用者說「把 xxx.md 上架到 xx 類別」時，自動執行以下步驟：

# 文章上架指令

當使用者說「把 xxx.md 上架到 xx 類別」時，自動執行以下步驟：

## 類別對應

| 類別 | 目錄 | 索引頁 |
|------|------|--------|
| AI | `docs/ai/` | `docs/ai/index.md` |
| 機器人 / robot | `docs/robot/` | `docs/robot/index.md` |
| 專案 / project | `docs/project/` | `docs/project/index.md` |
| 學習 / learning | `docs/learning/` | `docs/learning/index.md` |

### 學習專區子分類

學習專區用於長期學習路徑，採用子目錄結構：

| 子分類 | 目錄 | 索引頁 |
|--------|------|--------|
| CS336 | `docs/learning/cs336/` | `docs/learning/cs336/index.md` |

## 上架步驟（自動執行）

### 一般分類（AI / 機器人 / 專案）

1. **讀取文章**：讀取指定的 `.md` 文件，提取標題（第一個 `# ` 開頭的行）
2. **確認文章位置**：確保文章已在正確的分類目錄下（如 `docs/ai/xxx.md`）
3. **更新分類索引頁**：在 `docs/{分類}/index.md` 的 `## Recent Posts` 下方第一行加入連結
4. **更新側邊欄**：在 `docs/.vitepress/config.ts` 對應分類的 sidebar items 中，Index 之後加入文章
5. **更新首頁**：在 `docs/index.md` 的 `## 📝 最新文章` 區塊最上方加入文章（需提供日期和摘要）
6. **本地預覽驗證**：執行 `npm run build` 確認無錯誤

### 學習專區（learning）

學習專區的上架方式稍有不同，因為採用子目錄結構：

1. **讀取文章**：讀取指定的 `.md` 文件，提取標題
2. **確認文章位置**：確保文章已在正確的子分類目錄下（如 `docs/learning/cs336/xxx.md`）
3. **更新子分類索引頁**：在 `docs/learning/{子分類}/index.md` 的對應區塊加入連結
4. **更新學習專區索引頁**：視需要更新 `docs/learning/index.md`
5. **更新側邊欄**：在 `docs/.vitepress/config.ts` 的 `/learning/` sidebar 對應子分類區塊加入文章
6. **更新首頁**：在 `docs/index.md` 的 `## 📝 最新文章` 區塊最上方加入文章（分類標籤使用「學習」）
7. **本地預覽驗證**：執行 `npm run build` 確認無錯誤

## 上架格式範例

### 一般分類

**分類索引頁格式**：
```markdown
- [文章標題](./文章檔名)
```

**側邊欄格式**：
```typescript
{ text: '文章標題', link: '/分類/文章檔名' }
```

### 學習專區

**子分類索引頁格式**：
```markdown
- [文章標題](./文章檔名)
```

**側邊欄格式**（在對應子分類區塊內）：
```typescript
{
  text: 'CS336 學習筆記',
  collapsed: false,
  items: [
    { text: '總覽', link: '/learning/cs336/' },
    { text: '文章標題', link: '/learning/cs336/文章檔名' }
  ]
}
```

### 首頁格式（通用）

```markdown
### [文章標題](/分類/文章檔名)
**分類標籤** · YYYY-MM-DD

文章摘要（1-2 句話）

---
```

學習專區文章的首頁格式：
```markdown
### [文章標題](/learning/子分類/文章檔名)
**學習** · YYYY-MM-DD

文章摘要（1-2 句話）

---
```

## 注意事項

- 首頁最新文章區塊只保留最新 5 篇，超過的移除
- 日期格式統一使用 `YYYY-MM-DD`
- 分類標籤使用：`AI`、`機器人`、`專案`、`學習`
- 若使用者未提供日期和摘要，需詢問
- 學習專區的文章路徑包含子分類目錄（如 `/learning/cs336/xxx`）

## 新增學習路徑子分類

若要新增新的學習路徑（如新課程），需要：

1. 建立子目錄：`docs/learning/{新子分類}/`
2. 建立索引頁：`docs/learning/{新子分類}/index.md`
3. 更新 `docs/learning/index.md` 加入新學習路徑連結
4. 更新 `docs/.vitepress/config.ts` 的 `/learning/` sidebar 加入新區塊：
   ```typescript
   {
     text: '新學習路徑名稱',
     collapsed: false,
     items: [
       { text: '總覽', link: '/learning/{新子分類}/' }
     ]
   }
   ```

---
> Source: [phileiny/aidream_site](https://github.com/phileiny/aidream_site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
