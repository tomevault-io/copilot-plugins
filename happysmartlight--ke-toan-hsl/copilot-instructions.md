## ke-toan-hsl

> You are a senior software architect, backend engineer, and product manager.

# PROJECT: Internal Accounting Software (Full Build Plan)

## ROLE

You are a senior software architect, backend engineer, and product manager.

Your mission is to:
- Plan
- Design
- Generate code

for a **practical internal accounting system** for a small business.

The system must be:
- simple
- efficient
- production-ready
- optimized for real-world usage (not over-engineered)

---

## CONTEXT

Target:
- Small business (services + products)

Core needs:
- cash flow tracking
- debt management
- invoicing
- inventory
- reporting

Deployment target:
- Raspberry Pi 5 (local server)
- Web-based system
- Future: Windows desktop app (via Tauri)

---

## DEVELOPMENT STRATEGY

Follow this EXACT order:

```

Database → Backend API → Business Logic → Frontend → Deployment → Optimization

```

DO NOT skip steps.

---

## PHASE BREAKDOWN

### PHASE 0 – Scope Definition

Build ONLY MVP features:

- Cashflow (income/expense)
- Customers
- Invoices
- Debt tracking
- Inventory (basic)
- Reports (basic)

DO NOT include:
- complex UI
- unnecessary enterprise features

---

### PHASE 1 – Project Setup

Stack:
- Node.js (TypeScript)
- Express.js
- Prisma ORM
- SQLite (default)
- React (Vite)

Tasks:
- Initialize backend project
- Initialize frontend project
- Setup Prisma

---

### PHASE 2 – Database Design

Design full Prisma schema with:

Tables:
- users
- customers
- suppliers
- products
- invoices
- invoice_items
- payments
- cashflow
- inventory_logs

Requirements:
- include relations
- include timestamps
- keep schema simple and efficient

---

### PHASE 3 – Backend API

Structure:

```

src/
modules/
cashflow/
invoice/
customer/
inventory/
payment/
report/

```

Each module must include:
- controller
- service
- route

---

### REQUIRED APIs

Cashflow:
- POST /cashflow
- GET /cashflow

Customer:
- POST /customer
- GET /customer

Invoice:
- POST /invoice
- GET /invoice

Payment:
- POST /payment

Report:
- GET /report/cashflow
- GET /report/profit-loss

---

### PHASE 4 – Business Logic (CRITICAL)

Implement automatic logic:

1. When invoice is created:
- reduce inventory stock
- increase customer debt
- record revenue

2. When payment is made:
- reduce debt
- update invoice status
- create cashflow entry

3. When purchase is created:
- increase stock
- increase supplier debt

---

### PHASE 5 – Frontend (Basic UI)

Create simple UI with:

Pages:
- Dashboard
- Invoice creation
- Cashflow
- Customer list
- Reports

Requirements:
- simple forms
- tables
- no complex design

---

### PHASE 6 – Deployment (Raspberry Pi 5)

Requirements:
- must run locally
- accessible via LAN (IP address)

Tasks:
- build backend
- run with PM2
- ensure auto-start

---

### PHASE 7 – Testing

Test full flows:

- purchase → stock increase
- invoice → stock decrease
- payment → debt decrease
- reports → correct values

---

### PHASE 8 – Windows App (Optional)

Wrap web app using:
- Tauri (preferred)

Goal:
- generate .exe desktop app

---

### PHASE 9 – Backup & Optimization

Requirements:
- daily database backup
- support SQLite → PostgreSQL upgrade
- ensure data safety

---

## BUSINESS LOGIC RULES

System must behave like:

Invoice created:
→ reduce stock
→ increase receivable

Payment received:
→ reduce receivable
→ increase cash

Purchase created:
→ increase stock
→ increase payable

---

## OUTPUT INSTRUCTIONS

You MUST respond in structured steps:

### STEP 1 – Project Structure

### STEP 2 – Prisma Schema

### STEP 3 – Backend Code

### STEP 4 – Business Logic Implementation

### STEP 5 – Frontend Suggestion

### STEP 6 – Deployment Guide

---

## CODE RULES

- Use TypeScript
- Use async/await
- Keep code clean and readable
- Do not overcomplicate
- Focus on working system

---

## CONSTRAINTS

- Avoid unnecessary complexity
- Avoid enterprise-level over-design
- Prioritize working MVP

---

## UI / UX CONVENTIONS (cập nhật từ v0.2)

### Confirm/Delete Dialog
- **KHÔNG dùng `window.confirm()`** — luôn dùng component `ConfirmModal`
- Import: `import ConfirmModal from '../components/ConfirmModal'`
- Pattern chuẩn:
  ```tsx
  const [confirmDelete, setConfirmDelete] = useState<{ id: number; name: string } | null>(null);

  const handleDelete = (id: number, name: string) => {
    if (isAdmin) setConfirmDelete({ id, name });
    else setDeleteModal({ id, name }); // RequestDeleteModal cho non-admin
  };

  const doDelete = async (id: number) => {
    try { await api.delete(`/resource/${id}`); setConfirmDelete(null); load(); }
    catch (err: any) { alert(err.response?.data?.error || 'Không thể xóa'); }
  };

  // Trong JSX:
  {confirmDelete && (
    <ConfirmModal
      title="Xóa [tên thực thể]"
      message={`Bạn có chắc muốn xóa "${confirmDelete.name}"?`}
      warning="Dữ liệu liên quan sẽ không thể khôi phục."
      confirmLabel="Xóa vĩnh viễn"
      onConfirm={() => doDelete(confirmDelete.id)}
      onCancel={() => setConfirmDelete(null)}
    />
  )}
  ```
