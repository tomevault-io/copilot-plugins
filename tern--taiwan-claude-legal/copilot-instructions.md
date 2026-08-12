## taiwan-claude-legal

> 提供台灣法律條文的自動更新與合約審查技能，支援 Gemini CLI（skill）與 Claude Code（plugin）。

# Taiwan Claude Legal — 專案情境

## 專案用途
提供台灣法律條文的自動更新與合約審查技能，支援 Gemini CLI（skill）與 Claude Code（plugin）。

## 目錄結構

```
laws/
  civil-code/          # 民法（B0000001）
  company-act/         # 公司法（J0080001）
  labor-standards-act/ # 勞動基準法（N0030001）
  pdpa/                # 個人資料保護法（I0050021）
  copyright-act/       # 著作權法（J0070017）
    articles.md        # 易讀版（audit 時讀這份）
    full.json          # 機器可讀版

skills/taiwan-legal-audit/SKILL.md   # 審查邏輯主檔
scripts/update_laws.py               # 法規爬蟲（Python）
```

## 法規更新

```bash
python3 scripts/update_laws.py
```

依賴：`requests`, `beautifulsoup4`（見 `requirements.txt`）。
每日由 GitHub Actions 自動執行。

## 輸出慣例

- **語言**：嚴格使用繁體中文（台灣用語，禁用「合同」應用「合約」）
- **引用格式**：`依據 [民法第 71 條](https://law.moj.gov.tw/LawClass/LawSingle.aspx?pcode=B0000001&flno=71)`
- **資料來源聲明**：全國法規資料庫（https://law.moj.gov.tw）
- **免責宣告**：每次法律分析結尾必須附上：
  > *注意：以上分析僅供參考，不構成正式法律意見。實際法律行為請諮詢專業律師。*

## 審查邏輯

詳見 `skills/taiwan-legal-audit/SKILL.md`，不在此重複。

---
> Source: [tern/taiwan-claude-legal](https://github.com/tern/taiwan-claude-legal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
