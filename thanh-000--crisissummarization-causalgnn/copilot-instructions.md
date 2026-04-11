## crisissummarization-causalgnn

> > Rules cụ thể cho project này. Global rules ở `~/.gemini/GEMINI.md`.

# GEMINI.md — CrisisSummarization Project Rules

> Rules cụ thể cho project này. Global rules ở `~/.gemini/GEMINI.md`.
> Antigravity merge cả hai khi start conversation.

---

## 🚀 Skills Auto-Activation (MANDATORY)

Đầu mỗi conversation, đọc toàn bộ `SKILL.md` trong `.agent/skills/` và áp dụng skill tương ứng.

### Trigger Table — Project CrisisSummarization

| User đề cập | Skill | Action |
|------------|-------|--------|
| bug, lỗi, crash, error, CUDA, fail, không chạy | **`debug`** | Đọc `.agent/skills/debug/SKILL.md` → workflow evidence-first |
| feature mới, implement, thêm, xây dựng, thiết kế | **`dev-lifecycle`** | Đọc `.agent/skills/dev-lifecycle/SKILL.md` → bắt đầu Phase 1 |
| hiểu code, document, giải thích, map module | **`capture-knowledge`** | Đọc `.agent/skills/capture-knowledge/SKILL.md` → analyse first |
| refactor, đơn giản, clean, phức tạp, technical debt | **`simplify-implementation`** | Đọc `.agent/skills/simplify-implementation/SKILL.md` → plan first |
| review docs, README, báo cáo, tài liệu | **`technical-writer`** | Đọc `.agent/skills/technical-writer/SKILL.md` → rate 4 dims |
| nhớ lại, lưu, convention, trước đây, đã làm gì | **`memory`** | Đọc `.agent/skills/memory/SKILL.md` → search trước khi hỏi |

### Announce khi activate:
> *"Đang áp dụng skill: `<tên>` — [lý do 1 câu]"*

---

## 📌 Project Snapshot

| Field | Value |
|-------|-------|
| **Project** | CrisisSummarization |
| **Paper** | Multimodal Classification of Social Media Disaster Posts with GNN (IEEE Access 2025) |
| **Notebook chính** | `mm_class_experiment.ipynb` |
| **Stack** | Python · PyTorch · CLIP · GNN · Google Colab (GPU) |
| **Dataset** | CrisisMMD v2.0 · Brazilian 7Sept Protest Dataset |

---

## 📂 Documentation

```
docs/ai/
├── requirements/     ← Yêu cầu và mục tiêu nghiên cứu
├── design/           ← Kiến trúc GNN + CLIP pipeline
├── planning/         ← Task breakdown
├── implementation/   ← Implementation notes + knowledge docs
├── testing/          ← Experiment configs và kết quả
├── deployment/       ← Colab setup, Google Drive mounting
└── monitoring/       ← Theo dõi training metrics
```

---

## ⚙️ Memory — Luôn dùng CLI

```bash
# Tìm kiếm trước khi hỏi lại
npx ai-devkit@latest memory search \
  --query "<topic>" \
  --scope "project:CrisisSummarization"

# Lưu sau khi resolve
npx ai-devkit@latest memory store \
  --title "<title>" \
  --content "<solution>" \
  --tags "crisis,gnn,pytorch" \
  --scope "project:CrisisSummarization"
```

---

## ⚠️ Hard Rules

1. `debug` / `simplify-implementation`: **KHÔNG sửa code** trước khi user approve
2. `capture-knowledge`: **KHÔNG viết doc** trước khi phân tích xong
3. `memory`: **Search memory trước** khi hỏi câu lặp lại
4. `dev-lifecycle`: **Phase 1 trước** cho mọi feature mới

---

## 💻 Code Style

- Python PEP8 + type hints khi có thể
- Notebook cells: mỗi cell có header comment
- Tên biến: tiếng Anh, snake_case
- Comment giải thích được viết tiếng Việt

---

## 🔗 Xem thêm

- Global rules → `~/.gemini/GEMINI.md`
- Full rules → `CLAUDE.md`
- Skills → `.agent/skills/*/SKILL.md`
- Phase docs → `docs/ai/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Thanh-000)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Thanh-000)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
