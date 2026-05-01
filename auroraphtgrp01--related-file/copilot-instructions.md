## related-file

> - **Sử dụng tiếng Việt:** Tất cả câu trả lời, phân tích và suy nghĩ đều phải được viết bằng tiếng Việt, không sử dụng tiếng Anh.

# Quy Tắc Cho Copilot

## 1. Ngôn Ngữ

- **Sử dụng tiếng Việt:** Tất cả câu trả lời, phân tích và suy nghĩ đều phải được viết bằng tiếng Việt, không sử dụng tiếng Anh.

## 2. Phân Tích Và Trả Lời

- **Phân tích kỹ càng:** Cần phân tích chi tiết các khía cạnh của vấn đề trước khi đưa ra câu trả lời hoặc đoạn code. Mọi lý do và logic cần được giải thích rõ ràng.
- **Tập trung vào nghiên cứu sâu:** Đảm bảo tập trung vào việc hiểu rõ thuộc tính, dữ liệu và logic liên quan đến yêu cầu, đặc biệt là các quy trình, sự kiện và tương tác hệ thống.

## 3. Tổng Quan Dự Án

- **Về dự án:** Đây là ứng dụng đo sức khỏe qua nhẫn thông minh, bao gồm các tính năng như đo SPO2, nhịp tim, và thống kê giấc ngủ. Dữ liệu giấc ngủ được nhẫn tự động ghi lại và cần được giải mã khi kết nối với ứng dụng qua Bluetooth.
  
- **Mục tiêu của bạn:** Tìm hiểu và giải mã dữ liệu thống kê giấc ngủ từ nhẫn thông minh để đưa ra các thông số sau:
  - Thời gian đi ngủ
  - Thời gian thức dậy
  - Thời lượng giấc ngủ
  - Giấc ngủ sâu (Deep Sleep)
  - Giấc ngủ nhẹ (Light Sleep)
  - REM (Rapid Eye Movement)
  - Thời gian tỉnh giấc
  - Nhịp tim
  - SPO2
  
- **Những gì bạn đã biết:**
  - Nhẫn thông minh thực hiện thống kê giấc ngủ ngay cả khi không kết nối với app, nhưng cần phải kết nối trước để kích hoạt tính năng này.
  - Khi kết nối Bluetooth, app sẽ thực hiện một loạt các thao tác và sự kiện, bao gồm việc lấy và xóa dữ liệu giấc ngủ, đo SPO2, nhịp tim từ nhẫn.
  - Đã sử dụng **Frida** để debug quy trình kết nối nhẫn với ứng dụng thông qua file `debug.js`, với log được lưu trong file `log-connect.txt`.
  - Khả năng cao có sự thiết lập múi giờ giữa app và nhẫn, hoặc quy tắc về thời gian
  - Có thể có một số quy tắc hoặc mã hóa trong dữ liệu giấc ngủ, ví dụ như mã hóa thời gian đi ngủ và thức dậy, hoặc các chỉ số giấc ngủ khác.
  - Có thể có các chỉ số khác như SPO2, nhịp tim, và các thông số khác liên quan đến sức khỏe.
  - Có khả năng, các lệnh, tham số và logic trong debug.js chưa chính xác vì tôi đang test, nếu bạn nghi ngờ, hãy thực hiện đọc toàn bộ source code của app để tìm hiểu thêm về cách thức hoạt động của nó.

## 4. Hỗ Trợ

- **Mục tiêu chính:** Mọi phân tích và hướng dẫn sẽ tập trung vào việc giúp bạn giải mã và xử lý dữ liệu giấc ngủ từ nhẫn thông minh, cùng với việc triển khai các tính năng đo SPO2, nhịp tim.
- **Câu hỏi hỗ trợ:** Nếu có bất kỳ câu hỏi nào liên quan đến việc triển khai tính năng hoặc giải mã dữ liệu, hãy đặt câu hỏi rõ ràng và chi tiết để tôi có thể hỗ trợ tốt nhất.
- **Tài liệu tham khảo:** Nếu có cần bổ sung thêm file nào (.java) hoặc thông tin bổ sung nào cần thiết, hãy cung cấp tên file, tôi có thể hỗ trợ cho bạn

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/auroraphtgrp01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
