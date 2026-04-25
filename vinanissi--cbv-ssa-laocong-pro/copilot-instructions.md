## cbv-ssa-laocong-pro

> TASK_MAIN PRO production baseline — schema, privacy, SHARED_WITH (v2.2.4+)


# TASK_MAIN — điểm Prod PRO (baseline)

Đây là **mốc production PRO** đã chốt cho bảng **TASK_MAIN** (Google Sheets + AppSheet + GAS). Khi sửa schema, filter, slice hoặc service task, **không hạ cấp** các phần sau trừ khi có quyết định migration có chủ đích.

## Gói tính năng (tham chiếu)

- **Tag Git:** `v2.2.4` trở đi (feat: SHARED_WITH + IS_PRIVATE — service, schema, audit, AppSheet docs).
- **Schema (`90_BOOTSTRAP_SCHEMA.gs` — `CBV_SCHEMA_MANIFEST.TASK_MAIN`):** có **`SHARED_WITH`** (list ID, comma-separated) và **`IS_PRIVATE`** (Yes/No, default logic nghiệp vụ false/blank) đúng thứ tự manifest; đồng bộ **`90_BOOTSTRAP_AUDIT_SCHEMA.gs`** (`optionalColumns`).
- **GAS:** `05_GAS_RUNTIME/45_SHARED_WITH_SERVICE.gs` — thêm cột, `canUserSeeTask`, `shareTaskWith` / `unshareTaskWith`, `setTaskPrivate`, audit helpers.
- **Hiển thị (AppSheet):** Security Filter row-level dùng **`ANY(SELECT(...))`** (không `FIRST` trong ngữ cảnh filter/security), **`CONTAINS([SHARED_WITH], ...)`** cho list — xem `04_APPSHEET/APPSHEET_SECURITY_FILTERS.md`, `APPSHEET_SLICE_MAP.md` (TASK_MY_OPEN, TASK_MY_TASKS), `TASK_MAIN_PRO_SPEC.md` (Visibility & Privacy).

## Logic visibility (tóm tắt)

- **ADMIN:** thấy tất cả.
- **IS_PRIVATE = false/blank:** mọi user (theo filter đã cấu hình).
- **IS_PRIVATE = true:** chỉ OWNER_ID, REPORTER_ID, và user có ID trong SHARED_WITH (+ ADMIN).

Khi thêm cột hoặc đổi tên cột TASK_MAIN, cập nhật **schema manifest**, **audit schema**, **field policy / ref map** tương ứng.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vinanissi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