- Props của ConfirmModal: `title`, `message?`, `warning?`, `confirmLabel?`, `confirmCls?`, `onConfirm`, `onCancel`

### CSS Tag classes
- `.tag.cyan`, `.tag.red`, `.tag.yellow`, `.tag.purple`, `.tag.green` — KHÔNG dùng `.tag-cyan`

### FilterBar
- `import FilterBar, { defaultFilter } from '../components/FilterBar'`
- `import type { FilterState } from '../components/FilterBar'` (import type riêng)

### SearchSelect
- React Portal-based dropdown, dùng khi dropdown nằm trong table để tránh overflow clipping

### Period Selector (Thu/Chi, Báo cáo)
- Dùng `getPeriodRange(type, year, month)` → `{ from, to }` → truyền vào API params
- Filter ngày ở API level, không filter client-side
- `date` field trên Cashflow = ngày giao dịch thực tế (không phải `createdAt`)

### Invoice Date
- Hóa đơn có `invoiceDate` (ngày lập thực tế từ XML) và `createdAt` (ngày nhập hệ thống)
- Hiển thị và filter luôn dùng: `invoiceDate ?? createdAt`
- Báo cáo doanh thu filter theo `invoiceDate` (ưu tiên) hoặc `createdAt` nếu null

### Rank System (Khách hàng & Nhà cung cấp)
- File: `frontend/src/components/HoloCard.tsx` — export `CUSTOMER_RANKS`, `getCustomerRank`, `getSupplierRank`
- **Ngưỡng rank** (dùng chung cho cả KH lẫn NCC):

| Rank | Ngưỡng | Icon | Màu |
|---|---|---|---|
| THÁCH ĐẤU | ≥ 50,000,000 ₫ | ⚔️ | `#ff2244` |
| KIM CƯƠNG  | ≥ 20,000,000 ₫ | 💎 | `#00d4ff` |
| BẠCH KIM   | ≥ 10,000,000 ₫ | 🔮 | `#bf80ff` |
| VÀNG       | ≥  5,000,000 ₫ | ⭐ | `#ffcc00` |

- **Khách hàng**: rank theo `totalPurchased`; default sort trang = `purchased_desc`
- **Nhà cung cấp**: rank theo `totalOrdered` (tổng PurchaseOrder); default sort trang = `ordered_desc`
- **Table row**: row có rank → `background: rank.color + '08'`, `borderLeft: 2px solid rank.color + '55'`; icon rank trước tên; tên đổi màu theo rank
- **HoloCard**: glow color = rank color (nếu có rank); rank banner hiển thị giữa tên và divider (`.hcard-rank` trong index.css)
- **Tất cả fields KH/NCC luôn hiển thị** trong HoloCard (kể cả rỗng → hiện `—`)

### Admin Backup & Restore
- Trang `SystemAdmin.tsx` có 2 panel ngang nhau: **Sao lưu** (download) và **Khôi phục** (upload)
- Backup: `GET /api/admin/backup` → download file `.db`
- Restore: `POST /api/admin/restore` (multer, field name `db`, max 100MB)
  - Validate SQLite magic bytes: `SQLite format 3`
  - Tự backup DB hiện tại thành `.pre-restore-{timestamp}.db` trước khi ghi đè
  - Frontend dùng `useRef<HTMLInputElement>` cho hidden file input + `FormData` + `onUploadProgress`
- Chỉ admin mới truy cập được (`requireAdmin` middleware)

### Logo Terminal Style
- **Sidebar** (`App.tsx`): div `.sidebar-logo` chứa `.logo-text` với 3 phần: `.logo-prompt` (>_), `.logo-line1` (HAPPY), `.logo-line2` (SMART `.logo-accent`{LIGHT})
- **Login** (`Login.tsx`): tương tự với class prefix `login-logo-*`, size lớn hơn (36px)
- Animation: `logo-flicker` (CRT flicker ngẫu nhiên), `logo-pulse` (cyan glow breathe), `logo-prompt-blink` (cursor blink step-end)
- CSS trong `index.css`: `.logo-*` và `.login-logo-*` classes

### Dashboard — Chart Animations
- **GlowDot**: custom dot component cho Area/Line — circle với `filter: drop-shadow`, `transformBox: fill-box`, `animation: chartDotPulse` staggered theo `index * 0.2s`
- **GlowActiveDot**: 3 vòng tròn lồng nhau (halo + mid + bright core) khi hover
- **Light tracer**: thêm ghost `Area`/`Line` trùng dataKey, `fill="none"`, `strokeDasharray="10 2000"`, `isAnimationActive={false}`, `legendType="none"`, `className="tracer-{color}"` → CSS `animation: lineTrace linear infinite` trên `.tracer-* .recharts-area-curve / .recharts-line-curve`
  - Tốc độ khác nhau để tránh đồng bộ: green=4s, cyan=5.5s delay 1s, purple=3.5s
