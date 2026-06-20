## facerecog

> **Agent & Developer Handbook** | Version 3.0 | Ngôn ngữ: Tiếng Việt + Code EN

# SKILL.md — FaceID Attendance System
**Agent & Developer Handbook** | Version 3.0 | Ngôn ngữ: Tiếng Việt + Code EN

> **📌 ĐỌC TRƯỚC KHI LÀM BẤT CỨ ĐIỀU GÌ**
> File này là nguồn sự thật duy nhất (single source of truth) của toàn bộ project.
> Agent: đọc hết file này trước khi tạo, sửa, hoặc xóa bất kỳ file nào.
> Developer mới: đọc §1 → §3 → §8 là đủ để bắt đầu làm việc.

---

## ═══ QUICK REFERENCE ═══

> Tìm nhanh — không cần đọc toàn bộ:

| Tôi cần... | Section | Ưu tiên |
| :--- | :--- | :--- |
| Hiểu project làm gì, tech stack | §1 | Đọc đầu tiên |
| Xem toàn bộ file và thư mục | §3 | Đọc đầu tiên |
| Build và chạy lần đầu | §8 | Đọc đầu tiên |
| Thêm field / sửa dữ liệu | §7.1 | Hay dùng nhất |
| Fix lỗi nhận diện mặt | §7.2 | Hay dùng nhất |
| Thêm tab/module UI mới | §7.3 | Hay dùng nhất |
| Viết SQL / query mẫu | §4.4 | Hay dùng nhất |
| Hiểu luồng nghiệp vụ chấm công | §5.3 | Quan trọng |
| Hiểu AI recognition pipeline | §5 | Quan trọng |
| Tra cứu màu sắc, font, GDI+ UI | §6 | Khi làm UI |
| Xem lỗi phổ biến và cách fix | §9 | Khi debug |
| Quy trình migration DB | §7.4 | Khi đổi schema |
| Checklist trước khi deploy | §10 | Trước khi release |
| Agent decision tree | §11 | Agent dùng |
| Lịch sử thay đổi | §12 | Tham khảo |

---

## §1 — PROJECT CONTEXT

### 1.1 Mục tiêu hệ thống

Hệ thống chấm công Windows chuyên nghiệp cho doanh nghiệp vừa và nhỏ. Nhận diện nhân viên qua webcam bằng deep learning (không cần thẻ từ, vân tay), lưu trữ toàn bộ trên SQLite với audit trail đầy đủ.

**Các chức năng cốt lõi**:
- Chấm công tự động qua nhận diện khuôn mặt real-time
- Quản lý nhân viên (thêm, sửa, xóa, gán khuôn mặt)
- Quản lý ca làm, phòng ban
- Xin và duyệt nghỉ phép
- Báo cáo chấm công theo ngày/tháng/nhân viên
- Audit log toàn bộ thao tác

### 1.2 Tech Stack

| Layer | Technology | Package NuGet | Version |
| :--- | :--- | :--- | :--- |
| UI | C# WinForms | — | .NET 4.6.1 |
| AI Recognition | Dlib ResNet-128 | `DlibDotNet` | 19.21.x |
| Computer Vision | OpenCV | `OpenCvSharp4` | 4.x |
| Database | SQLite | `System.Data.SQLite` | 6.x |
| Architecture | N-Tier (UI → Service → Repo) | — | — |

### 1.3 Ràng buộc bất di bất dịch (Non-negotiables)

```
⚠️  Platform:     x64 ONLY — Dlib native không có x86/AnyCPU build
⚠️  .NET:         Framework 4.6.1 — không phải .NET Core hay .NET 5+
⚠️  Model path:   PHẢI qua ModelsDirectoryResolver (xem §5.1 — Unicode bug)
⚠️  SQL:          Raw SQL trong Repository.cs — KHÔNG dùng ORM, Entity Framework
⚠️  UI threading: DB/AI calls KHÔNG được chạy trên UI thread
```

---

## §2 — SYSTEM ARCHITECTURE

### 2.1 Sơ đồ tổng quan

```
╔══════════════════════════════════════════════════════════════════╗
║                     FaceIDApp — WinForms UI                      ║
║  ┌──────────────┬───────────────────┬──────────┬──────────────┐  ║
║  │ UCAttendance │UCEmployeeManagement│ UCReports│UCLeaveRequest│  ║
║  └──────────────┴───────────────────┴──────────┴──────────────┘  ║
╚═══════════╤══════════════════════════════╤═══════════════════════╝
            │ gọi Service                  │ gọi Service
            ▼                              ▼
╔═══════════════════════════╗   ╔══════════════════════════════╗
║  FaceRecognitionService   ║   ║    WebcamCaptureService       ║
║  - MatchFace()            ║   ║    - StartCapture()           ║
║  - RegisterFace()         ║   ║    - GetCurrentFrame()        ║
║  - GetAttendanceStatus()  ║   ║    (OpenCvSharp4)             ║
╚═══════════╤═══════════════╝   ╚══════════════════════════════╝
            │ gọi AI Lib + Repo
     ┌──────┴──────┐
     ▼             ▼
╔══════════════╗  ╔══════════════════════════════════════╗
║ Repository   ║  ║  FaceRecog Lib  (src/FaceRecog/)     ║
║ .cs          ║  ║  - FaceRecognition.cs                ║
║ Raw SQL      ║  ║  - Wrapper cho DlibDotNet P/Invoke   ║
╚══════╤═══════╝  ╚══════════════════════════════════════╝
       │
       ▼
╔══════════════════╗
║  SQLite DB   ║
║  face_attendance ║
╚══════════════════╝
```

### 2.2 Nguyên tắc kiến trúc (Architecture Rules)

```
Rule 1: UI không được gọi Repository trực tiếp.
        UI → Service → Repository (bắt buộc đi qua Service layer)

Rule 2: Service không chứa SQL.
        SQL chỉ được viết trong Repository.cs

Rule 3: Repository không chứa business logic.
        Chỉ CRUD thuần túy — không if/else nghiệp vụ ở đây

Rule 4: DTO là vật trung gian duy nhất giữa các layer.
        Không truyền SqlDataReader, System.Data.SQLiteDataReader ra ngoài Repository

Rule 5: Mọi exception phải được log trước khi bubble up.
        Không bao giờ để exception âm thầm bị nuốt (catch không xử lý)
```

### 2.3 Luồng dữ liệu — Chấm công (Happy Path)

```
[Webcam] → frame (Bitmap)
    │
    ▼
WebcamCaptureService.GetCurrentFrame()
    │
    ▼
FaceRecognitionService.MatchFace(bitmap)
    │
    ├─► FaceRecogLib.LocateFaces(bitmap, upsampling=1)
    │       └─ [Nếu rỗng] retry upsampling=2
    │
    ├─► FaceRecogLib.GetFaceEncodings(bitmap, locations)
    │       └─ Trả về List<double[128]>
    │
    ├─► Repository.GetAllFaceEncodings()
    │       └─ Trả về Dict<employeeId, List<double[128]>>
    │
    ├─► FaceRecogLib.FaceDistance(candidate, known) cho từng encoding
    │       └─ Tìm distance nhỏ nhất, so với tolerance (0.6)
    │
    └─► [Match] Repository.UpsertAttendanceRecord(employeeId, DateTime.Now)
            └─ INSERT hoặc UPDATE check_in/check_out tùy trạng thái ngày
```

