## warehouse-management

> Tóm tắt project để Claude (hoặc bất kỳ AI agent nào) hiểu nhanh, không cần đọc lại toàn bộ source mỗi lần.

# CLAUDE.md — Warehouse Dashboard (DongYang)

Tóm tắt project để Claude (hoặc bất kỳ AI agent nào) hiểu nhanh, không cần đọc lại toàn bộ source mỗi lần.

## 1. Tech stack
- Vue 3 (Composition API, `<script setup lang="ts">`) + Vite + TypeScript
- Pinia (state management) — `src/stores/`
- PrimeVue 4 (UI components: Card, Button, Dropdown, Calendar, DataTable, Column, Chart, Dialog, SelectButton, Chip, ProgressSpinner...)
- Tailwind CSS v4 (`@import "tailwindcss"` trong `src/style.css`)
- vue-i18n — đa ngôn ngữ `src/locales/{en,vi,ko}.json`
- Chart.js (qua primevue Chart / vue-chartjs)
- Tauri 2.x — desktop app wrapper
- Axios — gọi API (`src/services/`)

## 2. Khái niệm cốt lõi: AreaKey (SMD / MAINLINE)
Doanh nghiệp có 2 khu vực sản xuất: **SMD** (xưởng cũ) và **MAINLINE** (xưởng mới). Toàn bộ
dashboard/report đều có thể lọc theo khu vực.

`AreaKey = "ALL" | "SMD" | "MAINLINE"` — định nghĩa & export tại `src/stores/dashboard.ts`.

### Helper functions (export từ `src/stores/dashboard.ts`) — TÁI SỬ DỤNG, KHÔNG viết lại logic mới
- `normalizeAreaPart(value)` — chuẩn hóa string -> "SMD" | "MAINLINE" | null
- `formatCodeArea(code)` — suy ra area từ tiền tố mã item (SMD-/SMD_ -> SMD; MAINLINE-/MAIN- -> MAINLINE)
- `getItemArea(item)` -> area của 1 item (default "MAINLINE" nếu không xác định được)
- `getOrderArea(order)` -> area của 1 order (ưu tiên account.areaPart > order.areaPart > item area)
- `getStockinArea(stockin)` -> area của 1 phiếu nhập
- `getLineArea(line)` / `getMachineArea(machine)` -> area của line/machine (dựa vào `Line.areaPart`)
- `isAreaMatch(current, area)` -> `current === "ALL" || current === area`
- `getItemsByArea`, `getOrdersByArea`, `getStockinsByArea`, `getLinesByArea`, `getMachinesByArea` — filter theo area dùng `isAreaMatch`
- `getStockStatus(item)` -> "critical" | "low" | "warning" | "normal" | "out-of-stock" | "not-configured" (so sánh stockQty/saveQuantity)

### Pattern chuẩn để thêm filter Area vào 1 view mới
```ts
import { useDashboardStore, type AreaKey } from "@/stores/dashboard";
const dashboardStore = useDashboardStore();
const selectedArea = ref<AreaKey>("ALL");
const areaOptions = [
  { label: t("reports.common.areaAll"), value: "ALL" as AreaKey },
  { label: "SMD", value: "SMD" as AreaKey },
  { label: "MAINLINE", value: "MAINLINE" as AreaKey },
];
// trong computed filter:
if (selectedArea.value !== "ALL") {
  items = items.filter(i => dashboardStore.getItemArea(i) === selectedArea.value);
}
```
i18n keys dùng chung: `reports.common.filterArea`, `reports.common.areaAll` (đã thêm vào en/vi/ko.json).

## 3. Stores chính (`src/stores/`)
- **dashboard.ts** (~900 dòng) — "trung tâm" của toàn bộ thống kê area-aware:
  - Interfaces: `DashboardStats`, `StockStatusCount`, `TopItem`, `LowStockItem`, `RecentOrderItem`, `RecentStockinItem`, `OrderStatusSummary`, `AreaSummary`, `ChartData`
  - Computed: `smdSummary`, `mainlineSummary`, `allSummary`, `currentSummary`, `areaSummaries`, `dashboardStats`, `orderStatusSummary`, `overviewStats`
  - Lists: `criticalStockItems`, `lowStockItemsByStatus`, `warningStockItems`, `notConfiguredItems`, `alertStockItems`, `topOrderedItems`, `topOrderedItemsByArea`, `recentOrderStatus`, `recentStockins`
  - Chart data: `ordersByStatusChartData`, `stockValueChartData`, `stockValueByAreaChartData`, `stockQtyByAreaChartData`, `orderStatusByAreaChartData`, `lineMachineByAreaChartData`, `monthlyMovementChartData`
  - Action: `setSelectedArea(area)`
