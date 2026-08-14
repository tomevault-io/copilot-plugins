## bida-api

> > **Thái Lai Billiards** · Cập nhật: tháng 6/2026

# Bida Manager — Tài liệu Website

> **Thái Lai Billiards** · Cập nhật: tháng 6/2026

---

## Tổng quan hệ thống

```
staff.html        ──── Google Sheets API (hiện tại)
owner_desktop.html ──── Go REST API + WebSocket (mới)
                            ↓
                       PostgreSQL @ 162.4.176.129
```

Hệ thống gồm 2 file HTML độc lập, không phụ thuộc lẫn nhau, dùng chung cùng 1 cơ sở dữ liệu.

---

## 1. Staff Web (`staff.html`)

**Dành cho:** Nhân viên vận hành hàng ngày  
**Kích thước:** ~130KB, 2,306 dòng  
**Database:** Google Sheets (JSONP + no-cors fetch)  
**Realtime:** Polling 5 giây  
**URL Sheets:** `https://script.google.com/.../exec`

### 1.1 Navigation (7 tab)

| Icon | Tab | Trang | Mô tả |
|------|-----|-------|-------|
| 🎯 | Tab khách | `pageTabs` | Quản lý tab đang mở |
| 🎱 | Bida | `pageBida` | Điều khiển bàn bida |
| 📦 | Nhập hàng | `pageInventoryStaff` | Kho tồn + nhập hàng |
| 📜 | Lịch sử | `pageHistoryStaff` | Xem lịch sử (chỉ đọc) |
| 📋 | Tổng kết | `pageDailySummary` | Tổng kết ngày |
| 💳 | Nợ | `pageDebts` | Danh sách công nợ |
| 📝 | Điều chỉnh | `pageStaffAdjust` | Thêm điều chỉnh doanh thu |

### 1.2 Tính năng chi tiết

#### 🎯 Tab khách
- **Tạo tab mới** — nhập tên khách → tạo tab với `id = 't' + Date.now()`
- **⚡ Order nhanh** — chọn đồ → thanh toán ngay không cần tab
  - Thanh toán tiền mặt / QR VietQR
  - Ghi nợ với thông tin khách
- **Tab card** hiển thị: tên khách, ngày giờ, tiền bida, danh sách đồ đã order
- **Bấm vào tab** → mở modal Order (thêm/trả đồ) hoặc Thanh toán

#### 🎱 Bida

**Vòng đời bàn bida:**
```
Trống → [Bắt đầu] → Đang chạy → [Dừng] → Tạm dừng
                                              ↓ [Tiếp]
                         [Order]          Đang chạy
                            ↓
                    Cart trong bidaState
                            ↓
               [Kết thúc] → Tạo tab khách
               [Thanh toán] → Thanh toán trực tiếp
               [Gắn tab] → Gộp vào tab có sẵn
```

- `bidaState` lưu trong Google Sheets (sheet `bida_state`), persist qua reload
- **Thời gian** tính từ `startTime` (ISO string), không bị reset khi dừng/tiếp
- **Đa thiết bị:** `closedAt` field phát hiện bàn bị đóng từ thiết bị khác
- **Giá bàn** chuyển theo giờ cắt (`CUTOFF_HOUR:CUTOFF_MIN`)

| Loại bàn | ID | Trước giờ cắt | Sau giờ cắt |
|---|---|---|---|
| Bàn phân 1 | b1 | 25.000đ/h | 30.000đ/h |
| Bàn phân 2 | b2 | 25.000đ/h | 30.000đ/h |
| Bàn 3C | b3 | 40.000đ/h | 45.000đ/h |

**Order từ bida:**
- Cart lưu trong `bidaState[bid].cart` (không tạo tab tạm)
- Hỗ trợ trả/đổi món qua nút **↩ Trả/Đổi hàng**
- Khi kết thúc → gộp cart vào tab

#### 📦 Nhập hàng
- Xem tồn kho theo danh mục
- Nhập hàng: chọn mặt hàng → số lượng → tổng tiền → ghi chú nguồn hàng
- Lịch sử nhập: hiện số lượng + tiền nhập

#### 📜 Lịch sử (chỉ đọc)
- Xem sessions hôm nay và gần đây (50 bản ghi)
- Mỗi session hiển thị: tên tab, giờ, nhân viên, 🎱 tiền bida, 🛒 đồ đã order, tổng tiền
- **Không có nút sửa/xóa** (khác với admin)

