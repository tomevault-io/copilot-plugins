## meeting-management-frontend

> **Trạng thái:** 🟢 Đang thực hiện

# 🚀 KẾ HOẠCH CHUYỂN ĐỔI GIAO DIỆN (6X6 REVAMP)

**Trạng thái:** 🟢 Đang thực hiện
**Màu chủ đạo:** Emerald (Xanh ngọc)
**Logo:** Số "36" cách điệu
**Tiến độ:** Giai đoạn 1 Hoàn tất. Đang chuyển sang Giai đoạn 2.

---

## ✅ GIAI ĐOẠN 1: NỀN TẢNG & CẤU HÌNH (DONE)
*Đã thiết lập xong móng nhà.*

- [ ] **1.1. Cấu hình Tailwind (`tailwind.config.cjs`)**: thêm màu Emerald, font Inter, bóng đổ soft.
- [ ] **1.2. Reset CSS toàn cục (`src/index.css`)**: xóa CSS cũ, thêm @tailwind, set nền xám `bg-neutral-50`.
- [ ] **1.3. Dọn dẹp (`src/App.css`)**: vô hiệu hóa file cũ.
- [ ] **1.4. Tạo Component Logo (`src/components/Logo.jsx`)**: tạo logo "6x6" Gradient.

---

## 🏗 GIAI ĐOẠN 2: KHUNG SƯỜN & ĐIỀU HƯỚNG (LAYOUTS)
*Mục tiêu: Thay đổi bộ mặt chính (Sidebar, Header) để người dùng thấy khác biệt ngay.*

- [ ] **2.1. Component Sidebar Mới (`src/components/user/Sidebar.jsx`)**
    - [ ] Chuyển sang Floating Sidebar (lơ lửng, bo góc).
    - [ ] Tích hợp `<Logo />` mới.
    - [ ] Thêm hiệu ứng hover/active xịn xò.
- [ ] **2.2. Layout Người dùng (`src/layouts/UserLayout.jsx`)**
    - [ ] Sửa lại bố cục Flexbox để chứa Sidebar mới.
    - [ ] Đảm bảo background hiển thị đúng màu xám `neutral-50`.
- [ ] **2.3. Header/Navbar (`src/components/user/Navbar.jsx` & `UserHeader.jsx`)**
    - [ ] Làm Header trong suốt hoặc tối giản.
    - [ ] Style lại Avatar và menu Dropdown.
- [ ] **2.4. Layout Admin & Sidebar Admin**
    - [ ] Áp dụng style tương tự cho `src/layouts/AdminLayout.jsx` và `src/components/admin/Sidebar.jsx`.

---

## 🧩 GIAI ĐOẠN 3: TÍNH NĂNG CỐT LÕI (CORE FEATURES)
*Mục tiêu: Làm đẹp các form và chức năng quan trọng nhất.*

- [ ] **3.1. Trang Đăng nhập (`src/pages/LoginPage.jsx`)**
    - [ ] Thiết kế lại giao diện login 2 cột (Ảnh + Form).
- [ ] **3.2. Lịch đặt phòng (`src/components/user/RoomSchedule.jsx`)**
    - [ ] **(Quan trọng)** Chuyển `<table>` thành Grid Timeline.
    - [ ] Thêm vạch đỏ chỉ báo giờ hiện tại.
- [ ] **3.3. Modal Đặt phòng (`src/components/user/BookRoomModal.jsx`)**
    - [ ] Style lại Input, Select, DatePicker theo phong cách mới.
- [ ] **3.4. Modal Chi tiết (`src/components/user/MeetingDetailModal.jsx`)**
    - [ ] Chuyển thành dạng Slide-over (trượt từ phải sang).

---

## 📊 GIAI ĐOẠN 4: CÁC TRANG DASHBOARD (PAGES)
*Mục tiêu: Hiển thị dữ liệu đẹp mắt.*

- [ ] **4.1. Dashboard User (`src/pages/user/DashboardPage.jsx`)**
    - [ ] Widget đếm ngược cuộc họp.
    - [ ] Thống kê dạng Card.
- [ ] **4.2. Danh sách cuộc họp (`src/pages/user/MyMeetingsPage.jsx`)**
    - [ ] Chuyển bảng thành dạng Card List hiện đại.
- [ ] **4.3. Quản lý Admin**
    - [ ] Làm đẹp các bảng quản lý Users, Rooms, Devices.

---

## 🧹 GIAI ĐOẠN 5: TINH CHỈNH CUỐI
- [ ] Kiểm tra Responsive Mobile (Hamburger menu).
- [ ] Kiểm tra lại toàn bộ icon.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gk12355a)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gk12355a)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
