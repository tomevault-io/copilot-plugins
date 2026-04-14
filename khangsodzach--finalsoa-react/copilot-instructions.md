## finalsoa-react

> - **Backend:** FastAPI (Python 3.12+), SQLModel, SQLAlchemy, Pydantic v2.

# SkyHome Project Rules & Coding Standards

## 1. Tech Stack Context
- **Backend:** FastAPI (Python 3.12+), SQLModel, SQLAlchemy, Pydantic v2.
- **Database:** PostgreSQL (psycopg2-binary), sử dụng SQLModel để định nghĩa Models và Schemas.
- **Frontend:** React 18 (Vite + TypeScript), Chakra UI, Axios, React Hook Form, TanStack Query.
- **Deployment:** Vercel (Frontend & Serverless Backend), Render (Alternative Backend).

## 2. Backend Coding Standards (FastAPI)
- **Structure:** - Routes: `backend/app/api/routes/`.
  - Models (DB): `backend/app/models/`.
  - Schemas (Pydantic): `backend/app/schemas/`.
  - Core logic: `backend/app/core/`.
- **Naming:** Sử dụng `snake_case` cho hàm, biến và file Python.
- **Rules:**
  - Luôn sử dụng `async def` cho các endpoint API.
  - Sử dụng `SQLModel` thay vì SQLAlchemy thuần khi định nghĩa bảng để tận dụng type-hinting.
  - Endpoint prefix luôn là `/api/v1`.
  - Luôn bao gồm xử lý lỗi với `HTTPException` và logging.

## 3. Frontend Coding Standards (React + TS)
- **Structure:**
  - Components: `src/components/`.
  - Pages: `src/pages/`.
  - API Services: `src/services/api.ts`.
  - Themes: `src/theme/`.
- **Naming:** - Component & File: `PascalCase` (ví dụ: `AdminDashboard.tsx`).
  - Biến & Hàm: `camelCase`.
- **Rules:**
  - Sử dụng Functional Components và Hooks.
  - Sử dụng Chakra UI cho styling để đảm bảo tính đồng nhất giao diện.
  - Quản lý API calls thông qua file `src/services/api.ts`, không gọi trực tiếp axios trong component.
  - Luôn định nghĩa Interface/Type cho dữ liệu trả về từ API.

## 4. Deployment & GitHub Integration
- Backend có tích hợp `mangum` để chạy dạng Serverless trên Vercel.
- Khi sửa đổi Backend, hãy kiểm tra tính tương thích với file `vercel.json` và `render.yaml`.
- Luôn giữ file `requirements.txt` và `package.json` cập nhật khi thêm thư viện mới.

## 5. Interaction Instructions for AI
- Khi code tính năng mới, hãy kiểm tra file `@backend/app/api/main.py` và `@src/services/api.ts` để đảm bảo route được đăng ký đúng.
- Luôn ưu tiên viết Unit Test (Sử dụng `pytest` cho backend và `playwright` cho frontend).
- Nếu gặp lỗi, hãy đọc nội dung file `@DEPLOYMENT_GUIDE.md` để hiểu quy trình sửa lỗi hệ thống.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KhangSoDzach) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