#### 📋 Tổng kết ngày
- **Thu:**
  - 🎱 Tiền bida: tổng `bidaCost` các sessions hôm nay
  - 🥤 Tiền vật phẩm: tổng `cartCost` các sessions hôm nay
  - Điều chỉnh (+): tổng adjustments dương
  - Tổng thu
- **Chi:**
  - 📦 Nhập hàng: tổng `cost` các lần nhập hôm nay
  - Các điều chỉnh âm (có tên cụ thể)
  - Tổng chi
- **Doanh thu ngày** = Tổng thu − Tổng chi
- **Bảng vật phẩm:** cột Bán / Nhập / ±

#### 💳 Nợ
- Danh sách khách đang nợ
- Trạng thái: `pending` / `partial` / `paid`
- Bấm **Trả nợ** → nhập số tiền trả → cập nhật remaining
- Tìm kiếm theo tên khách

#### 📝 Điều chỉnh
- **➕ Cộng tiền:** thu thêm (khách trả bù, thu linh tinh)
- **➖ Chi phí vận hành:** mua đá, mua đồ dùng, chi phí nhỏ
- Danh sách điều chỉnh hôm nay và toàn bộ lịch sử

### 1.3 Thanh toán (modal `mo-pay`)

Flow thanh toán tab khách:
```
openPayModal(tabId)
  → renderPayBody()
     - Hiện cart (đồ đã order)
     - Thêm đồ bổ sung (extraCart)
     - Trả lại đồ (returnCart) → trừ khỏi cart trước khi tính
     - Tiền bida (nếu có)
     - Grand total = roundUp1k(bidaCost + cartCost)
  → [Thanh toán] → finalPay(grand)
  → [QR] → hiện VietQR → xác nhận
  → [Cho nợ] → openDebtFromPay(grand) → form ghi nợ
```

**Làm tròn:** `roundUp1k(n)` = làm tròn lên đến nghìn gần nhất

### 1.4 Data Layer (Google Sheets)

**Cấu trúc save:**
- `saveAll()` — debounce 2s, gửi toàn bộ data
- `saveNow(bid)` — debounce 300ms, chỉ gửi `bidaState` của 1 bàn
- `saveAllNow(bid)` — debounce 300ms, gửi toàn bộ data + bidaState của bàn đó

**Sync:**
- JSONP GET mỗi 5 giây (tránh CORS khi mở `file://`)
- So sánh `_hash` để bỏ qua nếu không có thay đổi
- `applyData()` normalize date/month từ Sheets → đúng format

**Sheets:**

| Sheet | Dữ liệu |
|-------|---------|
| `users` | Tài khoản người dùng |
| `tabs` | Tab khách đang mở |
| `sessions` | Lịch sử thanh toán |
| `expenses` | Chi phí |
| `adjustments` | Điều chỉnh doanh thu |
| `invImports` | Lịch sử nhập hàng |
| `inventory` | Kho hàng |
| `debts` | Công nợ |
| `settings` | Cài đặt (giá, giờ cắt) |
| `bida_state` | Trạng thái bàn bida |

### 1.5 Modals

| Modal ID | Chức năng |
|----------|-----------|
| `mo-order` | Order thêm / trả đồ |
| `mo-pay` | Thanh toán |
| `mo-qr` | Hiển thị QR VietQR |
| `mo-newtab` | Tạo tab mới / Order nhanh |
| `mo-debt` | Ghi nợ |
| `mo-inv` | Nhập hàng |
| `mo-adj` | Điều chỉnh / Kết thúc bida |
| `mo-prod` | Quản lý sản phẩm |
| `mo-prod-edit` | Thêm/sửa sản phẩm |

### 1.6 Cấu hình

```javascript
// Bàn bida
const BIDA_TABLES = [
  {id:'b1', name:'Bàn phân 1', type:'phan'},
  {id:'b2', name:'Bàn phân 2', type:'phan'},
  {id:'b3', name:'Bàn 3C',     type:'ba_bang'}
]

// Danh mục vật phẩm
const CATLAB = {
  nuoc:  '🥤 Nước',
  thuoc: '🚬 Thuốc',
  an:    '🍜 Đồ ăn',
  khac:  '📦 Khác'
}

// QR thanh toán
src="./qr.jpg"  // file qr.jpg cùng thư mục
```

---

## 2. Owner Web (`owner_desktop.html`)

**Dành cho:** Chủ quán, quản lý  
**Kích thước:** ~66KB, 1,300 dòng  
**Database:** PostgreSQL qua Go REST API  
**Realtime:** WebSocket (`ws://162.4.176.129/ws`)  
**API:** `http://162.4.176.129/api`  
**Giao diện:** Light theme, desktop-first, sidebar cố định