### 2.4 Luồng dữ liệu — Đăng ký khuôn mặt

```
UCEmployeeManagement → [Chụp ảnh hoặc Upload]
    │
    ▼
FaceRecognitionService.RegisterFace(employeeId, bitmap)
    │
    ├─► Kiểm tra employee đã có bao nhiêu face (tối đa 5)
    ├─► FaceRecogLib.GetFaceEncodings(bitmap)
    ├─► FaceEncodingCodec.Encode(double[128]) → string
    └─► Repository.InsertFaceData(employeeId, encodingString)
```

---

## §3 — KNOWLEDGE MAP (Bản đồ file đầy đủ)

> ⚠️ Agent rule: Tra bảng này TRƯỚC khi tạo hoặc sửa bất kỳ file nào.
> Không được tạo file ở thư mục sai, không được đặt SQL trong Service layer.

### 3.1 Cây thư mục đầy đủ

```
FaceIDAttendance/                        ← Root solution
│
├── FaceIDApp/                           ← Project chính (WinForms)
│   │
│   ├── Data/                            ← LAYER: Logic + Data
│   │   ├── FaceRecognitionService.cs    ★ Matching, RegisterFace, AttendanceLogic
│   │   ├── Repository.cs               ★ Mọi SQL đều ở đây — duy nhất
│   │   ├── Dtos.cs                      ★ Toàn bộ Data Transfer Objects
│   │   ├── FaceEncodingCodec.cs        ★ Encode/Decode vector 128D ↔ string
│   │   └── DatabaseBootstrapper.cs      Khởi tạo và migrate schema khi startup
│   │
│   ├── UserControls/                    ← LAYER: UI Modules
│   │   ├── UCAttendance.cs             ★ Màn hình chấm công real-time
│   │   ├── UCEmployeeManagement.cs     ★ CRUD nhân viên + đăng ký khuôn mặt
│   │   ├── UCReports.cs                 Báo cáo chấm công
│   │   ├── UCLeaveRequest.cs            Quản lý đơn nghỉ phép
│   │   └── UCSettings.cs               Cài đặt hệ thống (tolerance, camera, v.v.)
│   │
│   ├── Database/
│   │   └── face_attendance_v3.sql      ★★ SINGLE SOURCE OF TRUTH cho DB schema
│   │
│   ├── Utils/
│   │   ├── ModelsDirectoryResolver.cs  ★★ BẮT BUỘC dùng để lấy model path
│   │   ├── AppLogger.cs                 Logging helper
│   │   └── UIHelpers.cs                 GDI+ drawing utils, rounded rect, v.v.
│   │
│   ├── Forms/
│   │   └── MainForm.cs                 ★ Entry point UI, sidebar navigation
│   │
│   ├── Properties/
│   │   ├── App.config                  ★ Connection string, tolerance, settings
│   │   └── Resources.resx               Icons, images
│   │
│   └── FaceIDApp.csproj
│
├── src/
│   └── FaceRecog/                       ← Project: AI Core (Class Library)
│       ├── FaceRecognition.cs          ★ Public API: LocateFaces, GetFaceEncodings, FaceDistance
│       ├── FaceRecognitionModels.cs     Các class model (FaceLocation, v.v.)
│       └── FaceRecog.csproj
│
├── models/                              ← AI Model Weights (KHÔNG commit nếu >100MB)
│   ├── dlib_face_recognition_resnet_model_v1.dat   ✅ BẮT BUỘC
│   └── shape_predictor_5_face_landmarks.dat        ✅ BẮT BUỘC
│
└── FaceIDAttendance.sln
```

`★` = file thường xuyên chỉnh sửa | `★★` = file cực kỳ quan trọng, sửa cẩn thận

### 3.2 Ai chịu trách nhiệm gì?

| File | Chịu trách nhiệm | KHÔNG được làm |
| :--- | :--- | :--- |
| `FaceRecognitionService.cs` | Business logic, orchestration | Viết SQL trực tiếp |
| `Repository.cs` | SQL, đọc/ghi DB | Chứa business logic |
| `Dtos.cs` | Định nghĩa data shape | Chứa methods phức tạp |
| `FaceEncodingCodec.cs` | Encode/Decode 128D vector | Gọi DB |
| `face_attendance_v3.sql` | Định nghĩa toàn bộ schema | — |
| `ModelsDirectoryResolver.cs` | Trả về safe path cho model | Bị bypass |
| `UCAttendance.cs` | Hiển thị UI, gọi Service | Gọi Repository trực tiếp |
| `MainForm.cs` | Navigation, khởi tạo app | Chứa business logic |

---

## §4 — DATABASE & SCHEMA

### 4.1 Entity Relationship Diagram (v3.0)

> **LƯU Ý:** Bản đồ bên dưới là bản thu gọn. File `Database/face_attendance_v3.sql` mới là **Single Source of Truth**. KHÔNG tự ý tạo bảng thiếu tham chiếu.

```
[A] DANH MỤC
  departments  ──< employees >── positions
                       │
              ┌────────┼──────────────────┐
              │        │                  │
           face_data  users    employee_shift_schedule
              │
     face_registration_logs 

[B] CHẤM CÔNG & LỊCH LÀM Việc
  employees ──< attendance_records >── work_shifts
                       │
                attendance_logs >── attendance_devices

  employees ──< leave_requests

[C] HỆ THỐNG
  audit_logs, system_settings, holidays, work_calendars
```

### 4.2 Các bảng trọng yếu (Core Tables)

Xem định nghĩa chính xác và Constraints tại `FaceIDApp/Database/face_attendance_v3.sql`. Các bảng dưới đây là xương sống của mọi tính năng:

**1. `employees`** — Lưu thông tin nhân sự cơ bản
- Các khóa ngoại: `department_id`, `position_id`, `default_shift_id`, `manager_id`.
- Trạng thái nhận diện (`is_face_registered`, `face_registered_at`) được duy trì tự động qua Trigger từ `face_data`.
- Chứa logic nghỉ phép: `annual_leave_days`, `used_leave_days`.

**2. `face_data`** — Nơi giao tiếp với AI (Dlib/OpenCV)
- `encoding` (TEXT): Vector 128-D được phân tách bằng dấu chấm phẩy `;`.
- `image_path` / `thumbnail_path`: Đường dẫn vật lý của file ảnh gốc/nhỏ.
- Hỗ trợ đánh giá chất lượng (QC) bởi HR: `is_verified`, `quality_score`.

