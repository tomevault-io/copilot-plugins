## autonomous-ai-investment-lab

> Tài liệu này áp dụng cho toàn bộ coding agents, automation agents và các phiên làm việc kỹ thuật trong dự án **Autonomous AI Investment Lab**.

# Autonomous AI Investment Lab — Agent Governance Rules

Tài liệu này áp dụng cho toàn bộ coding agents, automation agents và các phiên làm việc kỹ thuật trong dự án **Autonomous AI Investment Lab**.

## 1. Thứ tự ưu tiên bắt buộc

Khi có mâu thuẫn, tài liệu có độ ưu tiên thấp hơn không được ghi đè tài liệu có độ ưu tiên cao hơn:

1. Binding Reviewer Amendments mới nhất.
2. Prompt của phase hiện tại.
3. Domain Contract đã được Reviewer duyệt.
4. `AGENTS.md`.
5. Skill chuyên môn liên quan.
6. Quy ước hoặc suy luận mặc định của coding agent.

Agent phải dừng triển khai và báo cáo rõ mâu thuẫn khi không thể giải quyết bằng thứ tự ưu tiên trên.

Không được âm thầm chọn một cách hiểu thuận tiện hoặc tự thay đổi thiết kế đã khóa.

---

## 2. Quy trình làm việc bắt buộc

Mọi phase phải đi qua đầy đủ các trạng thái:

1. `PLAN ONLY`
2. Reviewer approval
3. `IMPLEMENT`
4. Test/build evidence
5. Manual review
6. Reviewer quyết định PASS hoặc yêu cầu sửa

Agent không được:

* Tự chuyển từ PLAN ONLY sang IMPLEMENT.
* Implement khi chưa có phê duyệt rõ ràng.
* Tự chuyển sang phase tiếp theo.
* Tự đánh dấu phase là PASS.
* Triển khai yêu cầu chưa được Reviewer phê duyệt.
* Mở rộng phạm vi vì cho rằng việc đó “hợp lý” hoặc “tiện hơn”.

Chỉ Reviewer có quyền:

* Phê duyệt plan.
* Cho phép implement.
* Chấp nhận thay đổi phạm vi.
* Chốt PASS.
* Cho phép chuyển sang phase tiếp theo.

---

## 3. Kiểm tra trước khi bắt đầu công việc

Trước khi sửa bất kỳ file nào, agent phải xác định và ghi rõ:

* Phase hiện tại.
* Trạng thái hiện tại: PLAN ONLY hay IMPLEMENT.
* Phạm vi được phép.
* Binding Reviewer Amendments đang áp dụng.
* Các entity hoặc module được phép thay đổi.
* Danh sách file dự kiến tạo hoặc sửa.
* Các test và build command dự kiến chạy.
* Những thành phần bị loại khỏi phạm vi.

Nếu không xác định được một trong các thông tin trên, agent phải dừng và báo cáo thay vì tự suy đoán.

---

## 4. Kiểm soát phạm vi và thay đổi

Agent phải:

* Chỉ sửa file cần thiết cho phase hiện tại.
* Giải thích rõ khi cần sửa file ngoài danh sách dự kiến.
* Giữ nguyên kiến trúc và Domain Contract đã khóa.
* Ưu tiên thay đổi nhỏ, có thể kiểm tra và dễ rollback.
* Tách domain logic khỏi framework và presentation khi thiết kế yêu cầu như vậy.
* Giữ tương thích với dữ liệu và migration hiện có.

Agent không được:

* Refactor diện rộng ngoài yêu cầu.
* Đổi tên hoặc cấu trúc domain đã khóa mà không có phê duyệt.
* Tự thêm entity, bảng, API hoặc module của phase sau.
* Thêm tính năng “để dùng sau”.
* Thay đổi policy nghiệp vụ để làm code dễ hơn.
* Xóa hoặc làm yếu đi invariant đã được duyệt.
* Sửa tài liệu cũ để che giấu sự khác biệt với implementation.

---

## 5. Database safety

### Quy tắc chung

* Không reset database.
* Không xóa database.
* Không xóa hoặc ghi đè dữ liệu thật.
* Không chạy destructive migration nếu chưa được Reviewer duyệt.
* Database test phải tách biệt hoàn toàn với development và production.
* Migration phải được tạo ở chế độ cho phép review trước khi apply.
* SQL migration phải được kiểm tra phạm vi và tác động.

### Các lệnh bị cấm mặc định

Không được chạy:

* `prisma migrate reset`
* `prisma db push --force-reset`
* `DROP DATABASE`
* `DROP SCHEMA`
* Các script xóa hàng loạt dữ liệu
* Các lệnh truncate dữ liệu ngoài test database tạm thời

Chỉ được sử dụng thao tác phá dữ liệu trong database test cô lập khi prompt hiện tại cho phép rõ ràng.

### Migration

Agent phải báo cáo:

* Tên migration.
* Bảng và trường được tạo hoặc thay đổi.
* Unique constraints.
* Foreign keys.
* Indexes.
* Triggers hoặc database guards.
* Nguy cơ mất dữ liệu.
* Cách rollback hoặc khôi phục nếu có.

Không được tạo bảng của phase sau.

---

## 6. Domain và invariant safety

Agent phải đọc Domain Contract và Binding Reviewer Amendments liên quan trước khi sửa domain.

Không được:

