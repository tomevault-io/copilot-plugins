## abc

> Website chia sẻ tài liệu học tập cho phép người dùng đăng ký, đăng nhập, tải lên và xem các tài liệu học tập (PDF, DOCX, v.v.). Admin quản lý nội dung và bình luận. Hệ thống sử dụng server-side rendering với layout dùng chung (header/footer).

# CLAUDE.md — Website Tài Liệu Học Tập

## Tổng quan dự án

Website chia sẻ tài liệu học tập cho phép người dùng đăng ký, đăng nhập, tải lên và xem các tài liệu học tập (PDF, DOCX, v.v.). Admin quản lý nội dung và bình luận. Hệ thống sử dụng server-side rendering với layout dùng chung (header/footer).

---

## Stack công nghệ

- **Backend**: Node.js + Express
- **Database**: MySQL (dùng package `mysql2`)
- **Template Engine**: EJS — server-side layout với partial `header` / `footer` dùng chung
- **Frontend**: HTML5, CSS3 thuần, JavaScript thuần (không Bootstrap, không jQuery, không thư viện ngoài)
- **Session**: `express-session`
- **File upload**: `multer`
- **AJAX**: `fetch` API thuần của trình duyệt

> **Quan trọng**: Không sử dụng CSS framework hay JS framework bên ngoài. Viết tay toàn bộ CSS và JS để đảm bảo hệ số tự làm.

---

## Cấu trúc thư mục

```
project/
├── public/
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── main.js
│   └── uploads/              # Tài liệu người dùng upload (multer)
├── views/
│   ├── partials/
│   │   ├── header.ejs
│   │   └── footer.ejs
│   ├── index.ejs             # Trang chủ
│   ├── documents.ejs         # Danh sách tài liệu
│   ├── document.ejs          # Chi tiết 1 tài liệu
│   ├── upload.ejs            # Upload tài liệu
│   ├── about.ejs             # Giới thiệu & liên hệ
│   ├── login.ejs
│   ├── register.ejs
│   └── admin/
│       ├── index.ejs         # Admin: thống kê view
│       ├── documents.ejs     # Admin: quản lý tài liệu
│       └── comments.ejs      # Admin: quản lý bình luận
├── routes/
│   ├── index.js              # Trang chủ
│   ├── documents.js          # Danh sách & chi tiết tài liệu
│   ├── auth.js               # Login, logout, register
│   ├── upload.js             # Upload tài liệu
│   ├── about.js              # Giới thiệu & liên hệ
│   ├── api.js                # AJAX endpoints (comment)
│   └── admin.js              # Tất cả route /admin/*
├── config/
│   └── db.js                 # Kết nối MySQL (mysql2/promise)
├── middleware/
│   └── auth.js               # Kiểm tra session, role admin
├── app.js                    # Entry point Express
├── package.json
└── database.sql
```

---

## Trang & chức năng bắt buộc (≥ 5 trang)

| Trang | Mô tả |
|---|---|
| Trang chủ (`index.php`) | Danh sách tài liệu nổi bật, popup quảng cáo sau 1 phút |
| Danh sách tài liệu (`documents.php`) | Lọc theo môn học, tìm kiếm |
| Chi tiết tài liệu (`document.php?id=X`) | Xem tài liệu, form bình luận & đánh giá |
| Giới thiệu & Liên hệ (`about.php`) | Thông tin trang web + form gửi ý kiến |
| Trang admin chính (`/admin`) | Thống kê view, quản lý tài liệu, xóa bình luận |
| Trang admin sửa tài liệu (`/admin/documents`) | CRUD tài liệu |
| Đăng nhập / Đăng ký | Xác thực người dùng |

---

## Database

### Bảng `users`
```sql
id, username, password (hashed), email, role ENUM('admin','user'), created_at
```

### Bảng `documents`
```sql
id, title, description, subject, file_path, uploaded_by, view_count, created_at, updated_at
```

### Bảng `comments`
```sql
id, document_id, name, email, content, rating (1-5), created_at
```

### Bảng `contacts`
```sql
id, name, email, message, created_at
```

### Bảng `page_views`
```sql
id, page, view_count
```

---

## Yêu cầu kỹ thuật chi tiết

### 1. Lưu trữ database
- Toàn bộ tài liệu, người dùng, bình luận lưu trong MySQL.
- Kết nối qua `config/db.js` dùng `mysql2/promise` với prepared statements (`pool.execute(sql, [params])`).

### 2. Đăng nhập / Đăng xuất / Đăng ký
- Phân biệt 2 role: `admin` và `user`.
- Mật khẩu hash bằng `bcrypt` (`bcryptjs` package).
- Trang đăng ký công khai cho phép tạo tài khoản role `user`.
- `express-session` quản lý trạng thái đăng nhập (`req.session.user`).

### 3. Server-side layout (EJS partials)
- Mọi view đều `<%- include('partials/header') %>` và `<%- include('partials/footer') %>`.
- Header nhận biến `user` từ `res.locals.user` (middleware gán từ session).
- Header chứa: logo, menu chính, thông tin user (nếu đã đăng nhập).

### 4. Header & Menu
- **Logo** bên trái, **menu chính** ở giữa/phải, **thông tin user** (tên + nút logout) bên phải nếu đã đăng nhập.
- Màn hình lớn (> 800px): hiển thị menu đầy đủ dạng text.
- Màn hình nhỏ (≤ 800px): ẩn menu, hiện icon hamburger ☰.
- Menu có **submenu** dropdown khi hover (CSS `position: absolute` + `:hover`).

### 5. Nút cuộn lên đầu trang
- Nút tròn, icon mũi tên lên, cố định `position: fixed` góc phải dưới.
- Mặc định `opacity: 0.5`, hover thành `opacity: 1` (dùng CSS `transition`).
- JS: hiển thị khi `scrollY > 300`, click thì `window.scrollTo({top:0, behavior:'smooth'})`.