**3. `attendance_records`** — Chấm công tổng hợp (1 nhân viên = 1 record / ngày)
- Mọi lần điểm danh trong ngày đều đẩy vào đây (check_in / check_out).
- Lưu kết quả tính toán tự động: `status` (Present/Late/EarlyLeave/Absent/Leave), `late_minutes`, `early_minutes`, `working_minutes`.
- Chặn sửa đổi trái phép bằng flag `is_manual_edit` + `manual_edit_reason`.

**4. `attendance_logs`** — Audit truy vết mọi lần quẹt mặt/thẻ
- Append-only database (KHÔNG XÓA, KHÔNG SỬA).
- Ghi lại thiết bị quét, confidence AI, JSON raw payload từ camera, hình chụp khoảnh khắc đó.

**5. `leave_requests`** — Đơn từ nghỉ phép
- Trạng thái: Pending, Approved, Rejected, Cancelled.
- Có Trigger tự động cộng/trừ lại số phép `used_leave_days` vào bảng `employees`.

**6. `face_registration_logs`** & **`audit_logs`** — Hệ thống Audit chi tiết
- `face_registration_logs`: Mọi thao tác Register/Delete face đều push vào đây lưu log.
- `audit_logs`: Theo dõi hành động Admin/HR thao tác trên form phần mềm (INSERT/UPDATE/DELETE).

### 4.3 Face Encoding Protocol

```
Kiểu dữ liệu gốc:  double[128]           (ResNet output, 128 chiều)
Lưu trong DB:       TEXT                  (field: face_data.encoding)
Format string:      "0.1234;-0.5678;..."  (128 số float, ngăn cách bằng ';')

Quy trình Encode (C# → DB):
    double[] vector = FaceRecogLib.GetFaceEncodings(...)[0];
    string text = FaceEncodingCodec.Encode(vector);
    Repository.InsertFaceData(employeeId, text);

Quy trình Decode (DB → C#):
    string text = reader["encoding"].ToString();
    double[] vector = FaceEncodingCodec.Decode(text);
    // Sẵn sàng để tính FaceDistance

Lỗi thường gặp:
    - Separator sai (dùng ',' thay vì ';') → Decode bị lỗi, âm thầm sai
    - Thiếu số (< 128 phần tử) → FaceDistance cho kết quả sai
    - Null hoặc empty string → NullReferenceException trong Decode
```

### 4.4 SQL Patterns chuẩn (Copy & dùng trong Repository.cs)

**Query cơ bản — LUÔN dùng @param, không string concatenation**
```csharp
using var conn = new System.Data.SQLiteConnection(_connectionString);
conn.Open();
using var cmd = new System.Data.SQLiteCommand(
    "SELECT id, name FROM employee WHERE id = @id AND is_active = TRUE", conn);
cmd.Parameters.AddWithValue("@id", employeeId);
using var reader = cmd.ExecuteReader();
while (reader.Read())
{
    // đọc data
}
```

**Insert và lấy ID vừa tạo**
```csharp
using var cmd = new System.Data.SQLiteCommand(
    @"INSERT INTO face_data (employee_id, encoding)
      VALUES (@empId, @enc)
      RETURNING id", conn);
cmd.Parameters.AddWithValue("@empId", employeeId);
cmd.Parameters.AddWithValue("@enc", encodingString);
int newId = (int)cmd.ExecuteScalar();
```

**Upsert chấm công — 1 record/người/ngày**
```csharp
using var cmd = new System.Data.SQLiteCommand(
    @"INSERT INTO attendance_record (employee_id, work_date, check_in, status)
      VALUES (@empId, @date, @time, 'present')
      ON CONFLICT (employee_id, work_date)
      DO UPDATE SET
          check_out = CASE
              WHEN attendance_record.check_out IS NULL THEN EXCLUDED.check_in
              ELSE attendance_record.check_out
          END", conn);
cmd.Parameters.AddWithValue("@empId", employeeId);
cmd.Parameters.AddWithValue("@date", DateTime.Today);
cmd.Parameters.AddWithValue("@time", DateTime.Now);
```

**Load tất cả encodings cho matching**
```csharp
// Trả về Dictionary<int employeeId, List<double[]>>
var result = new Dictionary<int, List<double[]>>();
using var cmd = new System.Data.SQLiteCommand(
    @"SELECT fd.employee_id, fd.encoding
      FROM face_data fd
      JOIN employee e ON e.id = fd.employee_id
      WHERE e.is_active = TRUE", conn);
using var reader = cmd.ExecuteReader();
while (reader.Read())
{
    int empId = reader.GetInt32(0);
    double[] vec = FaceEncodingCodec.Decode(reader.GetString(1));
    if (!result.ContainsKey(empId))
        result[empId] = new List<double[]>();
    result[empId].Add(vec);
}
return result;
```

**Report chấm công theo tháng**
```csharp
using var cmd = new System.Data.SQLiteCommand(
    @"SELECT e.name, ar.work_date, ar.check_in, ar.check_out,
             ar.status, ar.is_late
      FROM attendance_record ar
      JOIN employee e ON e.id = ar.employee_id
      WHERE EXTRACT(MONTH FROM ar.work_date) = @month
        AND EXTRACT(YEAR  FROM ar.work_date) = @year
        AND (@deptId = 0 OR e.department_id = @deptId)
      ORDER BY ar.work_date, e.name", conn);
cmd.Parameters.AddWithValue("@month",  month);
cmd.Parameters.AddWithValue("@year",   year);
cmd.Parameters.AddWithValue("@deptId", departmentId);  // 0 = tất cả phòng ban
```

**Ghi audit log**
```csharp
using var cmd = new System.Data.SQLiteCommand(
    @"INSERT INTO attendance_log (record_id, action, old_value, new_value, changed_by)
      VALUES (@recId, @action, @old::jsonb, @new::jsonb, @by)", conn);
cmd.Parameters.AddWithValue("@recId",  recordId);
cmd.Parameters.AddWithValue("@action", action);         // "check_in" | "manual_edit"
cmd.Parameters.AddWithValue("@old",    JsonConvert.SerializeObject(oldValue));
cmd.Parameters.AddWithValue("@new",    JsonConvert.SerializeObject(newValue));
cmd.Parameters.AddWithValue("@by",     currentUser);
cmd.ExecuteNonQuery();
```

### 4.5 Quy tắc bắt buộc khi làm việc với DB

```
1. KHÔNG dùng ORM (Entity Framework, Dapper, v.v.)
   → Project dùng raw SQL thuần qua System.Data.SQLite

2. KHÔNG string concatenation trong SQL
   → Luôn dùng Parameters.AddWithValue() để tránh SQL Injection

3. KHÔNG sửa schema trực tiếp trên DB trong production
   → Chỉ sửa face_attendance_v3.sql, để DatabaseBootstrapper apply

4. KHÔNG xóa dữ liệu attendance_log
   → Đây là audit trail pháp lý, xóa là vi phạm compliance

5. LUÔN dùng `using` cho connection, command, reader
   → Tránh connection leak, connection pool exhausted

6. Mỗi thay đổi schema phải có comment ngày tháng
   → -- [2025-06-01] Thêm cột phone vào employee
```