* Tự đổi state machine.
* Bỏ qua precondition của transition.
* Dùng UUID, wall-clock timestamp hoặc database insertion order trong deterministic business key hoặc deterministic hash.
* Chuyển BigInt VND qua JavaScript `number`.
* Chuyển PostgreSQL `DATE` qua timezone transformation làm thay đổi ngày.
* Update hoặc delete dữ liệu đã được xác định là append-only, sealed, posted hoặc immutable.
* Tách aggregate update và event insert thành hai transaction độc lập khi contract yêu cầu atomicity.

Khi một invariant chưa thể được bảo vệ đầy đủ, agent phải báo cáo rõ thay vì giả định application logic là đủ.

---

## 7. Test safety

Agent không được:

* Xóa test cũ để build xanh.
* Sửa expected result chỉ để phù hợp với code sai.
* Chuyển test đang lỗi thành `skip`, `todo` hoặc disabled mà không có phê duyệt.
* Mock mất chính invariant đang cần kiểm tra.
* Chỉ chạy TypeScript compile rồi tuyên bố phase hoàn thành.
* Nói test “should pass”, “likely passes” hoặc “expected to pass” khi chưa chạy thật.

Các test bắt buộc của phase phải được chạy thực tế.

Nếu một test không thể chạy, agent phải báo cáo:

* Command đã thử.
* Exit code.
* Lỗi thực tế.
* Nguyên nhân đã xác minh.
* Phạm vi chưa được kiểm chứng.
* Tác động đến quyết định nghiệm thu.

Không được gọi phase là hoàn thành khi test bắt buộc chưa chạy hoặc đang FAIL.

---

## 8. Build và verification evidence

Trước khi báo cáo hoàn thành, agent phải chạy các nhóm phù hợp với phase:

* Unit tests.
* Integration tests.
* Security tests.
* Concurrency/idempotency tests.
* Regression tests liên quan.
* Migration verification.
* Production build.
* Type checking và lint nếu dự án có cấu hình.

Mọi bằng chứng phải là kết quả thực tế từ command đã chạy.

Agent không được tự tạo hoặc yêu cầu screenshot để chứng minh UI.

Screenshot giao diện sẽ do người dùng kiểm tra thủ công và gửi Reviewer khi cần.

---

## 9. Báo cáo hoàn thành bắt buộc

Báo cáo cuối của mỗi IMPLEMENT phải có:

1. Phase và phạm vi đã triển khai.
2. Files created.
3. Files changed.
4. Lý do thay đổi từng nhóm file.
5. Schema fields.
6. Migration summary.
7. Database constraints và triggers.
8. State transitions hoặc invariants đã triển khai.
9. Security controls.
10. Seed data summary.
11. Test commands.
12. Test logs.
13. Exit code.
14. Tổng số test PASS.
15. Tổng số test FAIL.
16. Test bị skip và lý do.
17. Build command.
18. Build log.
19. Known limitations.
20. Những nội dung chưa hoàn thành.
21. Manual verification checklist.

Không được che giấu:

* Test thất bại.
* Build warning quan trọng.
* Migration conflict.
* Security limitation.
* Phần code chưa được kiểm chứng.

---

## 10. Quy tắc trung thực khi báo cáo

Agent chỉ được sử dụng từ `PASS` cho một command khi:

* Command đã được chạy.
* Exit code thành công.
* Không có test bắt buộc bị bỏ qua trái phép.
* Log thực tế được cung cấp.

Agent không được tuyên bố:

* “Phase PASS”.
* “Implementation fully complete”.
* “Production ready”.

Quyết định nghiệm thu cuối cùng thuộc về Reviewer.

Cách diễn đạt phù hợp:

> Implementation đã hoàn thành trong phạm vi được giao. Các command được liệt kê đã chạy với kết quả đính kèm và đang chờ Reviewer nghiệm thu.

---

## 11. Stop conditions

Agent phải dừng và báo cáo ngay khi gặp một trong các trường hợp:

* Prompt IMPLEMENT chưa được phê duyệt.
* Yêu cầu hiện tại vượt phạm vi phase.
* Có mâu thuẫn giữa Reviewer Amendment và tài liệu khác.
* Migration có nguy cơ mất dữ liệu.
* Phải reset database mới tiếp tục được.
* Cần thêm entity hoặc bảng của phase sau.
* Cần thay đổi Domain Contract đã khóa.
* Test bắt buộc không thể chạy.
* Build thất bại.
* Phát hiện secret trong source hoặc log.
* Phát hiện dữ liệu production có nguy cơ bị ảnh hưởng.
* Không thể bảo đảm invariant quan trọng.
* Không thể xác minh kết quả đã triển khai.

Khi dừng, agent phải mô tả vấn đề cụ thể và không tự triển khai phương án thay thế chưa được phê duyệt.

---

## 12. Quy tắc kết thúc tác vụ

Agent phải dừng sau khi hoàn thành đúng phạm vi được giao.

Không được:

* Tự bắt đầu phase tiếp theo.
* Tự áp dụng đề xuất ngoài phạm vi.
* Tự tiếp tục refactor.
* Tự tạo thêm tính năng.
* Tự chạy migration production.
* Tự tuyên bố PASS.

Báo cáo cuối phải kết thúc bằng trạng thái:

> Hoàn thành implementation trong phạm vi được giao và đang chờ Reviewer nghiệm thu.

---
> Source: [phamdanghung/autonomous-ai-investment-lab](https://github.com/phamdanghung/autonomous-ai-investment-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
