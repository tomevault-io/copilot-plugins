## kysoqr

> XSign refactor — 1 app Next.js, không DB.

# CLAUDE.md

XSign refactor — 1 app Next.js, không DB.

- **Trước khi code bất kỳ gì**: đọc và tuân thủ [`Rule/RULES.md`](Rule/RULES.md) — quy tắc cố định (lệnh build/test/lint, Definition of Done, ngưỡng bật Plan Mode, Verification Loop bắt buộc sau mỗi lần sửa code). Đây không phải gợi ý, là bắt buộc.
- **API**: nguồn sự thật là OpenAPI [`public/openapi/vi.yaml`](public/openapi/vi.yaml) + [`public/openapi/en.yaml`](public/openapi/en.yaml) (xem tương tác tại `/docs`). Khi sửa route trong `app/api/` hoặc đổi request/response/mã lỗi, cập nhật cả 2 file YAML trong cùng thay đổi — test `lib/openapi.test.ts` sẽ fail nếu route hoặc mã lỗi trong code lệch với spec.
- **Kiến trúc/quyết định thiết kế**: xem `Plan/Plan refactor.md` nếu có — tài liệu kế hoạch nội bộ, cố tình không nằm trong git/repo public; hỏi chủ dự án nếu bạn không có sẵn file này.

---
> Source: [KysoQR/KysoQR](https://github.com/KysoQR/KysoQR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