- **line_machine.ts** — quản lý Line/Machine (CRUD đầy đủ). Getters: `smdLines`, `mainlineLines`, `getMachinesByLineId`. `Line.areaPart` quyết định SMD/MAINLINE của line.
- **itemStore.ts**, **orderStore.ts**, **stockinStore.ts** — state cho items/orders/stockins (đơn giản, set/get list).

## 4. Views chính (`src/views/`)
- **DashboardView.vue** — đã area-aware đầy đủ (area filter buttons ALL/SMD/MAINLINE, KPI grid, area comparison grid, order status grid, charts, lists).
- **ReportsView.vue** — report cha: 3 card điều hướng (stockin/orders/inventory) + bar chart "Total trend / Items trend" với filter ngày + tabs Day/Week/Month. **Đã thêm Area dropdown** (ALL/SMD/MAINLINE) ở header-controls — filter `allStockins`/`allOrders` (Total trend) và `itemsByArea` (Items trend dropdown sản phẩm).
- **reports/InventoryReportView.vue** — báo cáo tồn kho. **Đã thêm Area dropdown** lọc `baseItems` qua `getItemArea`.
- **reports/OrdersReportView.vue** — báo cáo đơn xuất. **Đã thêm Area dropdown** lọc `filteredOrders` qua `getOrderArea`.
- **reports/StockinReportView.vue** — báo cáo phiếu nhập. **Đã thêm Area dropdown** lọc `filteredStockins` qua `getStockinArea`.

## 5. Style / Font conventions (`src/style.css` + `index.html`)
- Font chữ: **Be Vietnam Pro** (body), **Plus Jakarta Sans** (h1-h6, font-weight 700). Cả 2 font load qua Google Fonts trong `index.html` (đã fix bug thiếu Plus Jakarta Sans — trước đó h1-h6 fallback sang sans-serif mặc định).
- CSS variables (`:root`): `--primary-color #6366f1`, `--success-color #10b981`, `--warning-color #f59e0b`, `--danger-color #ef4444`, `--info-color #3b82f6`, `--gray-50..900`.
- Badge classes: `.badge-success/warning/danger/info` (dùng cho status chips).
- Màu theo area (convention trong DashboardView): SMD = blue (#3b82f6 family), MAINLINE = green (#10b981 family).
- Nhiều view dùng inline style với màu cố định (`#666`, `#999`, `#94a3b8`, `#e5e7eb`...) cho text phụ/border — chấp nhận được về contrast (WCAG AA) ở mức hiện tại, nhưng nếu cần đồng bộ hơn nữa có thể thay bằng biến `var(--gray-*)`.

## 6. Stock status logic
`getStockStatus(item)` so sánh `item.stockQty` với `item.saveQuantity`:
- `not-configured`: chưa cấu hình saveQuantity
- `out-of-stock`: stockQty = 0
- `critical` / `low` / `warning` / `normal`: theo tỷ lệ stockQty/saveQuantity (xem dashboard.ts để biết ngưỡng chính xác)

## 7. Khi cần sửa/thêm tính năng liên quan Area
1. Không viết lại logic detect area — luôn dùng helper trong `dashboard.ts`.
2. Thêm `AreaKey` dropdown theo pattern ở mục 2.
3. Nhớ thêm i18n key `reports.common.filterArea` / `areaAll` nếu file locale chưa có (đã có ở en/vi/ko).
4. Item không có `areaPart` → fallback theo prefix mã (`formatCodeArea`) → default `MAINLINE`.

---
> Source: [DThuong/WAREHOUSE_MANAGEMENT](https://github.com/DThuong/WAREHOUSE_MANAGEMENT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