### 6. Hiệu ứng CSS
Bắt buộc sử dụng tất cả:
- `box-shadow` (card tài liệu, button)
- `border-radius` (card, avatar, button)
- `gradient` (header background hoặc banner)
- `transition` (hover effects)
- `animation` (loading spinner, fade-in khi trang load)

### 7. Trang chi tiết tài liệu (theo ID)
- URL dạng `/documents/:id` (Express route param).
- Mỗi lần truy cập tăng `view_count` trong bảng `documents`.
- Hiển thị thông tin tài liệu: tiêu đề, môn học, mô tả, link tải xuống.

### 8. Form bình luận & đánh giá
Fields bắt buộc: `name`, `email`, `content`, `rating` (1–5 sao).

**Validation phía client (JS)**:
- Kiểm tra danh sách từ cấm: `["spam", "quảng cáo", "hack", "xxx", "cheat"]` — nếu content chứa từ nào trong danh sách thì chặn submit, hiện thông báo lỗi, không gửi request.

**Gửi bằng AJAX/fetch**:
```js
fetch('/api/comments', {
  method: 'POST',
  body: formData
})
.then(r => r.json())
.then(data => {
  if (data.success) { /* append bình luận mới vào DOM mà không reload trang */ }
})
```

- Sau khi gửi thành công: bình luận mới hiển thị ngay (prepend vào danh sách), không reload trang.

### 9. Popup quảng cáo trang chủ
- Sau **1 phút** (60000ms) kể từ khi mở trang chủ: hiện popup giới thiệu tài liệu nổi bật.
- Popup có nút "Đóng ✕".
- Khi đóng: lưu `localStorage.setItem('popup_closed', '1')` (hoặc cookie).
- Lần sau mở trang: kiểm tra `localStorage.getItem('popup_closed')`, nếu có thì không hiện popup nữa.

### 10. Trang Giới thiệu & Liên hệ
Gồm 3 phần trên cùng một trang (`about.php`):
- **Giới thiệu**: mô tả website, sứ mệnh.
- **Thông tin liên hệ**: địa chỉ, email, số điện thoại.
- **Form gửi ý kiến**: fields `name`, `email`, `message` → lưu vào bảng `contacts`.

### 11. Trang quản trị (chỉ admin)
Middleware `middleware/auth.js` kiểm tra `req.session.user?.role === 'admin'`, nếu không thì `res.redirect('/login')`.

- **Thống kê**: tổng số view của toàn bộ website (sum `view_count` từ bảng `documents`).
- **Quản lý tài liệu**: danh sách, sửa tiêu đề/mô tả/môn học, xóa.
- **Quản lý bình luận**: danh sách tất cả bình luận, xóa bình luận vi phạm.

### 12. Responsive (3 breakpoint)
```css
/* Mobile: ≤ 800px */
@media (max-width: 800px) { ... }

/* Tablet: 801px – 1200px */
@media (min-width: 801px) and (max-width: 1200px) { ... }

/* Desktop: > 1200px */
@media (min-width: 1201px) { ... }
```
- Desktop/Tablet: menu text đầy đủ, layout nhiều cột.
- Mobile: menu thu gọn thành icon hamburger.

---

## Quy tắc bảo mật

- Dùng **`mysql2` prepared statements** (`pool.execute(sql, [params])`) cho mọi query — không string concatenation.
- Hash mật khẩu bằng `bcryptjs` (`bcrypt.hash` / `bcrypt.compare`).
- Validate và sanitize file upload qua `multer`: chỉ cho phép `.pdf`, `.doc`, `.docx`, `.ppt`, `.pptx`; kiểm tra `mimetype` trong `fileFilter`.
- Middleware `isAdmin` bảo vệ toàn bộ route `/admin/*`.
- EJS tự escape HTML theo mặc định (`<%= %>`) — dùng `<%- %>` chỉ khi thực sự cần render HTML tin cậy.

---

## Khởi tạo project

```bash
npm init -y
npm install express ejs mysql2 express-session bcryptjs multer
```

`app.js` cấu hình tối thiểu:
```js
const express = require('express');
const session = require('express-session');
const app = express();

app.set('view engine', 'ejs');
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(express.static('public'));
app.use(session({ secret: 'secret_key', resave: false, saveUninitialized: false }));

// Gán user vào res.locals để mọi view dùng được
app.use((req, res, next) => {
  res.locals.user = req.session.user || null;
  next();
});

app.use('/', require('./routes/index'));
app.use('/documents', require('./routes/documents'));
app.use('/auth', require('./routes/auth'));
app.use('/upload', require('./routes/upload'));
app.use('/about', require('./routes/about'));
app.use('/api', require('./routes/api'));
app.use('/admin', require('./routes/admin'));

app.listen(3000);
```

---

## Hướng dẫn cho Claude khi làm việc với project này

- Không dùng thư viện CSS/JS bên ngoài (Bootstrap, jQuery, v.v.) — viết tay hoàn toàn.
- Mỗi tính năng mới phải tự kiểm tra trên cả 3 kích thước màn hình.
- Khi thêm query database, luôn dùng `pool.execute(sql, [params])` — không nối chuỗi SQL.
- Không tạo file comment/documentation thừa — code tự giải thích qua tên biến rõ ràng.
- Ưu tiên sửa file hiện có thay vì tạo file mới.
- Sau khi sửa CSS/JS, nhắc người dùng hard-refresh trình duyệt (Ctrl+Shift+R).
- Route admin luôn dùng middleware `isAdmin` từ `middleware/auth.js`.

---
> Source: [navihat/abc](https://github.com/navihat/abc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