### 2.1 Navigation (6 tab)

| Icon | Tab | Trang | Mô tả |
|------|-----|-------|-------|
| 📊 | Tổng quan | `pageOwnerDashboard` | Metrics + top items |
| 📅 | Lịch sử | `pageOwnerHistory` | Lọc theo ngày/tháng/năm |
| 💰 | Tài chính | `pageOwnerFinance` | Doanh thu, chi phí, lợi nhuận |
| 💳 | Công nợ | `pageDebts` | Quản lý nợ khách |
| 👥 | Nhân sự | `pageUsers` | Quản lý tài khoản |
| ⚙️ | Cài đặt | `pageOwnerSettings` | Giá bàn, giờ cắt, sản phẩm |

### 2.2 Tính năng chi tiết

#### 📊 Tổng quan (Dashboard)

**Metrics grid (4 cột):**
- Hôm nay / Hôm qua / Tuần này / Tháng này

**Chi tiết từng kỳ** (6 cards):
- Hôm nay — ngày cụ thể
- Hôm qua — so sánh
- Tuần này — từ thứ 2
- Tuần trước — so sánh
- Tháng này — M/YYYY
- Tháng trước — so sánh

Mỗi card hiển thị:
- Tổng doanh thu (net = revenue + adjustments)
- Phân tích: 🎱 Bida / 🥤 Vật phẩm / 📝 Điều chỉnh

**Top vật phẩm:**
- Bán chạy tháng này (top 5 theo số lần bán)
- Bán chạy tháng trước (top 5)

**Nhập hàng tháng này:**
- Tổng chi nhập hàng
- 10 lần nhập gần nhất

#### 📅 Lịch sử

**3 chế độ lọc:**

1. **Lọc theo ngày cụ thể** (điền cả ngày + tháng + năm):
   - Hiện tổng thu/chi của ngày đó
   - Click vào Thu/Chi → mở rộng danh sách chi tiết từng giao dịch

2. **Lọc theo tháng** (điền tháng + năm, bỏ trống ngày):
   - Liệt kê tất cả các ngày trong tháng
   - Mỗi ngày có thể click để xem chi tiết
   - Hiện: số GD, bida, vật phẩm, điều chỉnh, tổng

3. **Lọc theo năm** (chỉ điền năm):
   - Group theo tháng
   - Click tháng để mở chi tiết: bida, vật phẩm, điều chỉnh

Nút ✏️ trên mỗi session → điều chỉnh doanh thu (cộng/trừ + lý do)

#### 💰 Tài chính

**Kỳ hiện tại:** tháng này

- Doanh thu thuần = sessions.total + adjustments
- Chi phí vận hành (có thể thêm mới)
- **Lợi nhuận** = Doanh thu − Chi phí
- Quỹ quản lý (5% lợi nhuận)
- **Chia cổ phần** theo tỷ lệ:

| Cổ đông | Tỷ lệ |
|---------|-------|
| Hùng | 43% |
| Đức | 35% |
| Nhân | 22% |

Thêm/sửa/xóa chi phí vận hành (lương, điện nước, mặt bằng...)

#### 💳 Công nợ
- Xem toàn bộ nợ (pending / partial / paid)
- Ghi nhận trả nợ → tự động tạo adjustment dương
- Badge màu theo trạng thái

#### 👥 Nhân sự
- Danh sách user (admin/staff)
- Thêm/sửa/vô hiệu hóa tài khoản
- Đổi mật khẩu

#### ⚙️ Cài đặt

**Sản phẩm:**
- Xem danh sách theo danh mục
- Nút ⚙️ Quản lý → thêm/sửa/xóa sản phẩm
- Trường: tên, giá bán, đơn vị, danh mục, ngưỡng cảnh báo tồn kho

**Giá bàn:**
- Giờ chuyển giá (chọn giờ + phút)
- Bàn phân: giá trước/sau giờ cắt
- Bàn 3C: giá trước/sau giờ cắt

### 2.3 Authentication

```
POST /api/auth/login → JWT token (7 ngày)
Token lưu localStorage: 'bida_token'
User lưu localStorage: 'bida_me'
Auto-login nếu token còn hạn
Chỉ role='admin' mới vào được
```

### 2.4 API Integration

**Base URL:** `http://162.4.176.129/api`

