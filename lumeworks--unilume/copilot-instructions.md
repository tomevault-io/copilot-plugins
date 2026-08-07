## unilume

> File này là quy tắc bắt buộc đối với mọi AI coding agent làm việc trong

# Hợp đồng làm việc cho AI coding agent

File này là quy tắc bắt buộc đối với mọi AI coding agent làm việc trong
repository UniLume.

## Thứ tự đọc

Trước khi sửa code, AI phải đọc theo thứ tự:

```text
README.md
CONTRIBUTING.md
AGENTS.md
Issue được giao
Các tài liệu kiến trúc liên quan
```

## Phạm vi và quyền hạn

1. Không code nếu chưa có Issue ở `Ready`.
2. Không tự chọn Issue từ `Inbox`.
3. Không tự chuyển Issue sang `Ready`.
4. Chỉ sửa đúng một Issue.
5. Không thay đổi ngoài scope và không sửa file không cần thiết.
6. Không bulk-format code kế thừa.
7. Không đổi thuật toán UniKey nếu Issue không yêu cầu.
8. Không đổi giấy phép, attribution hoặc copyright header.
9. Không thêm dependency hoặc abstraction dựa trên phỏng đoán.
10. Không tự viết backend Wayland/X11 lớn ngoài Issue.
11. Không sửa warning kế thừa hàng loạt chỉ để làm code đẹp.
12. Test phải dùng API thật; không mock vô nghĩa.
13. Phải xem `git diff` và `git status` trước commit.
14. Không commit nếu build hoặc test thất bại.
15. Không push hoặc merge nếu người dùng chưa giao quyền rõ ràng.

Khi Issue thiếu thông tin hoặc mâu thuẫn, AI không được đoán. Hãy dừng phần
thay đổi liên quan và báo blocker cụ thể. Khi phát hiện việc ngoài scope, ghi
lại, đề xuất Issue mới và không sửa ké.

## Báo cáo bắt buộc

Báo cáo cuối phải nêu:

- file đã thay đổi;
- quyết định kỹ thuật;
- lệnh build/test;
- kết quả;
- blocker;
- commit hash nếu có.

## Mẫu giao việc

```text
Thực hiện Issue #<id> của dismonjames/UniLume.

Đọc AGENTS.md và CONTRIBUTING.md trước khi sửa code.
Chỉ làm đúng phạm vi Issue.
Không tự mở rộng kiến trúc.
Build và test đầy đủ.
Không merge PR.
```

---
> Source: [LumeWorks/UniLume](https://github.com/LumeWorks/UniLume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