---

## §5 — AI & COMPUTER VISION

### 5.1 ⛔ CRITICAL BUG: Unicode Path

> Đây là bug quan trọng nhất của project. Phải đọc và nhớ.

**Vấn đề**: Dlib native DLL không xử lý được đường dẫn có ký tự non-ASCII (Unicode). Tất cả người dùng Việt Nam có nguy cơ bị ảnh hưởng vì username thường có dấu.

```
Path gây crash:
    C:\Users\Nguyễn Văn A\Desktop\FaceIDApp\models\   ← CRASH
    D:\Dự Án\FaceID\models\                           ← CRASH
    C:\Users\admin\models\                             ← OK
```

**Giải pháp — dùng ModelsDirectoryResolver**:

```csharp
// ✅ LUÔN dùng cách này — không ngoại lệ
string modelDir = ModelsDirectoryResolver.GetSafePath();

// Cơ chế bên trong resolver:
// 1. Lấy AppDomain.CurrentDomain.BaseDirectory + "models"
// 2. Kiểm tra path có chứa ký tự non-ASCII không
// 3. Nếu CÓ → copy toàn bộ *.dat sang %TEMP%\FaceIDApp_models\
// 4. Trả về path an toàn (ASCII only)

// ❌ KHÔNG BAO GIỜ làm thế này
var modelDir = Path.Combine(Application.StartupPath, "models");
var fr = new FaceRecognition(modelDir);  // → Crash ngay nếu path có Unicode
```

### 5.2 Recognition Pipeline (chi tiết kỹ thuật)

```csharp
// Bước 1: Lấy frame từ webcam
Bitmap frame = _webcamService.GetCurrentFrame();
if (frame == null) return null;

// Bước 2: Detect vị trí khuôn mặt trong frame
var locations = _faceRecog.LocateFaces(frame, numberOfTimesToUpsample: 1);

// Bước 3: Nếu không tìm thấy mặt → retry với upsampling cao hơn
if (!locations.Any())
    locations = _faceRecog.LocateFaces(frame, numberOfTimesToUpsample: 2);
// Lưu ý: upsampling=2 detect mặt nhỏ hơn nhưng chậm hơn ~4x

// Bước 4: Lấy encoding vector cho từng khuôn mặt tìm được
var unknownEncodings = _faceRecog.GetFaceEncodings(frame, locations);
// Mỗi encoding = double[128]

// Bước 5: Load toàn bộ known encodings từ DB (cache trong RAM)
var knownEncodings = _cachedEncodings ?? _repository.GetAllFaceEncodings();

// Bước 6: Tính khoảng cách Euclidean và tìm match tốt nhất
int? matchedEmployeeId = null;
double bestDistance = double.MaxValue;

foreach (var unknownEnc in unknownEncodings)
{
    foreach (var (empId, knownEncList) in knownEncodings)
    {
        foreach (var knownEnc in knownEncList)
        {
            double dist = FaceRecognition.FaceDistance(knownEnc, unknownEnc);
            if (dist < bestDistance)
            {
                bestDistance = dist;
                matchedEmployeeId = (dist < _tolerance) ? empId : (int?)null;
            }
        }
    }
}

// Bước 7: Ghi attendance nếu match và qua cooldown
if (matchedEmployeeId.HasValue && !IsInCooldown(matchedEmployeeId.Value))
{
    _repository.UpsertAttendanceRecord(matchedEmployeeId.Value, DateTime.Now);
    SetCooldown(matchedEmployeeId.Value);
}
```

### 5.3 State Machine — Trạng thái chấm công trong ngày

```
                    ┌─────────────────┐
                    │   START OF DAY  │
                    │ (chưa có record │
                    │    hôm nay)     │
                    └────────┬────────┘
                             │ Face detected lần 1
                             ▼
                    ┌─────────────────┐
                    │   CHECKED_IN    │
                    │ check_in = now  │
                    │ check_out = NULL│
                    │ status=present  │
                    └────────┬────────┘
                             │ Face detected lần 2
                             ▼
                    ┌─────────────────┐
                    │  CHECKED_OUT    │
                    │ check_out = now │
                    └────────┬────────┘
                             │ Face detected lần 3+
                             ▼
                    ┌─────────────────┐
                    │   (bỏ qua)      │
                    │ đã có cả in/out │
                    └─────────────────┘

Trạng thái đặc biệt (set thủ công bởi admin):
    'absent'  → Không đến, không có đơn xin phép
    'leave'   → Có đơn nghỉ phép được duyệt
    'holiday' → Ngày lễ/nghỉ chung
    'late'    → check_in > work_shift.start_time + late_after_minutes

Logic tính 'late' (tự động khi check_in):
    is_late = (check_in.TimeOfDay > shift.start_time + TimeSpan.FromMinutes(shift.late_after))
```

### 5.4 Tuning Parameters

| Tham số | Mặc định | Vị trí | Hướng dẫn điều chỉnh |
| :--- | :--- | :--- | :--- |
| `tolerance` | `0.6` | `App.config` key `FaceRecog.Tolerance` | Giảm → chặt hơn, ít false positive. Thử 0.5 nếu nhận sai người |
| `upsample_initial` | `1` | `FaceRecognitionService.cs` | Tăng nếu camera xa, mặt nhỏ trong frame |
| `upsample_retry` | `2` | `FaceRecognitionService.cs` | Chỉ dùng khi initial không tìm thấy gì |
| `frame_interval_ms` | `200` | `UCAttendance.cs` (Timer Interval) | Giảm → nhanh hơn, tốn CPU hơn |
| `cooldown_seconds` | `3` | `FaceRecognitionService.cs` | Khoảng cách tối thiểu giữa 2 lần chấm cùng người |

### 5.5 Model Files

| File | Kích thước | Bắt buộc | Chức năng |
| :--- | :--- | :--- | :--- |
| `dlib_face_recognition_resnet_model_v1.dat` | ~21MB | ✅ Có | Tạo face encoding 128D |
| `shape_predictor_5_face_landmarks.dat` | ~9MB | ✅ Có | Align khuôn mặt nhanh |
| `shape_predictor_68_face_landmarks.dat` | ~95MB | ❌ Không | Landmark chi tiết (precision cao hơn) |

Download: http://dlib.net/files/ hoặc GitHub dlib releases

### 5.6 Performance Guidelines

```
Benchmark tham khảo (Core i5-8250U, 8GB RAM, không GPU):
    LocateFaces upsampling=1:  ~80ms/frame
    LocateFaces upsampling=2:  ~300ms/frame
    GetFaceEncodings:           ~120ms/face
    FaceDistance 1000 vectors:  ~2ms (C# loop thuần)

Tối ưu:
    ✓ Cache knownEncodings trong RAM, chỉ reload khi có INSERT vào face_data
    ✓ Chạy toàn bộ recognition trên background thread (Task.Run)
    ✓ Dùng frame_interval_ms ≥ 200ms để không overload CPU
    ✓ Resize frame về 640×480 trước khi xử lý nếu camera 1080p
    ✓ Không gọi GetAllFaceEncodings() mỗi frame — cache lại
```