| Method | Endpoint | Chức năng |
|--------|----------|-----------|
| GET | `/inventory` | Kho hàng |
| POST/PUT/DELETE | `/inventory/:id` | CRUD sản phẩm |
| GET | `/sessions?limit=200` | Lịch sử thanh toán |
| GET | `/adjustments` | Điều chỉnh |
| POST | `/adjustments` | Thêm điều chỉnh |
| DELETE | `/adjustments/:id` | Xóa điều chỉnh |
| GET | `/debts` | Danh sách nợ |
| POST | `/debts/:id/pay` | Ghi nhận trả nợ |
| GET | `/expenses` | Chi phí |
| POST/PUT/DELETE | `/expenses/:id` | CRUD chi phí |
| GET | `/settings` | Cài đặt |
| PUT | `/settings` | Cập nhật cài đặt |
| GET | `/users` | Người dùng |
| POST/PUT/DELETE | `/users/:id` | CRUD users |
| GET | `/imports` | Lịch sử nhập hàng |

### 2.5 WebSocket Events

Kết nối: `ws://162.4.176.129/ws?token=<jwt>`

**Nhận events (realtime update):**

| Entity | Khi nào |
|--------|---------|
| `bida` | Bàn bida thay đổi trạng thái |
| `bida_all` | Refresh toàn bộ bida state |
| `tabs`, `tabs_add`, `tabs_remove`, `tab_update` | Tab khách thay đổi |
| `inventory`, `inventory_add/update/remove` | Kho thay đổi |
| `session_add` | Có thanh toán mới |
| `adjustment_add/remove` | Điều chỉnh mới |
| `debt_update` | Nợ cập nhật |
| `expense_add/update/remove` | Chi phí thay đổi |
| `settings` | Cài đặt thay đổi |

Auto-reconnect khi mất kết nối (sau 3 giây)

### 2.6 Design System

- **Font:** Inter (400/500/600) + JetBrains Mono (số tiền)
- **Color scheme:** Light — nền `#f8f8f6`, surface `#fff`, border `#e8e6df`
- **Layout:** Sidebar 220px cố định + topbar 52px sticky
- **Sidebar:** Logo → Nav items → User pill (tên + role + logout)
- **Topbar:** Page title + sync badge (Live/Offline) + refresh button
- **Modals:** Backdrop blur, border-radius 14px, max-height 90vh scroll

---

## 3. So sánh Staff vs Owner

| Tính năng | Staff | Owner |
|-----------|-------|-------|
| Database | Google Sheets | PostgreSQL |
| Realtime | Polling 5s | WebSocket |
| Login | Không (auto admin) | Có (JWT) |
| Bida điều khiển | ✅ | ❌ |
| Tab khách | ✅ | ❌ |
| Thanh toán | ✅ | ❌ |
| Nhập hàng | ✅ | ❌ |
| Lịch sử | Xem (read-only) | Xem + điều chỉnh |
| Tổng kết ngày | ✅ | ❌ |
| Dashboard | ❌ | ✅ |
| Tài chính/chia lợi nhuận | ❌ | ✅ |
| Quản lý sản phẩm | ❌ | ✅ |
| Quản lý người dùng | ❌ | ✅ |
| Cài đặt giá | ❌ | ✅ |
| Responsive | Mobile + Desktop | Desktop only |

---

## 4. Deploy

### Staff Web
```bash
# Upload lên hosting Nhân Hoà
# Cần cùng thư mục với qr.jpg
# Truy cập: http://danangrider.com/staff.html
```

### Owner Web
```bash
# Upload lên VPS
scp owner_desktop.html root@162.4.176.129:/var/www/html/owner.html

# Truy cập: http://162.4.176.129/owner.html
# Yêu cầu: Go API đang chạy tại port 3000
```

### Go API (backend cho Owner Web)
```bash
cd bida-go
make build-linux          # Build binary cho Linux
make deploy               # Upload + restart PM2 trên VPS
```

---

## 5. Known Issues & Roadmap

### Issues hiện tại (staff.html)
- Google Sheets cold start ~3-5s gây lag khi mở lần đầu
- Race condition khi 2 thiết bị cùng save (đã giảm thiểu bằng upsert per row)
- Tiền bida trong session đôi khi sai nếu startTime bị Sheets convert

### Roadmap
- [ ] Chuyển staff.html thành React Native app
- [ ] Staff web kết nối Go API (bỏ Google Sheets)
- [ ] Push notification khi bàn bida bị đóng từ xa
- [ ] Báo cáo lợi nhuận theo nguồn (bida vs bán hàng)
- [ ] Export báo cáo PDF/Excel

---

*Tài liệu này mô tả trạng thái hiện tại của hệ thống tính đến tháng 6/2026.*

---
> Source: [ThanhDucNguyen/bida-api](https://github.com/ThanhDucNguyen/bida-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
