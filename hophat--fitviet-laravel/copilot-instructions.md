## fitviet-laravel

> FitViet là ứng dụng quản lý phòng tập GYM được phát triển với Laravel 11+ và Vue 3.

# Cấu trúc dự án FitViet - Quản lý phòng tập GYM

FitViet là ứng dụng quản lý phòng tập GYM được phát triển với Laravel 11+ và Vue 3.

## Backend (Laravel)
- [routes/web.php](mdc:routes/web.php) - Định nghĩa các routes của ứng dụng
- [app/Models](mdc:app/Models) - Các model quản lý dữ liệu
- [app/Http/Controllers](mdc:app/Http/Controllers) - Controllers xử lý logic
- [app/Rules](mdc:app/Rules) - Validation rules
- [app/Policies](mdc:app/Policies) - Policies phân quyền

## Frontend (Vue 3)
- [resources/js/layouts](mdc:resources/js/layouts) - Các layout chung
- [resources/js/pages](mdc:resources/js/pages) - Các trang của ứng dụng
- [resources/js/components](mdc:resources/js/components) - Các components UI
- [resources/js/stores](mdc:resources/js/stores) - State management với Pinia
- [resources/js/router](mdc:resources/js/router) - Cấu hình Vue Router

## Các tính năng chính
1. Xác thực và phân quyền với Laravel Sanctum
2. Dashboard với thống kê và báo cáo
3. Quản lý hội viên và gói tập
4. Quản lý huấn luyện viên
5. Bán hàng/dịch vụ (POS)
6. Quản lý lịch tập và đặt lịch
7. Quản lý cơ sở vật chất

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hophat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
