## nopgiaoan

> Đây là hệ thống web nội bộ dùng để **số hóa quy trình nộp, quản lý và ký duyệt giáo án** cho Lữ đoàn 575. Hệ thống được triển khai trên **mạng LAN nội bộ, không kết nối Internet**, phục vụ cấu trúc đơn vị phân cấp:

# AGENTS.md — Hệ thống Nộp & Quản lý Giáo án Lữ đoàn 575

## 1. Tổng quan dự án

Đây là hệ thống web nội bộ dùng để **số hóa quy trình nộp, quản lý và ký duyệt giáo án** cho Lữ đoàn 575. Hệ thống được triển khai trên **mạng LAN nội bộ, không kết nối Internet**, phục vụ cấu trúc đơn vị phân cấp:

```
Lữ đoàn (LU_DOAN)
 └─ Tiểu đoàn (TIEU_DOAN)
     └─ Đại đội (DAI_DOI)
         └─ Trung đội (TRUNG_DOI)
```

Mục tiêu: thay thế quy trình nộp giáo án thủ công bằng hệ thống điện tử có kiểm soát hạn nộp, phê duyệt bằng chữ ký số, thống kê tiến độ và nhật ký hoạt động.

## 2. Kiến trúc & Tech stack

| Thành phần | Công nghệ |
|---|---|
| Backend | NestJS + TypeORM + SQL Server |
| Frontend | React + Vite + TailwindCSS |
| Auth | JWT (access + refresh token), Passport |
| File xử lý | Multer (upload), AdmZip/Archiver (nén/giải nén ZIP), Mammoth (đọc .docx), pdf-parse + pdf-lib (đọc/quét .pdf) |
| Dịch vụ phụ | `image-service/` — Python (venv riêng, tách biệt khỏi backend Node) |
| Triển khai | Docker (`Dockerfile` ở cả `backend/` và `frontend/`), Nginx (`nginx.conf`) |

Cấu trúc thư mục gốc:
```
/backend         → NestJS API (prefix /api)
/frontend        → React SPA (Vite)
/image-service   → Dịch vụ Python riêng (venv, chưa tích hợp sâu vào flow chính)
```

## 3. Các module chức năng chính (backend)

1. **Auth & phân quyền** (`auth/`) — đăng nhập, refresh token, đổi mật khẩu bắt buộc lần đầu (`forceChangePassword`), guard theo role (`ADMIN`, `MOD`, `USER`) và theo unit type.
2. **Quản lý đơn vị** (`units/`) — cây tổ chức 4 cấp, hoán đổi Đại đội/Trung đội, thêm/xóa Trung đội kèm audit log.
3. **Giao hạn & theo dõi hạn nộp** (`deadlines/`) — cấp trên giao hạn cho cấp dưới theo quyền phân cấp (`canAssignDeadline`), theo dõi trạng thái CHƯA NỘP / ĐANG NỘP / ĐÃ NỘP / NỘP TRỄ / QUÁ HẠN.
4. **Nộp & quản lý giáo án** (`lesson-plans/`) — upload file ZIP, tự động quét chữ ký trong `.docx`/`.pdf` (tìm ảnh + từ khóa "PHÊ DUYỆT"/"KẾT LUẬN"), tải giáo án mẫu (`TEMPLATE`), lịch sử nộp, gỡ nộp trong 24h.
5. **Chữ ký số** (`digital-signatures/`) — admin cấp phát chữ ký (upload ảnh hoặc ký tay qua canvas) cho từng user, dùng để đối chiếu khi phê duyệt.
6. **Dashboard** (`dashboard/`) — thống kê tổng quan, tiến độ nộp theo đơn vị, thống kê định mức theo loại đơn vị (Tiểu đoàn/Đại đội/Trung đội).
7. **Thông báo** (`notification/`) — thông báo hạn nộp mới, đánh dấu đã đọc.
8. **Audit log** (`audit-logs/`) — ghi nhận reset mật khẩu, tạo/xóa user, hoán đổi đơn vị (kèm IP, User-Agent).
9. **Người dùng** (`users/`) — CRUD user, gắn với `unitId`, hồ sơ cá nhân, QR định danh.

