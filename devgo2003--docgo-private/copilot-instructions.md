## docgo-private

> Quy tắc chuẩn hóa response format cho tất cả API endpoints

# Chuẩn hóa Response Envelope (RestResponse)

## Cấu trúc Response Chuẩn
Tất cả response nên bọc trong cấu trúc sau để đồng nhất giữa Java và Python:

```json
{
  "apiVersion": "v1",
  "statusCode": 200,
  "shortMessage": "Success | Not Found | Bad Request | Conflict | No Content | Internal Server Error",
  "description": "Mô tả ngắn gọn kết quả",
  "data": <object|null>,
  "timestamp": <ISO-8601>,
  "requestId": <uuid>,
  "path": "/api/v1/..."
}
```

## HTTP Status Codes
- **201 Created**: tạo mới
- **200 OK**: trả dữ liệu thành công (bao gồm cả trường hợp không có dữ liệu với statusCode 204)
- **400 Bad Request**: dữ liệu không hợp lệ (validation)
- **404 Not Found**: không tìm thấy tài nguyên
- **409 Conflict**: xung đột nghiệp vụ
- **500 Internal Server Error**: lỗi không lường trước

## Quy tắc HTTP Status 204
**Lưu ý quan trọng về HTTP Status 204:**
- **KHÔNG BAO GIỜ** trả về HTTP status 204 No Content
- Thay vào đó, luôn trả về HTTP status 200 OK với `statusCode: 204` trong response body
- Điều này đảm bảo tính nhất quán trong API response format và dễ dàng xử lý ở client
- Ví dụ response khi không có dữ liệu:
```json
{
  "apiVersion": "v1",
  "statusCode": 204,
  "shortMessage": "No Content",
  "description": "Không có hợp đồng nào.",
  "data": null,
  "timestamp": "2024-01-01T00:00:00Z",
  "requestId": "uuid-here",
  "path": "/api/v1/contract-management-service/contracts"
}
```

## Implementation

### Java (Spring Boot)
- Java đã có `RestResponse<T>` và `GlobalExceptionHandler` sinh đúng schema
- Sử dụng `ResponseEntity<RestResponse<T>>` trong Controller
- Ví dụ:
```java
return ResponseEntity.ok(RestResponse.<Page<Document>>builder()
    .statusCode(200)
    .shortMessage("Success")
    .description("Đã lấy danh sách tài liệu thành công")
    .data(documents)
    .build());
```

### Python (FastAPI)
- Python service cần trả về cùng định dạng (đã có trong router của AI service; tiếp tục duy trì)
- Sử dụng Pydantic model để validate response format
- Ví dụ:
```python
class RestResponse(BaseModel):
    apiVersion: str = "v1"
    statusCode: int
    shortMessage: str
    description: str
    data: Optional[Any] = None
    timestamp: str
    requestId: str
    path: str
```

## Ví dụ Response Types

### Success Response (200)
```json
{
  "apiVersion": "v1",
  "statusCode": 200,
  "shortMessage": "Success",
  "description": "Đã lấy danh sách tài liệu thành công",
  "data": {
    "content": [Document objects],
    "pageable": {...},
    "totalElements": 100,
    "totalPages": 10
  },
  "timestamp": "2024-01-01T00:00:00Z",
  "requestId": "uuid-here",
  "path": "/api/v1/file-management-service/documents"
}
```

### Created Response (201)
```json
{
  "apiVersion": "v1",
  "statusCode": 201,
  "shortMessage": "Created",
  "description": "Đã tạo người dùng thành công",
  "data": {
    "id": "user-123",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "timestamp": "2024-01-01T00:00:00Z",
  "requestId": "uuid-here",
  "path": "/api/v1/user-management-service/users"
}
```

### Error Response (400)
```json
{
  "apiVersion": "v1",
  "statusCode": 400,
  "shortMessage": "Bad Request",
  "description": "Dữ liệu đầu vào không hợp lệ",
  "data": {
    "errors": [
      "Email không đúng định dạng",
      "Tên không được để trống"
    ]
  },
  "timestamp": "2024-01-01T00:00:00Z",
  "requestId": "uuid-here",
  "path": "/api/v1/user-management-service/users"
}
```

### Not Found Response (404)
```json
{
  "apiVersion": "v1",
  "statusCode": 404,
  "shortMessage": "Not Found",
  "description": "Không tìm thấy tài nguyên",
  "data": null,
  "timestamp": "2024-01-01T00:00:00Z",
  "requestId": "uuid-here",
  "path": "/api/v1/user-management-service/users/123"
}
```

---

**Lưu ý**: Tất cả API endpoints phải tuân thủ format này để đảm bảo tính nhất quán và dễ dàng xử lý ở client.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DevGO2003)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DevGO2003)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