---

## §6 — UI DESIGN SYSTEM

### 6.1 Color Tokens (Fluent Slate Theme)

| Token | Hex | Dùng cho |
| :--- | :--- | :--- |
| `SidebarBg` | `#0F172A` | Nền sidebar trái |
| `SidebarText` | `#94A3B8` | Text nav không active |
| `SidebarActiveItem` | `#1E293B` | Background nav item đang active |
| `PrimaryBlue` | `#38BDF8` | Accent, nút chính, highlight, badge |
| `AccentHover` | `#0EA5E9` | Hover state của nút PrimaryBlue |
| `ContentBg` | `#F8FAFC` | Nền vùng content chính |
| `CardBg` | `#FFFFFF` | Nền card/panel |
| `CardBorder` | `#E2E8F0` | Viền card |
| `TextPrimary` | `#1E293B` | Text chính trên nền sáng |
| `TextDim` | `#64748B` | Text phụ, placeholder, label |
| `Success` | `#22C55E` | Chấm công thành công, trạng thái present |
| `Warning` | `#F59E0B` | Đi muộn (is_late = true) |
| `Danger` | `#EF4444` | Lỗi, vắng mặt, unknown face |

```csharp
// Định nghĩa trong UIHelpers.cs hoặc static class AppColors:
public static class AppColors
{
    public static readonly Color SidebarBg         = Color.FromArgb(15, 23, 42);
    public static readonly Color PrimaryBlue        = Color.FromArgb(56, 189, 248);
    public static readonly Color SidebarActiveItem  = Color.FromArgb(30, 41, 59);
    public static readonly Color TextDim            = Color.FromArgb(100, 116, 139);
    public static readonly Color ContentBg          = Color.FromArgb(248, 250, 252);
    public static readonly Color Success            = Color.FromArgb(34, 197, 94);
    public static readonly Color Warning            = Color.FromArgb(245, 158, 11);
    public static readonly Color Danger             = Color.FromArgb(239, 68, 68);
}
```

### 6.2 Typography

| Dùng cho | Font | Size | Style |
| :--- | :--- | :--- | :--- |
| Heading module | Segoe UI | 14pt | Bold |
| Label field | Segoe UI | 10pt | Regular |
| Body / input | Segoe UI | 10pt | Regular |
| Sidebar nav | Segoe UI | 10pt | Regular |
| Badge / status | Segoe UI | 8pt | Bold |
| Timestamp | Segoe UI Mono | 9pt | Regular |

### 6.3 DataGridView Standard (GridStyleHelper)

Toàn bộ hệ thống phải sử dụng form chuẩn cho DataGridView để đảm bảo UI đồng nhất (header tối màu, flat list).

```csharp
// Đặt trong constructor của UserControl, ngay sau InitializeComponent()
GridStyleHelper.ApplyStandard(dgvMyGrid);

// Đối với các cột trạng thái (Status), thêm event CellFormatting để nhận màu:
dgvMyGrid.CellFormatting += GridStyleHelper.StatusCellFormatting("ColumnName");
```

### 6.4 Drawing với GDI+ (Card rounded corner)

```csharp
// ❌ KHÔNG dùng WinForms built-in borders — trông xấu, cứng
panel.BorderStyle = BorderStyle.FixedSingle;

// ✅ Override OnPaint và vẽ bằng GDI+
protected override void OnPaint(PaintEventArgs e)
{
    base.OnPaint(e);
    var g = e.Graphics;
    g.SmoothingMode = SmoothingMode.AntiAlias;

    var rect = new Rectangle(8, 8, Width - 16, Height - 16);
    using var path   = GetRoundedRectPath(rect, radius: 12);
    using var brush  = new SolidBrush(AppColors.CardBg);
    using var pen    = new Pen(AppColors.CardBorder, 1f);

    g.FillPath(brush, path);
    g.DrawPath(pen, path);
}

// Helper ở UIHelpers.cs — dùng chung toàn app
public static GraphicsPath GetRoundedRectPath(Rectangle rect, int radius)
{
    int d = radius * 2;
    var path = new GraphicsPath();
    path.AddArc(rect.X,            rect.Y,             d, d, 180, 90);
    path.AddArc(rect.Right - d,    rect.Y,             d, d, 270, 90);
    path.AddArc(rect.Right - d,    rect.Bottom - d,    d, d, 0,   90);
    path.AddArc(rect.X,            rect.Bottom - d,    d, d, 90,  90);
    path.CloseFigure();
    return path;
}
```

### 6.4 Cấu trúc UI và Navigation

```
MainForm
├── pnlSidebar (fixed 220px, SidebarBg)
│   ├── pnlLogo
│   ├── NavButton "Chấm công"   → UCAttendance
│   ├── NavButton "Nhân viên"   → UCEmployeeManagement
│   ├── NavButton "Báo cáo"     → UCReports
│   ├── NavButton "Nghỉ phép"   → UCLeaveRequest
│   └── NavButton "Cài đặt"     → UCSettings
└── pnlContent (fill, ContentBg)
    └── [UserControl active — DockStyle.Fill]

// Switch UserControl:
private void ShowUserControl(UserControl uc)
{
    pnlContent.Controls.Clear();
    uc.Dock = DockStyle.Fill;
    pnlContent.Controls.Add(uc);
    uc.BringToFront();
}
```

### 6.5 UX Rules bắt buộc

```
1. Loading state: Mọi tác vụ DB/AI ≥ 500ms phải hiện spinner/status
2. Lỗi thân thiện: "Không thể kết nối CSDL. Kiểm tra kết nối mạng."
   KHÔNG hiển thị stack trace hay exception message thô cho end user
3. Confirm trước khi xóa: MessageBox Confirm với tên đối tượng cụ thể
4. Thành công chấm công: flash tên + ảnh nhân viên 3 giây, kèm âm thanh
5. Thất bại nhận diện: hiện "Không nhận ra" với border đỏ, không popup
6. Không dùng MessageBox.Show() cho notification thường xuyên → dùng status bar
```

---

## §7 — AGENT INSTRUCTION MANUAL

### §7.1 — Thêm Field Mới Vào Data Model

**Ví dụ: Thêm `employee.position` (chức vụ) — thực hiện đúng thứ tự 5 bước:**

**Bước 1 — Schema** (`face_attendance_v3.sql`):
```sql
-- [YYYY-MM-DD] Thêm cột position (chức vụ) vào employee
ALTER TABLE employee ADD COLUMN IF NOT EXISTS position VARCHAR(100);
```

