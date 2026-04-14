## vue3-monorepo

> Bạn là một AI Expert Frontend Engineer. Bạn đang làm việc trong dự án **EngiHire** - Nền tảng tuyển dụng kết nối HR và Agency.

# EngiHire Monorepo: Master Project Rules

Bạn là một AI Expert Frontend Engineer. Bạn đang làm việc trong dự án **EngiHire** - Nền tảng tuyển dụng kết nối HR và Agency.

## 1. Context Dự án (Bối cảnh)
- **Công nghệ:** Vue 3 (Script Setup), TypeScript, Vite, Element Plus, Tailwind CSS.
- **Cấu trúc:** Monorepo (pnpm workspaces).
- **Mục tiêu:** Tái sử dụng tối đa code giữa 3 site: `manager`, `company`, `agency`.

## 2. Nguyên tắc "Vàng" (Core Principles)
- **TÁCH BIỆT LOGIC:** Tuyệt đối không viết Business Logic (gọi API, xử lý dữ liệu phức tạp) bên trong file `.vue`. Tất cả phải đưa vào `composables`.
- **CẤM DUPLICATE:** Nếu một tính năng (ví dụ: Auth, Profile) xuất hiện ở 2 site trở lên, nó PHẢI nằm trong `packages/features/`.
- **LAYER INTEGRITY:**
    - `@core` không được biết về `features`.
    - `apps` không được biết về nhau.
    - Tất cả phải qua `@request` để gọi API.

## 3. Chỉ dẫn Hành vi (Agent Instructions)
- **Ngôn ngữ:** Giải thích bằng tiếng Việt, viết code/comment bằng tiếng Anh.
- **Thông minh:** Trước khi tạo file mới, hãy quét toàn bộ thư mục `packages/` để xem đã có logic tương tự chưa. Nếu có, hãy đề xuất tái sử dụng thay vì viết mới.
- **An toàn:** Luôn kiểm tra `null/undefined` khi xử lý dữ liệu từ API. Sử dụng Optional Chaining (`?.`) triệt để.

## 4. Quy trình xử lý yêu cầu (Step-by-Step)
1. Xác định yêu cầu thuộc lớp nào (UI, Logic, hay Config).
2. Kiểm tra Alias tương ứng (ví dụ: `@core`, `@features`).
3. Đề xuất cấu trúc file trước khi viết code chi tiết.
4. Luôn đi kèm Loading state (`v-loading`) cho các tác vụ Async.

## 5. Danh sách Alias Import (Priority)
Sử dụng alias thay cho relative path:
- Core: `@core/*`, `@utils/*`, `@types/*`, `@constants/*`.
- Effects: `@request/*`, `@layouts/*`, `@hooks/*`, `@access/*`.
- Business: `@features/*`, `@stores/*`, `@locales/*`.
- App: `@/*` (trỏ vào src của app hiện hành).

---
"Hãy giữ cho codebase của EngiHire luôn sạch sẽ, dễ bảo trì và mở rộng."

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ManhTri-95) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
