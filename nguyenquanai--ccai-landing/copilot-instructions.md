## ccai-landing

> Landing page bán ebook "Claude Cowork AI Thực Chiến" của tác giả Nguyễn Quân (gọi "sếp Quân").

# Claude Cowork AI Thực Chiến — Landing Page

## Bối cảnh dự án
Landing page bán ebook "Claude Cowork AI Thực Chiến" của tác giả Nguyễn Quân (gọi "sếp Quân").
Website LIVE tại https://nguyenquanai.vn, GitHub Pages, repo này (nguyenquanai/ccai-landing), branch main.
Domain qua Tenten.vn, hết hạn 26/06/2027.

## Cấu trúc file
- index.html — toàn bộ trang chủ (1 file, inline CSS + JS)
- gioi-thieu.html — trang "Về tôi" giới thiệu tác giả Nguyễn Quân (đã xong, xem mục Đã hoàn thành)
- assets/ — ảnh bìa, favicon
- CNAME — domain config, KHÔNG XÓA/SỬA trừ khi đổi domain

## Sản phẩm & giá (BẤT BIẾN — không tự đổi giá)
Tập 1: 119.000đ | Tập 2: 179.000đ | Tập 3: 219.000đ
Combo 3 tập: 399.000đ (gói chủ lực, tiết kiệm 118k) | Bản đầy đủ: 519.000đ

## Đã hoàn thành (KHÔNG cần làm lại)
- Modal đặt mua 3 bước (form → xác nhận → QR/tặng miễn phí), mở từ mọi nút mua trên trang
- Hệ thống Khuyến mãi tự động qua Google Sheet "Khuyến mãi" (Apps Script v11)
- Hệ thống Tặng miễn phí qua Google Sheet "Tặng miễn phí" (Apps Script v12)
- Nút "Đọc thử" cho 3 tập (mở Heyzine thật, có GA4 event read_sample_click)
- Section "Bên trong từng tập có gì?" — đã gộp mục lục accordion, KHÔNG còn section mục lục riêng
- Logo header 2 dòng (icon Q + "Claude Cowork" / "AI Thực Chiến")
- GA4 đã chạy thật (G-LSDVNKJLVM), UTM tracking cho Facebook/Zalo/Google/TikTok Ads
- OG image đã sửa thành URL tuyệt đối (https://nguyenquanai.vn/assets/og-image.jpg)
- Footer đã điền đủ thông tin liên hệ thật (không còn placeholder)
- Trang `gioi-thieu.html` — trang "Về tôi" giới thiệu tác giả Nguyễn Quân, nội dung lấy cảm hứng từ https://nguyen-van-quan-udolx7d.gamma.site/ (học vấn, kinh nghiệm CNC Vina/APS/BKMECH, định hướng AI Agent & Automation, thế mạnh). Đã đồng bộ màu/font với trang chủ, có scroll reveal effect và timeline card nền cam nổi 3D nhẹ. Đã thêm tab "Giới thiệu" vào nav (desktop + mobile)

## Đang dở / sắp làm
- Facebook Pixel còn để placeholder 'FB_PIXEL_ID', chưa cài ID thật — chỉ cần khi chạy Facebook Ads

## Backend / hệ thống ngoài repo (KHÔNG nằm trong file này, chỉ tham khảo)
- Apps Script v12: https://script.google.com/macros/s/AKfycbwwQm4fftbg0WXXCnrIoG1_qkM_fxKbFu8vsWZa1hnhjrrWeGcqnXZObmH0QEnRHwjT/exec
- Google Sheet CCAI-Leads ID: 1TuUHkSc57z3u0_1enhEpxPdftx5dtYVpqCbKdW11Uoo (tabs: Leads, Content_Calendar, Tặng miễn phí, Khuyến mãi, Heyzine_Config, Reviews, Dashboard, Chi phí & Lợi nhuận, FAQ_Bot)
- SePay webhook + VietQR thanh toán
- Việc liên quan Google Sheets/Drive/GA4/Facebook nên làm ở claude.ai (có kết nối trực tiếp), KHÔNG làm ở Claude Code

## Quy tắc làm việc (BẮT BUỘC tuân thủ)
- Thay đổi lớn/rủi ro (đụng thanh toán, cấu trúc dữ liệu Apps Script): mô tả rõ trước, hỏi sếp Quân xác nhận "ok làm đi" trước khi code
- Thay đổi nhỏ/an toàn (CSS, text, thứ tự hiển thị): làm luôn không cần hỏi
- Trước khi sửa index.html: PULL bản mới nhất từ GitHub (git pull) — không dùng bản cache cục bộ cũ
- Sau khi sửa: kiểm tra HTML cân bằng thẻ, không trùng ID, JS hợp lệ, rồi mới commit + push
- Đổi URL Apps Script → phải nhắc cập nhật CẢ 2 nơi: SHEET_ENDPOINT trong index.html VÀ webhook SePay
- Luôn nói rõ đã đọc/kiểm tra thông tin từ nguồn nào (file local, GitHub, website live...) khi báo cáo cho sếp Quân
- Văn phong: tiếng Việt, gọi sếp Quân là "sếp Quân", chuyên nghiệp, thực chiến, không thổi phồng

## Phong cách thiết kế (bắt buộc giữ nhất quán)
Clean tech, nền kem/trắng ngà, chữ đen than, nhấn cam công nghệ. Font Be Vietnam Pro (nội dung),
EB Garamond + Caladea (riêng logo 2 dòng). Nhiều khoảng trắng, card gọn, responsive mobile tốt.

---
> Source: [nguyenquanai/ccai-landing](https://github.com/nguyenquanai/ccai-landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
