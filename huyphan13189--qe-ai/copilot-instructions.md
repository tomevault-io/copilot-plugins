## analyzing-bugs-and-logs

> Dùng AI đọc log/stacktrace để khoanh vùng nguyên nhân, triage mức độ, và soạn bug report tái hiện được. Dùng khi có lỗi 500/stacktrace cần tìm nguyên nhân, hoặc log quá dài cần tóm tắt.


# Analyzing Bugs and Logs with AI

Khi cần phân tích log/stacktrace để tìm nguyên nhân lỗi, ĐỌC và LÀM THEO nguyên văn skill đầy đủ tại
`.claude/skills/analyzing-bugs-and-logs/SKILL.md`.

Nguyên tắc cốt lõi: AI tóm tắt log và đoán nguyên nhân rất nhanh, rất TỰ TIN — kể cả khi sai. Output
của AI là GIẢ THUYẾT cần tái hiện để kiểm chứng, không phải kết luận. Làm sạch secret/PII trong log
trước khi đưa cho AI, và không viết bug report cho đến khi nguyên nhân đã được xác nhận.

---
> Source: [huyphan13189/QE-AI](https://github.com/huyphan13189/QE-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
