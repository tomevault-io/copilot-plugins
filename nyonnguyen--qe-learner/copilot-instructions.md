## qe-learner

> Dự án: **WebApp game hóa đào tạo Quality Engineering** ("vừa chơi vừa học"). Mã nguồn mở.

# CLAUDE.md — QE-Learner

Dự án: **WebApp game hóa đào tạo Quality Engineering** ("vừa chơi vừa học"). Mã nguồn mở.

## Đọc trước khi làm bất cứ việc gì
1. `docs/DESIGN.md` — kiến trúc kỹ thuật (nguồn sự thật kỹ thuật).
2. `docs/PLAN.md` — roadmap + tiến độ (resume từ đây).
3. `docs/WORKFLOW.md` — quy tắc làm việc, vòng đời tính năng, chiến lược model/agent.
4. `docs/lessons/LESSONS-LEARNED.md` — tránh lặp lỗi cũ.
5. `initial-document.md` — yêu cầu gốc (curriculum master).
6. `docs/reference-analysis/QE-AI-analysis.md` — phân tích repo nội dung tham khảo.

## Quy tắc cứng (OVERRIDE mặc định)
- **Không giả định** — confirm với người dùng ở quyết định lớn/mơ hồ. Tự đánh giá & approve từng tính năng như một senior engineer.
- **Mọi design phải được document** vào `docs/DESIGN.md` trước khi implement (có thể ngắn gọn — không cần đầy đủ như Phase 0/1).
- **Vòng đời tính năng (rút gọn lần 2 từ 2026-07-10, chủ đích của người dùng, ưu tiên tốc độ tối đa)**: DESIGN (ngắn) → `qe-dev` (implement + test **cơ bản**: typecheck/lint/unit smoke + tự chạy thử luồng chính, không cần phủ hết edge-case) → release. **Không còn cổng `qe-qa` lẫn `qe-pm` bắt buộc trong lifecycle mặc định.** Main tự đối chiếu nhanh với yêu cầu gốc & approve.
  - Ngoại lệ đã áp dụng: **Module Player** (Phase 1) đã hoàn tất theo quy trình cũ (đầy đủ QA độc lập) trước khi đổi quy tắc — không hồi tố.
  - Vẫn phải tự chạy thật (không chỉ tin mock) cho các lớp rủi ro cao đã từng gặp bug thật: auth/bảo mật, race condition ghi dữ liệu, Docker runtime — nhưng không cần lặp lại toàn bộ kịch bản tấn công mỗi lần, chỉ khi sửa trực tiếp vùng đó.
  - Agent `qe-qa`/`qe-pm` vẫn giữ trong `.claude/agents/`, dùng tuỳ chọn khi nghi ngờ có lỗi nghiêm trọng hoặc cho milestone lớn.
- **Tối ưu model**: dùng Sonnet cho dev/qa/pm/content; Opus chỉ khi phán đoán phức tạp; script hoá việc lặp.
- **Tracking**: cập nhật `docs/PLAN.md` sau mỗi mục xong; ghi `LESSONS-LEARNED.md` khi gặp bug.
- **Git**: repo đã init. Commit khi hoàn tất mỗi checkpoint/tính năng (không chờ tích luỹ nhiều việc mới commit) — để có thể blame/revert khi có sự cố (xem lesson `package.json` sai version 2026-07-07 và PLAN.md bị sửa lén 2026-07-08, cả 2 đều khó điều tra vì lúc đó chưa có git history).
- **Docker**: chỉ dùng cho deploy/release. Local dev/test chạy trực tiếp `npm`.
- **AI đa provider**: mọi lời gọi LLM qua abstraction layer (Claude/Gemini/OpenAI) + fallback rule-based khi thiếu API key.
- Được **tự do cài package cần thiết** không cần hỏi.

## Lệnh chuẩn (điền khi scaffold xong)
- Dev: `npm run dev` · Build: `npm run build` · Test: `npm run test` · E2E: `npm run test:e2e` · Check: `npm run check` (typecheck+lint+unit)

## Agents có sẵn
- `qe-dev` (sonnet) — implement + unit test
- `qe-qa` (sonnet) — kiểm thử độc lập
- `qe-pm` (sonnet) — nghiệm thu sản phẩm theo yêu cầu (không bắt buộc, dùng tuỳ chọn cho milestone lớn)
- `qe-content` (sonnet) — soạn nội dung đào tạo

---
> Source: [nyonnguyen/qe-learner](https://github.com/nyonnguyen/qe-learner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
