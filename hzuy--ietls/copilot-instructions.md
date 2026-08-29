## ietls

> Đây là quy tắc bắt buộc khi thực hiện bất kỳ tác vụ nào trong project này.

# Quy trình sử dụng Skill cho project ielts-app

Đây là quy tắc bắt buộc khi thực hiện bất kỳ tác vụ nào trong project này.
Luôn tra cứu và áp dụng đúng skill theo giai đoạn tương ứng bên dưới, theo đúng thứ tự.
Không bỏ qua bước kiểm thử/bảo mật để tiết kiệm thời gian.

## Nguyên tắc chung
- Trước khi bắt đầu bất kỳ feature/task mới nào, luôn xác định đang ở giai đoạn nào trong quy trình dưới đây.
- Không tự ý nhảy cóc giai đoạn (ví dụ: không code thẳng khi chưa có plan cho feature lớn).
- Với task nhỏ, đơn giản, có thể rút gọn nhưng vẫn phải qua bước test + security trước khi commit.

## Giai đoạn 1 — Ý tưởng & Lập kế hoạch
Dùng khi: bắt đầu feature mới, thay đổi kiến trúc, hoặc chưa rõ yêu cầu.
Thứ tự:
1. ak-interview-docs — làm rõ yêu cầu, ghi thành tài liệu
2. ak-research / ak-research-prompt — nghiên cứu công nghệ nếu cần
3. ak-brainstorm — so sánh phương án, phản biện
4. ak-plan — lập roadmap chi tiết
5. ak-predict — phản biện thiết kế trước khi code

## Giai đoạn 2 — Phát triển
Dùng khi: đã có plan rõ ràng, bắt đầu viết code.
Thứ tự:
1. ak-scout — khảo sát codebase liên quan trước khi sửa
2. ak-databases — schema/query nếu liên quan DB
3. ak-backend-development — API Express
4. ak-frontend-development + ak-react-best-practices — code React
5. ak-ui-styling + ak-frontend-design + ak-ui-ux-pro-max — giao diện
6. ak-cook — triển khai tính năng theo pipeline có cấu trúc
7. ak-mermaidjs-v11 — vẽ sơ đồ nếu cần minh họa

## Giai đoạn 3 — Kiểm thử & Rà soát (BẮT BUỘC trước khi commit)
Thứ tự:
1. ak-scenario — sinh test case/edge case
2. ak-test / ak-web-testing — viết và chạy test
3. ak-debug — nếu có lỗi, tìm root cause trước khi sửa
4. ak-fix — sửa lỗi sau khi đã rõ nguyên nhân
5. ak-code-review — tự review code
6. ak-security / ak-security-scan — audit bảo mật, quét secret rò rỉ
7. ak-web-design-guidelines — review UI/UX/accessibility

## Giai đoạn 4 — Commit & Release
Thứ tự:
1. ak-git — commit theo convention
2. ak-review-pr — review PR (nếu có)
3. ak-ship — pipeline merge → test → review → push → PR

## Giai đoạn 5 — Triển khai
Thứ tự:
1. ak-devops — cấu hình hạ tầng
2. ak-deploy — deploy lên Render/Vercel
3. seo-analysis — tối ưu SEO nếu cần

## Giai đoạn 6 — Tài liệu & Bảo vệ đồ án
Thứ tự:
1. ak-docs — sinh/audit tài liệu kỹ thuật
2. ak-document-skills — xuất báo cáo Word/PDF
3. ak-show-off — trang demo cho buổi bảo vệ
4. ak-journal — nhật ký kỹ thuật
5. ak-retro — tổng kết sprint

## Dùng khi cần (không theo thứ tự cố định)
ak-ask, ak-advise, ak-sequential-thinking, ak-problem-solving,
ak-docs-seeker, ak-repomix, ak-gkg, ak-graphify, ak-tech-graph,
ak-project-management, ak-plans-kanban, ak-watzup,
ak-agent-browser, ak-chrome-profile

## Quy tắc tối giản cho công việc hàng ngày (project 1 người làm)
Nếu không chắc dùng skill nào, ưu tiên theo trình tự lõi:
ak-plan → ak-cook → ak-test + ak-security → ak-git + ak-ship → ak-deploy

---
> Source: [hzuy/ietls-](https://github.com/hzuy/ietls-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