**Bước 2 — DTO** (`FaceIDApp/Data/Dtos.cs`):
```csharp
public class EmployeeDto
{
    public int    Id           { get; set; }
    public string Name         { get; set; }
    public string Email        { get; set; }
    public string Phone        { get; set; }
    public int?   DepartmentId { get; set; }
    public int?   ShiftId      { get; set; }
    public bool   IsActive     { get; set; }
    public string Position     { get; set; }  // ← THÊM
}
```

**Bước 3 — Repository** (`FaceIDApp/Data/Repository.cs`):
```csharp
// SELECT: thêm position vào danh sách cột
"SELECT id, name, email, phone, department_id, shift_id, is_active, position FROM employee"

// Mapper: đọc từ reader
Position = reader["position"] == DBNull.Value ? null : reader.GetString(reader.GetOrdinal("position"))

// INSERT: thêm vào VALUES
@"INSERT INTO employee (name, email, phone, department_id, shift_id, position)
  VALUES (@name, @email, @phone, @deptId, @shiftId, @position)"
cmd.Parameters.AddWithValue("@position", (object)dto.Position ?? DBNull.Value);

// UPDATE: thêm vào SET
"UPDATE employee SET name=@name, email=@email, position=@position WHERE id=@id"
cmd.Parameters.AddWithValue("@position", (object)dto.Position ?? DBNull.Value);
```

**Bước 4 — UI** (`UCEmployeeManagement.cs`):
```csharp
// Load: gán vào TextBox
txtPosition.Text = selectedEmployee.Position ?? "";

// Save: đọc từ TextBox
dto.Position = txtPosition.Text.Trim();
if (dto.Position.Length > 100) // validate theo VARCHAR(100)
{
    MessageBox.Show("Chức vụ không được quá 100 ký tự.");
    return;
}
```

**Bước 5 — Apply migration (development)**:
```bash
psql -U postgres -d face_attendance -c \
  "ALTER TABLE employee ADD COLUMN IF NOT EXISTS position VARCHAR(100);"
```

---

### §7.2 — Diagnosis & Fix Lỗi Nhận Diện

**Bảng chẩn đoán:**

| Triệu chứng | Nguyên nhân | Fix |
| :--- | :--- | :--- |
| Luôn "Unknown" | Tolerance quá thấp hoặc encoding corrupt | Debug distance (xem bên dưới), kiểm tra DB |
| Nhận sai người | Tolerance quá cao hoặc ảnh training không đủ | Giảm tolerance → 0.5, thêm ảnh đa góc |
| "Face Not Found" | Upsampling thấp, ánh sáng kém | Tăng upsample_retry, cải thiện ánh sáng |
| App crash khi start | Unicode path bug | Xem §5.1 — kiểm tra ModelsDirectoryResolver |
| Nhận diện chậm | Upsampling=2 mọi frame, frame lớn | Chỉ retry upsampling=2 khi cần, resize frame |
| Chấm công 2 lần liên tiếp | Cooldown quá ngắn | Tăng cooldown_seconds |

**Debug script: Kiểm tra encoding trong DB**
```sql
-- Tìm encoding bị lỗi (không đúng 128 chiều)
SELECT id, employee_id,
       array_length(string_to_array(encoding, ';'), 1) AS num_dims
FROM face_data
WHERE array_length(string_to_array(encoding, ';'), 1) != 128;
-- Kết quả rỗng = tất cả OK
```

**Debug: Log FaceDistance để tìm ngưỡng phù hợp**
```csharp
// Thêm tạm vào FaceRecognitionService.MatchFace() khi debug:
System.Diagnostics.Debug.WriteLine(
    $"[FaceMatch] Emp {empId}: distance={dist:F4} | threshold={_tolerance}");
// Xem Output window trong Visual Studio
// Nếu đúng người mà distance > 0.5 → cần thêm ảnh training đa dạng hơn
```

---

### §7.3 — Thêm UI Module/Tab Mới

**Ví dụ: Thêm "UCStatistics" (Thống kê)**

**Bước 1 — Tạo file UserControl:**
```
FaceIDApp/UserControls/UCStatistics.cs
FaceIDApp/UserControls/UCStatistics.Designer.cs
```

**Template chuẩn (copy và đổi tên class):**
```csharp
public partial class UCStatistics : UserControl
{
    private readonly Repository _repo;

    public UCStatistics()
    {
        InitializeComponent();
        _repo = new Repository(
            ConfigurationManager.ConnectionStrings["FaceAttendanceDB"].ConnectionString);
        this.Load += async (s, e) => await LoadDataAsync();
    }

    private async Task LoadDataAsync()
    {
        try
        {
            SetLoadingState(true);
            var data = await Task.Run(() => _repo.GetStatisticsSummary());
            // UpdateUI phải Invoke nếu gọi từ Task.Run
            this.Invoke(new Action(() => UpdateUI(data)));
        }
        catch (Exception ex)
        {
            AppLogger.Error("UCStatistics.LoadDataAsync", ex);
            this.Invoke(new Action(() =>
                MessageBox.Show("Không thể tải thống kê. Vui lòng thử lại.",
                                "Lỗi", MessageBoxButtons.OK, MessageBoxIcon.Warning)));
        }
        finally
        {
            this.Invoke(new Action(() => SetLoadingState(false)));
        }
    }

    private void SetLoadingState(bool loading)
    {
        pnlLoading.Visible = loading;
        pnlContent.Visible = !loading;
    }

    private void UpdateUI(StatisticsDto data)
    {
        lblTotalEmployees.Text = data.TotalEmployees.ToString();
        lblPresentToday.Text   = data.PresentToday.ToString();
        // v.v.
    }
}
```

**Bước 2 — Đăng ký trong MainForm.cs:**
```csharp
AddNavButton("Thống kê", Resources.icon_chart,
    () => ShowUserControl(new UCStatistics()));
```

**Bước 3 — Thêm DTO và Repository method nếu cần** → theo §7.1.

**Quy tắc bắt buộc cho mỗi UserControl:**
```
✓ Có async LoadDataAsync() gọi từ Load event
✓ Tất cả DB/AI calls trong Task.Run()
✓ Mọi UI update qua this.Invoke()
✓ Try/catch với log và thông báo thân thiện
✓ Loading state (spinner hoặc panel) trong khi chờ
✓ Không gọi Repository trực tiếp — phải qua Service hoặc qua Repository (không qua UI)
```

---

### §7.4 — Database Migration (Schema Change)

```
Quy trình chuẩn cho môi trường đã có data production:

Bước 1: Backup DB
    pg_dump -U postgres face_attendance > backup_YYYYMMDD_HHMM.sql

Bước 2: Sửa face_attendance_v3.sql
    Thêm ALTER TABLE (không DROP TABLE)
    Thêm comment: -- [2025-06-01] Mô tả thay đổi

Bước 3: Cập nhật DatabaseBootstrapper.cs
    Thêm version check và apply logic

Bước 4: Test trên DB copy
    Restore backup → apply migration → test chức năng

Bước 5: Deploy
    Restart app → DatabaseBootstrapper auto-apply khi startup
```

