## taiwan-traditional-chinese

> 中文書寫須符合台灣繁體正體常見說法；含可手動維護的用語對照表


# 台灣繁體正體用語

撰寫、修改、產出任何中文內容時（含回覆、註解、commit message、Markdown、YAML、Skill 說明等），須使用**台灣繁體正體**的常見說法，避免大陸簡體用語、大陸慣用詞或不符合台灣語感的寫法。寫完後用台灣鄉民的角度看一遍，如果不順就修正到順為止。

## 基本原則

- 使用繁體正體字，不用簡體字
- 優先採台灣常見詞彙與語序
- 專有名詞若業界在台灣有慣用譯法，以台灣慣用為準
- 不確定時，選台灣媒體、政府公文、本土品牌常見的說法

## 用語對照表

`.cursor/rules/taiwan-traditional-chinese.lexicon.yaml`

@taiwan-traditional-chinese.lexicon.yaml

## Agent 行為

- 套用本 rule 時，須讀取 `@taiwan-traditional-chinese.lexicon.yaml`（或上述路徑）；若內容尚未在 context 中，須以 Read 工具讀取後再產出或修改中文
- 依 lexicon 的「避免 → 建議」對照替換用語
- 對照表未列出的詞，仍須以台灣繁體正體語感判斷，勿引入中國（俗稱：大陸、支那）慣用詞或支語

---
> Source: [maylogger/AGI](https://github.com/maylogger/AGI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