## 4. Frontend — cấu trúc chính

- `src/pages/upload/` — các tab: Nộp giáo án, Xem hạn nộp, Giao hạn, Giáo án cấp dưới, Lịch sử nộp, File mẫu (điều hướng qua query param `?tab=`).
- `src/pages/admin/` — quản lý người dùng, nhật ký hoán đổi đơn vị (chỉ ADMIN).
- `src/context/AuthContext.tsx` — quản lý phiên đăng nhập, tự refresh token qua interceptor axios (`axiosClient.ts`).
- `src/components/` — Sidebar, Topbar, thông báo dropdown, cây tổ chức đơn vị (`UnitTree.tsx`, `OrgMenuNode.tsx`).

## 5. Trạng thái dữ liệu quan trọng cần lưu ý khi sửa code

- `LessonPlan` entity có nhiều cột `nullable: true` (`unitId`, `deadlineId`, `groupId`) — hệ thống chấp nhận dữ liệu thiếu liên kết ở một số giai đoạn, cần cẩn trọng khi thêm ràng buộc NOT NULL/FK mới.
- **`training_lessons` (kế hoạch Sprint 3, chưa triển khai):** dự kiến thêm cột `lessonId` (nullable) vào `lesson_plans` để liên kết tự động với ngân hàng bài học, sau khi parser DOCX/XLSX nhận diện được nội dung bài học ổn định. Sprint 2 giữ hai phần độc lập, không ép FK ngay để tránh vướng dữ liệu cũ/không map được.
- `synchronize: false` trong TypeORM — mọi thay đổi schema phải đi qua migration thủ công (`src/migrations/`).

## 6. Quy ước Git / vệ sinh repo (đã xử lý gần đây)

- `.gitignore` của cả `backend/` và `frontend/` đã được dọn lại (loại bỏ conflict marker cũ, bỏ nhầm dòng `/public` từng khiến static asset không được track).
- Lịch sử git từng phình tới **305MB** do lỡ commit `node_modules`, `dist`, `image-service/venv` (Python venv ~211MB), và đặc biệt file `frontend/public/videos/huongdan.mp4` (221MB) — đã dùng `git filter-repo` để loại bỏ hoàn toàn khỏi lịch sử, đưa `.git` về còn **~4.9MB**.
- **Lưu ý:** video hướng dẫn (`huongdan.mp4`) không còn nằm trong repo/lịch sử — cần host ở nơi khác (Drive/CDN/YouTube nội bộ) và trỏ URL ngoài trong `Helps.tsx`, tránh commit lại file media lớn vào git.
- `image-service/venv/` cần được thêm vào `.gitignore` (Python: `venv/`, `__pycache__/`, `*.pyc`) nếu chưa có, để tránh lặp lại sự cố phình repo.

## 7. Lưu ý khi làm việc trên dự án này

- Ngôn ngữ giao tiếp & văn phong: tiếng Việt, có thể pha thuật ngữ kỹ thuật (giáo án, phân quyền, LAN nội bộ, bản vá bảo mật...).
- Đây là hệ thống quân sự nội bộ — ưu tiên tính ổn định, kiểm soát quyền truy cập chặt theo cấp bậc đơn vị, và ghi log đầy đủ các thao tác nhạy cảm (reset mật khẩu, hoán đổi đơn vị, cấp chữ ký số).
- Nhiều đoạn code hiện tại còn code thử nghiệm/comment cũ (ví dụ trong `Units.tsx`, `AdminUsers.tsx`) — cần xem kỹ trước khi refactor để tránh xóa nhầm logic đang dùng.

---
> Source: [danangcitizen/nopgiaoan](https://github.com/danangcitizen/nopgiaoan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