**KHÔNG làm:**
```sql
-- ❌ Xóa và tạo lại table (mất toàn bộ data)
DROP TABLE employee;
CREATE TABLE employee (...);

-- ✅ Thêm column an toàn
ALTER TABLE employee ADD COLUMN IF NOT EXISTS position VARCHAR(100);

-- ✅ Đổi tên column an toàn
ALTER TABLE employee RENAME COLUMN old_name TO new_name;
```

---

### §7.5 — Thêm Setting Mới Vào App.config

```xml
<!-- App.config — thêm key mới với prefix module -->
<appSettings>
  <add key="FaceRecog.Tolerance"        value="0.6" />
  <add key="FaceRecog.UpsampleInitial"  value="1" />
  <add key="FaceRecog.UpsampleRetry"    value="2" />
  <add key="FaceRecog.CooldownSeconds"  value="3" />
  <add key="Camera.DeviceIndex"         value="0" />
  <add key="Camera.FrameIntervalMs"     value="200" />
  <!-- Thêm key mới ở đây, đặt tên theo format: Module.SettingName -->
</appSettings>
```

**Đọc trong code:**
```csharp
// Đọc với fallback default an toàn
double tolerance = double.TryParse(
    ConfigurationManager.AppSettings["FaceRecog.Tolerance"], out double t) ? t : 0.6;

int intervalMs = int.TryParse(
    ConfigurationManager.AppSettings["Camera.FrameIntervalMs"], out int ms) ? ms : 200;
```

---

## §8 — BUILD, SETUP & DEPLOYMENT

### 8.1 Prerequisites

| Yêu cầu | Version | Ghi chú |
| :--- | :--- | :--- |
| Windows | 10/11, 64-bit | x64 bắt buộc |
| Visual Studio | 2019 hoặc 2022 | Community edition OK |
| .NET Framework | 4.6.1 | Có sẵn trong VS install |
| SQLite | 15+ | Cài đặt riêng |
| VC++ Redistributable | 2015-2022 x64 | Dlib native cần |
| Model files | 2 file `.dat` | Download thủ công |

### 8.2 NuGet Packages

| Package | Version khuyến nghị | Dùng cho |
| :--- | :--- | :--- |
| `DlibDotNet` | 19.21.0.20230823 | AI face recognition |
| `OpenCvSharp4` | 4.8.0.20230708 | Webcam capture |
| `OpenCvSharp4.runtime.win` | 4.8.0.20230708 | OpenCV native DLLs |
| `System.Data.SQLite` | 6.0.11 | SQLite connector |
| `Newtonsoft.Json` | 13.0.3 | JSON cho audit log |

### 8.3 Setup Lần Đầu (Step by Step)

```bash
# 1. Clone project
git clone <repo-url>
cd FaceIDAttendance

# 2. Download model files thủ công vào thư mục models/
#    dlib_face_recognition_resnet_model_v1.dat  (~21MB)
#    shape_predictor_5_face_landmarks.dat        (~9MB)
#    Nguồn: http://dlib.net/files/

# 3. Tạo SQLite database
createdb -U postgres face_attendance

# 4. Apply schema
psql -U postgres -d face_attendance -f FaceIDApp/Database/face_attendance_v3.sql

# 5. Cấu hình App.config
#    Sửa Password=YOUR_PASSWORD thành password thực

# 6. Mở Visual Studio
#    → Tools → Options → Projects → Build → Platform target = x64
#    → QUAN TRỌNG: Solution Configuration = Debug, Platform = x64
#    → Build → Build Solution (Ctrl+Shift+B)

# 7. Copy model files vào output
xcopy /Y /I models\*.dat FaceIDApp\bin\x64\Debug\models\

# 8. Run (F5)
```

### 8.4 App.config đầy đủ

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <connectionStrings>
    <add name="FaceAttendanceDB"
         connectionString="Host=localhost;Port=5432;Database=face_attendance;Username=postgres;Password=YOUR_PASSWORD;Pooling=true;MinPoolSize=1;MaxPoolSize=10;"
         providerName="System.Data.SQLite" />
  </connectionStrings>
  <appSettings>
    <add key="FaceRecog.Tolerance"       value="0.6" />
    <add key="FaceRecog.UpsampleInitial" value="1" />
    <add key="FaceRecog.UpsampleRetry"   value="2" />
    <add key="FaceRecog.CooldownSeconds" value="3" />
    <add key="Camera.DeviceIndex"        value="0" />
    <add key="Camera.FrameIntervalMs"    value="200" />
  </appSettings>
  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.6.1"/>
  </startup>
</configuration>
```

### 8.5 Build Release (Deployment)

```bash
# Build Release x64
msbuild FaceIDApp.sln /p:Configuration=Release /p:Platform=x64

# Copy dependencies vào thư mục publish
xcopy /Y /I FaceIDApp\bin\x64\Release\* publish\
xcopy /Y /I models\*.dat publish\models\

