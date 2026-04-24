## carbonflow

> 本文件为 ClimateSeal 项目中所有内部和外部API的设计提供了统一的规范和最佳实践。一个一致、可预测的API是保证系统稳定、降低前后端联调成本的关键。

# API Design Guide & Communication Protocols

本文件为 ClimateSeal 项目中所有内部和外部API的设计提供了统一的规范和最佳实践。一个一致、可预测的API是保证系统稳定、降低前后端联调成本的关键。

---

### **1. 核心原则**

-   **风格**: 所有新的API应遵循 **RESTful** 设计风格。使用标准的HTTP方法表达操作意图。
-   **无状态**: API应该是无状态的。服务器不应保存任何客户端的会话状态。每次请求都应包含所有必要的信息。
-   **JSON**: 所有API的请求体和响应体都必须使用 **JSON** 格式。`Content-Type` 应设置为 `application/json`。

---

### **2. 命名与URL结构**

-   **资源驱动**: URL应围绕"资源"进行设计。使用名词而不是动词。
-   **URL格式**: `/<api-version>/<resource-plural>`
    -   示例: `/v1/projects`, `/v1/users`, `/v1/projects/{projectId}/calculations`
-   **命名规范**:
    -   路径和查询参数: 使用 `camelCase` (e.g., `?userId=123`)
    -   JSON字段: 使用 `camelCase`

---

### **3. HTTP方法使用 (CRUD)**

-   **`GET`**: 读取资源。安全且幂等。
    -   `GET /v1/projects`: 获取所有项目列表。
    -   `GET /v1/projects/{projectId}`: 获取单个项目的详细信息。
-   **`POST`**: 创建一个新资源。非幂等。
    -   `POST /v1/projects`: 创建一个新项目。
-   **`PUT`**: **完全替换**一个已存在的资源。幂等。
    -   `PUT /v1/projects/{projectId}`: 传入完整的项目对象，替换现有项目。
-   **`PATCH`**: **部分更新**一个已存在的资源。非幂等。
    -   `PATCH /v1/projects/{projectId}`: 只传入需要修改的字段，如 `{"name": "New Project Name"}`。
-   **`DELETE`**: 删除一个资源。幂等。
    -   `DELETE /v1/projects/{projectId}`: 删除一个项目。

---

### **4. 标准化响应结构**

#### **成功响应 (2xx)**

-   `200 OK`: 请求成功。
-   `201 Created`: 资源创建成功。响应中应包含 `Location` 头，指向新资源的URL。
-   `204 No Content`: 请求成功，但没有响应体（例如 `DELETE` 操作）。

**数据响应格式**:

```json
{
  "data": {
    // 资源对象或资源对象列表
    "id": "proj_123",
    "name": "My Project"
  }
}
```

**列表响应格式 (分页)**:

```json
{
  "data": [
    { "id": "proj_123", "name": "My Project" },
    { "id": "proj_456", "name": "Another Project" }
  ],
  "pagination": {
    "total": 100,
    "limit": 20,
    "offset": 0
  }
}
```

#### **错误响应 (4xx, 5xx)**

-   `400 Bad Request`: 客户端请求无效（如参数错误）。
-   `401 Unauthorized`: 未认证。
-   `403 Forbidden`: 已认证，但无权限访问该资源。
-   `404 Not Found`: 请求的资源不存在。
-   `500 Internal Server Error`: 服务器端发生未知错误。

所有错误响应都必须遵循以下格式:

```json
{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "The 'projectName' field cannot be empty.",
    "requestId": "req_abc123"
  }
}
```

---

### **5. 文档**

-   所有API端点都**必须**有清晰的文档。
-   推荐使用 **OpenAPI (Swagger)** 规范来编写和维护API文档。文档应与代码一同存放在版本库中。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AllenPeng0209) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
