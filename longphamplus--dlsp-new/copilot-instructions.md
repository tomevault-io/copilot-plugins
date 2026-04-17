## dlsp-new

> > Landing Page + Booking System + Pilot Info + Multilanguage Support


# 🪂 DỰ ÁN WEBSITE DÙ LƯỢN SAPA
> Landing Page + Booking System + Pilot Info + Multilanguage Support  
> Framework chính: **Nuxt 3 (Vue 3, Composition API)**

---

## 🧭 MỤC TIÊU DỰ ÁN
Tạo website giới thiệu và đặt lịch **dịch vụ dù lượn tại Sapa**, có giao diện hiện đại, đa ngôn ngữ và dễ dàng tương tác với khách hàng qua Zalo / WhatsApp.

---

## ⚙️ CÔNG NGHỆ SỬ DỤNG
- **Nuxt 3** (Composition API, Server Routes)
- **Pinia** (quản lý state cho Booking)
- **Vue I18n** (đa ngôn ngữ: EN / VI / FR / RU)
- **TailwindCSS** hoặc **SCSS Modules** cho UI
- **Nodemailer / EmailJS** (gửi mail xác nhận)
- **Axios / Fetch** cho API nội bộ

---

## 🧩 CẤU TRÚC THƯ MỤC

root/
├─ assets/
│ ├─ images/
│ ├─ css/
│ └─ locales/ (json đa ngôn ngữ)
├─ components/
│ ├─ HeaderFloating.vue
│ ├─ FooterSection.vue
│ ├─ ChatFloatingIcons.vue
│ ├─ LanguageSwitcher.vue
│ ├─ BookingForm.vue
│ └─ PilotCard.vue
├─ data/
│ ├─ pilots.json
│ └─ photos.json
├─ layouts/
│ └─ default.vue
├─ pages/
│ ├─ index.vue
│ ├─ booking.vue
│ ├─ pilots/
│ │ ├─ index.vue
│ │ └─ [id].vue
│ ├─ pre-notice.vue
│ └─ about.vue
├─ server/
│ └─ api/
│ └─ send-booking.post.js
├─ store/
│ └─ booking.js
└─ README.md
---

## 🤖 QUY TẮC CODE CHO CHATBOT (RULESET)

### 1. Quy tắc tổng quát
- Luôn **tuân thủ cấu trúc thư mục trên**.
- Code **theo module**, mỗi phần UI là một component riêng.
- **Không hardcode dữ liệu** (phi công, ảnh, text đa ngôn ngữ → lấy từ `/data/` hoặc `/locales/`).
- Mọi text đều hỗ trợ **đa ngôn ngữ** (dùng i18n key).
- Mọi form đều có **validation cơ bản**.
- Code **tối ưu cho mobile-first**.

---

### 2. Quy tắc UI / UX
- **Header:** là nút nổi, click mở menu (overlay hoặc sidebar).
- **Footer:** chứa thông tin liên hệ + social icons.
- **Hero section:** hình nền dù lượn + tiêu đề “DÙ LƯỢN SAPA”.
- **Chat Floating Icons:** luôn cố định ở góc phải (Zalo, WhatsApp).
- **Màu sắc:** tông **xanh trời – trắng – vàng nhẹ** (gợi cảm giác tự do, du lịch).
- **Font:** Inter / Noto Sans / Open Sans.

---

### 3. Quy tắc logic trang Booking
#### Screen 1: Dịch vụ
- Chọn loại dịch vụ → hiển thị giá, chi tiết, dịch vụ tuỳ chọn.
- Tính tổng tiền tự động theo số lượng & option.
- Áp dụng khuyến mãi theo bảng:
=2: -50k/người
=3: -70k/người
=4: -100k/người
=6: -150k/người

#### Screen 2: Liên hệ
- Chọn ngày, giờ (7h00–18h00, cách 60 phút).
- Nhập email, số điện thoại, yêu cầu đặc biệt.

#### Screen 3: Thông tin cá nhân
- Họ tên, Ngày sinh, Giới tính, CCCD/Passport, Cân nặng, Quốc tịch.

#### Screen 4: Xác nhận
- Hiển thị toàn bộ thông tin nhập.
- Checkbox đồng ý điều khoản (link đến trang chính sách).

#### Screen 5: Hoàn tất
- Hiển thị QR Zalo & WhatsApp.
- Thông báo: “Đặt bay không cần thanh toán trước”.
- Gửi email tới `sapa.paragliding@gmail.com`.

---

### 4. Quy tắc đa ngôn ngữ (i18n)
- File ngôn ngữ: `/assets/locales/{lang}.json`
- Cấu trúc mẫu:
```json
{
  "hero": {
    "title": "DÙ LƯỢN SAPA",
    "subtitle": "Giấc mơ bay giữa đại ngàn Sapa"
  },
  "buttons": {
    "book": "Đặt lịch ngay",
    "chatZalo": "Chat qua Zalo",
    "chatWhatsApp": "Chat qua WhatsApp"
  }
}
### 5. Quy tắc dữ liệu phi công (Pilots)

Nguồn dữ liệu: /data/pilots.json

Cấu trúc:{
  "pilot3": {
    "name": "Tuan Nguyen",
    "nickname": "Second Song",
    "role": "Team Pilot",
    "des": "Gentle and attentive pilot...",
    "img": "Tuan Nguyen_Team Pilot.jpg",
    "fullname": "TUAN NGUYEN",
    "experience": "Completed over 1,000 tandem flights safely...",
    "records": "Over 1,000 safe tandem flights...",
    "personality": "Friendly and gentle..."
  }
}

Trang /pilots hiển thị danh sách.

Trang /pilots/[id] hiển thị chi tiết phi công.
6. Quy tắc Pre-Notice

Trang /pre-notice hiển thị danh sách các lưu ý trước chuyến bay:

Có mặt tại điểm bay trước 30 phút.

Trang phục phù hợp (không giày cao gót).

Mang điện thoại trống tối thiểu 4GB để lưu ảnh.

Hiển thị tọa độ điểm bay (link Google Maps).

7. Quy tắc gửi email

API /api/send-booking gửi nội dung JSON qua Nodemailer.

Mẫu email:Subject: [DÙ LƯỢN SAPA] Đơn đặt bay mới
Body: Thông tin khách, thời gian, số lượng, yêu cầu đặc biệt...
Gửi đến: sapa.paragliding@gmail.com

8. Quy tắc kỹ thuật bổ sung

SEO Ready: <meta> title, description, og:image.

Tự động scroll-top khi chuyển route.

Lazy load ảnh trong gallery.

Mọi số tiền hiển thị đồng thời VNĐ + USD.

Form Booking dùng Pinia store để lưu trạng thái qua các màn hình.

9. Quy tắc đóng góp (cho dev team / chatbot)

Mọi commit phải rõ mục đích:
feat: add booking form screen 2, fix: i18n missing ru

Khi chatbot tạo code mới:

Giữ nguyên cấu trúc thư mục.

Code sạch, có comment mô tả logic chính.

Không tạo thư mục thừa hoặc file demo.

Khi cập nhật text hoặc UI → cập nhật luôn i18n keys tương ứng.

10. Thông tin liên hệ

Email quản trị: sapa.paragliding@gmail.com

***Lưu ý: Không cần tạo các file md hướng dẫn.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LongPhamplus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