# Checklist file publish\:
#   FaceIDApp.exe
#   FaceRecog.dll
#   DlibDotNet.dll + Native dlls
#   OpenCvSharp.dll + OpenCvSharpExtern.dll
#   System.Data.SQLite.dll
#   App.config (đã cấu hình đúng production DB)
#   models\dlib_face_recognition_resnet_model_v1.dat
#   models\shape_predictor_5_face_landmarks.dat
```

### 8.6 Checklist Cài Đặt Trên Máy Client

```
□ Windows 10/11 64-bit
□ Visual C++ Redistributable 2015-2022 x64 đã cài
□ .NET Framework 4.6.1 (có sẵn Win 10+)
□ Thư mục models\ với 2 file .dat
□ App.config trỏ đúng server DB
□ SQLite server accessible từ máy client
□ Webcam kết nối và được Windows nhận dạng
```

---

## §9 — ERROR CATALOG

### 9.1 Lỗi khởi động

| Lỗi / Exception | Nguyên nhân | Fix |
| :--- | :--- | :--- |
| `DllNotFoundException: dlib.dll` | Build với AnyCPU hoặc x86 | Đổi Platform → x64 trong VS |
| `FileNotFoundException: *.dat` | Model file không đúng thư mục | Copy dat files vào `bin\x64\Debug\models\` |
| `System.Data.SQLiteException: connection refused` | SQLite chưa chạy | Kiểm tra service SQLite đang chạy |
| `System.Data.SQLiteException: password authentication failed` | Sai password | Sửa App.config |
| `System.Data.SQLiteException: database does not exist` | Chưa tạo DB | Chạy `createdb face_attendance` |
| Crash ngay khi load model, không exception | Unicode path bug | Xem §5.1, dùng ModelsDirectoryResolver |

### 9.2 Lỗi nhận diện

| Triệu chứng | Debug | Fix |
| :--- | :--- | :--- |
| Luôn trả về Unknown | Log distance — thấy distance > tolerance | Giảm tolerance hoặc thêm ảnh training |
| Nhận sai người | Log distance — thấy sai empId match | Kiểm tra encoding có trùng lắp không |
| Không detect mặt | locations.Count = 0 mọi frame | Tăng upsampling, kiểm tra ánh sáng |
| Encoding corrupt | Query kiểm tra §4.3 | Re-register khuôn mặt cho employee đó |
| Crash khi GetFaceEncodings | Bitmap null hoặc disposed | Null-check frame trước khi gọi |

### 9.3 Lỗi UI

| Triệu chứng | Nguyên nhân | Fix |
| :--- | :--- | :--- |
| UI freeze vài giây | Gọi DB/AI trên UI thread | Dùng `await Task.Run()` |
| `InvalidOperationException: Cross-thread operation` | Update control từ background thread | Dùng `this.Invoke()` |
| DataGridView không refresh | DataSource không được reset | `dgv.DataSource = null; dgv.DataSource = list;` |
| Form mở chậm | Load DB trong constructor | Chuyển sang `Load` event + async |

### 9.4 Lỗi DB

| Exception | Nguyên nhân | Fix |
| :--- | :--- | :--- |
| `unique_violation on (employee_id, work_date)` | INSERT thay vì Upsert | Dùng `ON CONFLICT DO UPDATE` |
| `connection pool exhausted` | Connection không đóng | Đảm bảo mọi connection có `using` |
| `invalid input syntax for type double` | Encoding string sai format | Kiểm tra FaceEncodingCodec.Encode |
| `value too long for type varchar(n)` | Input không validate | Thêm length check trước khi lưu |
| `foreign_key_violation` | Delete employee có attendance | Dùng soft delete: `is_active = FALSE` |

---

## §10 — CHECKLIST TRƯỚC KHI DEPLOY

### 10.1 Checklist Tính năng

```
□ Chấm công nhận đúng ≥ 95% trong ánh sáng bình thường
□ Đăng ký khuôn mặt mới: tối thiểu 3 ảnh từ góc khác nhau
□ CRUD nhân viên: thêm, sửa, ẩn (soft delete), tìm kiếm
□ Báo cáo xuất đúng dữ liệu theo tháng/phòng ban
□ Đơn nghỉ phép: tạo → duyệt/từ chối → phản ánh vào attendance
□ Audit log ghi đầy đủ mọi thao tác chỉnh sửa
□ Không có UI freeze khi chấm công liên tục 5 phút
□ App xử lý đúng khi mất kết nối DB (thông báo, không crash)
```

### 10.2 Checklist Kỹ thuật

```
□ Build Release x64 thành công, không có warning lỗi
□ Platform = x64 trong file .csproj (kiểm tra tag <PlatformTarget>)
□ Connection string đúng DB production
□ Model files đã có trong package deploy
□ App.config tolerance và settings đã review với stakeholder
□ VC++ Redistributable x64 có trong installer hoặc pre-installed
□ Backup DB đã tạo trước khi apply migration
□ Migration đã test trên DB copy
□ AppLogger ghi ra file, không chỉ console
□ Không có hardcoded path có ký tự Unicode
□ Tất cả connection đều trong using statement
```

### 10.3 Checklist Bảo mật

```
□ Connection string KHÔNG commit lên git (.gitignore hoặc user secrets)
□ Tất cả SQL dùng parameterized queries
□ Input validation: length, null, type check trước khi lưu DB
□ Không log face encoding vector ra file log
□ Không log password ở bất cứ đâu
```

---

## §11 — DECISION TREE CHO AGENT

> Agent: Dùng section này để tự quyết định trước khi hành động.

```
NHẬN ĐƯỢC YÊU CẦU
│
├─ "Thêm chức năng X / field Y"
│   ├─ Cần data mới?  YES → §7.1 (thêm field) trước
│   ├─ Cần UI mới?    YES → §7.3 (thêm UC)
│   └─ Cần setting?   YES → §7.5 (thêm App.config key)
│
├─ "Fix lỗi nhận diện / chấm công sai"
│   └─ §7.2 + §9.2 → chạy debug distance script
│
├─ "Fix lỗi crash / exception"
│   └─ §9 → match exception message với bảng Error Catalog
│   └─ Nếu crash lúc startup + không có exception → §5.1 Unicode bug
│
├─ "Sửa giao diện / màu / font / layout"
│   └─ §6 → dùng đúng AppColors tokens, GDI+ pattern
│
├─ "Thay đổi schema / thêm bảng / đổi cột"
│   └─ §7.4 → backup → ALTER TABLE (không DROP) → update Bootstrapper
│
└─ "Build / setup / deploy / cài lên máy mới"
    └─ §8 → theo đúng thứ tự, không bỏ bước copy model files

NGUYÊN TẮC AGENT (không bao giờ vi phạm):
    1. Không tạo file ở thư mục không có trong §3.1
    2. Không viết SQL trong Service layer
    3. Không gọi Repository từ UserControl trực tiếp
    4. Không dùng model path trực tiếp — phải qua ModelsDirectoryResolver
    5. Không update UI từ background thread không có Invoke
    6. Không string concatenation trong SQL (SQL Injection)
    7. Không DROP TABLE khi đang có data
    8. Không bỏ qua audit log khi sửa attendance_record thủ công
    9. Hỏi lại nếu yêu cầu mâu thuẫn với các rule trên
```

---

## §12 — VERSIONING & CHANGELOG

### SKILL.md Changelog

| Version | Ngày | Nội dung |
| :--- | :--- | :--- |
| v1.0 | — | Khởi tạo, nội dung cơ bản |
| v2.0 | 2025-04 | Thêm §9, §10, chuẩn hóa §7, thêm code mẫu |
| v3.0 | 2025-04 | Toàn diện: §2 kiến trúc chi tiết, §4.4 SQL patterns đầy đủ, §5.3 State Machine, §5.6 Performance, §6 Design System hoàn chỉnh, §8 Build step-by-step, §9 Error Catalog 4 nhóm lỗi, §10 Deploy Checklist, §11 Agent Decision Tree |

### Khi nào và cách cập nhật SKILL.md

```
BẮT BUỘC cập nhật khi:
    ✓ Thêm file mới → cập nhật §3.1
    ✓ Thay đổi schema DB → cập nhật §4.1, §4.2
    ✓ Thêm setting mới → cập nhật §7.5 và §8.4
    ✓ Phát hiện bug mới và cách fix → thêm vào §9
    ✓ Thêm UserControl mới → cập nhật §3.1 và §6.4
    ✓ Thay đổi architecture rule → cập nhật §2.2 và §11

Quy trình:
    1. Sửa nội dung trong đúng section
    2. Cập nhật QUICK REFERENCE nếu thêm section mới
    3. Tăng version: patch → x.y+1, thay đổi lớn → x+1.0
    4. Thêm dòng vào bảng §12 với ngày và mô tả ngắn gọn
```

---

*End of SKILL.md v3.0*
*Câu hỏi hoặc thông tin còn thiếu: thêm vào đúng section và cập nhật Changelog.*

---
> Source: [mkhoi2004/FaceRecog](https://github.com/mkhoi2004/FaceRecog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