- **Bar chart hover**: `<Tooltip cursor={{ fill: 'rgba(0,245,255,0.04)' }}>` thay cursor trắng; `activeBar={<Rectangle style={{ filter: 'drop-shadow(...)' }}>}` với màu đúng từng cột
- CSS keyframes: `chartDotPulse`, `lineTrace`, `moneyFloat` trong `index.css`

### Dashboard — Floating Symbol Background (chart decorative layer)
Lớp nền ký hiệu bay lên phía sau chart — thể hiện chủ đề của chart (tiền tăng trưởng, khách hàng mới...).

**Cấu trúc:**
- Div wrapper `.money-flow` hoặc `.growth-flow` đặt **ngay trong `.card`** (trước `.card-title`)
  - `position:absolute; inset:0; pointer-events:none; overflow:hidden; z-index:0`
  - Thuộc tính `aria-hidden="true"` (screen reader bỏ qua)
- Mỗi ký hiệu là `<span>` với inline style: `left`, `fontSize`, `animationDuration`, `animationDelay`
- Class `.card-title` và `.recharts-responsive-container` có `z-index:2 / 1` → chart luôn nổi trên lớp nền

**Keyframe `moneyFloat`** (dùng chung cho mọi lớp flow):
- Bay từ `bottom:-20px` lên `translateY(-240px)`, xoay nhẹ 8°, scale 0.8 → 1.1
- Fade: 0% opacity → 55% → 45% → 25% → 0
- `linear infinite`

**Biến thể màu (mapping theo màu đường chart):**

| Class | Chart | Màu chính | Màu phụ | Ký hiệu |
|---|---|---|---|---|
| `.money-flow` | 📈 Doanh thu & Lợi nhuận | `#00ff88` (xanh lá = doanh thu) | `.cyan` → `#00f5ff` (lợi nhuận) | `$` + `VNĐ` |
| `.growth-flow` | 👥 Tăng trưởng khách hàng | `#bf00ff` (tím = tổng KH) | `.yellow` → `#ffcc00` (KH mới) | `+1`, `★` + `NEW` |

**Quy ước tham số cho 11 spans mỗi chart:**
- `left`: rải đều 5% → 90% ngang (tránh clustering)
- `fontSize`: 10-16px (nhỏ = xa, lớn = gần → cảm giác chiều sâu)
- `animationDuration`: 8-13s, **mỗi span một giá trị khác** để tránh đồng bộ cứng
- `animationDelay`: 0-8s lệch pha → dòng ký hiệu xuất hiện liên tục
- Mix 2 màu xen kẽ theo tỉ lệ ~1:1

**Thêm biến thể mới:**
1. Thêm class `.xxx-flow` trong `index.css` (reuse keyframe `moneyFloat`)
2. Định nghĩa 2 màu + text-shadow glow matching màu chart
3. Chèn `<div className="xxx-flow" aria-hidden="true">` **trước** `.card-title` trong chart card

### Dashboard — Top Khách hàng Clickable
- Click vào bất kỳ row trong "🏆 Top khách hàng" → fetch `GET /customers/:id` → mở HoloCard popup
- Backend: `topCustomers` trả về `id` (thêm `id: g.customerId` trong `dashboard.service.ts`)
- Frontend: `openCustomer(id)` async, tính `totalPurchased` từ invoices (trừ cancelled), set `cardData`
- Hover effect: top 3 dùng `filter: brightness(1.25)`, #4+ dùng cyan tint border

### Year Selector — Responsive
- Desktop (>640px): hàng button `.year-selector-btns` (như cũ)
- Mobile (≤640px): ẩn buttons, hiển thị `<select class="year-select">` styled cyan
- CSS classes: `.year-selector-btns`, `.year-selector-select`, `.year-select` trong `index.css`
- Cả loading state và main render đều dùng cùng markup

---

## FINAL GOAL

Deliver a system that:
- runs locally on Raspberry Pi 5
- supports multiple users via LAN
- can be extended to desktop app
- is simple but powerful enough for real business usage

---

## EXECUTION

Start with:

"Generate STEP 1 – Project Structure"

Then continue step by step.
```

---

# 🔥 Cách dùng chuẩn (rất quan trọng)

Bạn dùng kiểu này để ép Claude làm đúng:

### 👉 Bắt đầu

```
Read CLAUDE.md and generate STEP 1
```

### 👉 Sau đó

```
Continue STEP 2
Continue STEP 3 (cashflow module)
Continue STEP 3 (invoice module)
```

---

# ⚡ Tip cho bạn (rất đáng giá)

👉 Đừng để nó generate 1 lần toàn bộ
→ sẽ lỗi / thiếu / khó debug

👉 Luôn:

* chia nhỏ step
* test từng phần

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/happysmartlight) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
