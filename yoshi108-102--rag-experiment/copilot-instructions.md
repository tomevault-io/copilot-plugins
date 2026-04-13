## rag-experiment

> > 自分の思いを深めるための、会話入口トリアージ型AIチャット

# Reflective Gate Chat

> 自分の思いを深めるための、会話入口トリアージ型AIチャット

---

## 🧠 背景

実習や技能継承の現場において重要なのは、
知識を教わることではなく、

**自分の思いを言語化すること**

である。

本プロジェクトは、

- 作業内容を理解したAIに話しかける
- 思考を深めるかどうかを入口で判断する
- 必要に応じて深掘りを促す

という、**思考促進型チャットツール**の最小実装を目的とする。

---

## 🎯 コンセプト

このツールは「解決策を出すAI」ではない。

目的は：

> 思考を止めず、必要なら深める。

そのために、会話開始時に必ず以下の3分類を行う。
DEEPEN | PARK | CLARIFY


---

## 🚦 会話入口トリアージ

ユーザーが話し始めたとき、AIはまず分類を行う。

### DEEPEN
迷い・判断・違和感・失敗・感情の揺れが含まれる話。

→ 深掘り質問を1つ返す。

---

### CLARIFY
話の核が不明確。  
背景説明のみで論点が見えない。

→ 核を1行で確認する質問を返す。

---

### PARK
雑談・報告・材料不足。  
今すぐ深掘る必要は薄い。

→ 保存するか、1点だけ掘るかを確認する。

---

## 🔧 出力仕様（最小）

入口判定AIは必ず以下のJSONを返す。

```json
{
  "route": "DEEPEN | PARK | CLARIFY",
  "reason": "10 words以内",
  "first_question": "次に聞く1問（1文）"
}
```

制約

解決策の提示は禁止

長文禁止

分類 + 理由 + 1問のみ

🧩 深掘りテンプレ（最小）

DEEPEN の場合、質問は以下のいずれかに限定する。

「どの瞬間に引っかかりを感じましたか？」

「そのとき、何を手がかりに判断しましたか？」

「他に選択肢はありましたか？」

🏗 アーキテクチャ（最小構成）
User Input
   ↓
Gate Model (JSON only)
   ↓
Route Branch
   ├── DEEPEN → Conversation Model
   ├── CLARIFY → Clarify Question
   └── PARK → Log or Confirm

ゲート判定は軽量モデルで十分。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yoshi108-102)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yoshi108-102)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
