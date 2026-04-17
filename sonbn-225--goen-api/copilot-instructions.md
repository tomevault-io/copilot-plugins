## goen-api

> - **Ngôn ngữ:** Go (Golang) phiên bản mới nhất.

# Cẩm nang Phát triển Dự án: Go Web Service - Clean Architecture

## 1. Tổng quan Dự án (Project Context)
- **Ngôn ngữ:** Go (Golang) phiên bản mới nhất.
- **Loại ứng dụng:** RESTful Web Service / API Server.
- **Kiến trúc:** Clean Architecture, tuân thủ nghiêm ngặt nguyên tắc Dependency Inversion. Luồng dữ liệu đi theo 1 chiều: `Handler` -> `Service` -> `Repository`.

## 2. Quy tắc Kiến trúc Cốt lõi (Core Architectural Rules)

1. **Tầng Domain (Trái tim của hệ thống):**
   - **Tuyệt đối không** phụ thuộc vào bất kỳ framework hay thư viện ngoại lai nào (ngoại trừ các thư viện chuẩn của Go).
   - **Entity (`internal/domain/entity/`):** Chứa các struct đại diện cho cấu trúc dữ liệu cốt lõi.
   - **DTO (`internal/domain/dto/`):** Chứa các struct định nghĩa dữ liệu đầu vào (Request) và đầu ra (Response).
   - **Interface (`internal/domain/interfaces/`):** Chứa các định nghĩa "hợp đồng" cho cả `Repository` và `Service`.

2. **Tầng Repository (Data Access):**
   - **Nhiệm vụ:** Thực thi các câu lệnh Database (SQL, NoSQL, Cache).
   - **Quy tắc:** Implement các interface đã định nghĩa ở `domain/interfaces/`. Chỉ nhận và trả về `Entity` hoặc `Error`.

3. **Tầng Service (Business Logic):**
   - **Nhiệm vụ:** Nơi chứa toàn bộ logic tính toán, kiểm tra điều kiện.
   - **Quy tắc:** Quản lý Transaction (`db.WithTx`) và điều phối các Repository.

## 4. Quy tắc Viết Code (Coding Guidelines)

### Tiêu chuẩn Xử lý Transaction: pattern "Flexible Queryer"

Để đảm bảo tính nhất quán dữ liệu (Atomicity) và khả năng tái sử dụng (Composition), tất cả các hàm ghi (Write) trong Repository phải tuân thủ pattern sau:

1. **Signature:** Phương thức phải có hậu tố `Tx` và nhận `tx pgx.Tx` làm tham số thứ hai.
   ```go
   func (r *Repo) SomeMethodTx(ctx context.Context, tx pgx.Tx, ...) error
   ```

2. **Thực thi linh hoạt (Queryer Pattern):**
   Sử dụng interface `database.Queryer` để tự động chọn giữa Transaction hoặc Pool. 
   - Nếu `tx != nil`: Tham gia vào transaction của Service.
   - Nếu `tx == nil`: Tự động lấy connection từ Pool để chạy Standalone.

   ```go
   func (r *SomeRepo) SaveSomethingTx(ctx context.Context, tx pgx.Tx, entity E) error {
       var q database.Queryer = tx
       if tx == nil {
           pool, _ := r.db.Pool(ctx)
           q = pool
       }
       // Thực thi SQL qua q
       _, err := q.Exec(ctx, "INSERT ...", ...)
       return err
   }
   ```

3. **Trách nhiệm của Service:**
   - Service quyết định khi nào cần Transaction (Unit of Work).
   - Sử dụng `s.db.WithTx(ctx, func(tx pgx.Tx) error { ... })` để bao bọc các lời gọi Repo.

## 5. Quy trình làm việc của AI (AI Workflow)
Khi được yêu cầu tạo tính năng mới (ví dụ: Feature X), AI phải tuân thủ nghiêm ngặt thứ tự sau:
Bước 1 (Domain) -> Bước 2 (Repository - Tuân thủ Flexible Queryer) -> Bước 3 (Service - Quản lý Tx) -> Bước 4 (Handler) -> Bước 5 (App/Main).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sonbn-225) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
