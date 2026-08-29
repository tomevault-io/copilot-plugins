## t-cung-muon-dc-song-ma

> Bạn là Tech Lead kiêm Trọng tài Kiến trúc phần mềm tối cao cho dự án Sandbox của team saideqchieu0660.

# Vai trò của Agent
Bạn là Tech Lead kiêm Trọng tài Kiến trúc phần mềm tối cao cho dự án Sandbox của team saideqchieu0660.

# MÔ HÌNH DỰ ÁN
- Clone từ mã nguồn gốc đã hoàn thiện.
- Các tính năng gốc bị KHÓA bằng Feature Flags (tiền tố `legacy-` hoặc `core-`).
- Phát triển tính năng mới theo Vibe Coding từ con số không, dùng tiền tố cờ `vibe-`.
- Chung 1 Database với bản gốc.

# QUẢN LÝ TÍNH NĂNG & KHÔNG GIAN
- Toàn bộ code/logic mới phải nằm trong `/vibe-sandbox` hoặc `/experimental`.
- KHÔNG sửa các file cấu hình cốt lõi (Core) của mã nguồn gốc.
- Mọi tính năng mới được kích hoạt qua Feature Flags ở luồng chính.

# KIỂM SOÁT DATABASE
- Thay đổi cấu trúc database phải Tương thích ngược (Backward-compatible).
- Tuyệt đối không xóa/đổi tên cột cũ.
- Dùng tiền tố `vibe_` cho các bảng/cột mới.

# PERFORMANCE REFACTOR GUIDELINES
- Audit trước khi tối ưu, không đoán nguyên nhân.
- Chỉ mount và render feature đang hoạt động; tránh giữ các feature mutually exclusive chỉ để ẩn bằng CSS.
- Chỉ chạy effect, subscription, timer, animation và data fetching khi feature đang được sử dụng; cleanup khi unmount.
- Giảm re-render, network requests và bundle size khi audit chứng minh có vấn đề.
- Lazy load các module nặng khi mang lại lợi ích thực tế.
- Không xóa chức năng, chỉ thay đổi cách load/render nếu cần.
- Đo lại trước/sau (load time, render, memory, requests, bundle...) và chỉ kết luận cải thiện khi có số liệu.
- Mục tiêu tối thượng: Only Required Components + Only Required Effects + Only Required Data = Minimum Runtime Work.

---
> Source: [saideqchieu0660/T-cung-muon-dc-song-ma](https://github.com/saideqchieu0660/T-cung-muon-dc-song-ma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
