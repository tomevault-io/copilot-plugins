## das-rest-api

> 本规则适用于DAS系统内所有REST API的开发工作，包括：

## 1. 文档说明

### 1.1 适用范围

本规则适用于DAS系统内所有REST API的开发工作，包括：
- 内部API（使用"/api"前缀）
- 对外OPEN API（使用"/openapi"前缀）
- 第三方集成API接口

### 1.2 约束强度分类

| 强度级别 | 关键词 | 说明 |
| -------- | ------ | ---- |
| **强制性** | MUST、SHALL、REQUIRED | 必须严格遵守，违反将导致API不符合规范 |
| **推荐性** | SHOULD、RECOMMENDED | 强烈建议遵守，有助于提升API质量 |
| **禁止性** | MUST NOT、SHALL NOT | 绝对禁止的行为或做法 |
| **可选性** | MAY、OPTIONAL | 可根据具体场景选择是否采用 |

### 1.3 使用指导

- 开发者**必须**在API设计阶段检查相关规则
- 代码审查时**应该**对照本规则进行检查
- 测试阶段**推荐**基于本规则制定测试用例
- 对于历史接口，**不建议**仅为遵循规则而进行重大变更

---

## 2. 基础原则

### 2.1 设计理念

- **一致性优先**：确保所有API遵循统一的设计模式
- **易用性导向**：让API简单直观，便于前后端协作
- **标准化实施**：遵循REST架构风格和HTTP协议标准
- **向后兼容**：保证API版本间的兼容性和稳定性

### 2.2 核心原则

- API**必须**遵循RESTful设计原则
- URL**必须**表示资源而非动作
- HTTP方法**必须**准确表达操作语义
- 响应格式**必须**保持结构化和一致性
- 错误处理**必须**提供明确的问题定位信息
- **必须**使用强类型设计，禁止使用Map<String, Object>
- **必须**使用统一的错误码枚举
- **必须**使用统一的请求和响应基类

---

## 3. R1 - URL与版本设计规则

### 3.1 URL结构规范

#### R1.1 基础结构【强制性】

**规则描述**：所有API的URL**必须**遵循以下结构格式

```
{schema}://{host}:{port}/{prefix}/{version}/{collection}[/{id}]
```

**字段说明**：
- `{schema}` - 协议（http或https）
- `{host}` - 服务域名或IP地址  
- `{port}` - 服务端口
- `{prefix}` - 接口标识前缀（内部接口使用"/api"，对外接口使用"/openapi"）
- `{version}` - 接口版本（格式：vX.Y）
- `{collection}` - 集合名称，必须为复数名词
- `{id}` - 资源唯一标识符

**正确示例**：
```http
https://das.com/api/v1.0/agents
https://das.com/api/v1.0/agents/JHF8UE6H5W34D
https://das.com/openapi/v2.1/devices/12345/sensors
```

**错误示例**：
```http
# 错误：使用单数名词
https://das.com/api/v1.0/agent
# 错误：缺少版本号
https://das.com/api/agents  
# 错误：使用动词
https://das.com/api/v1.0/getAgents
```

#### R1.2 URL长度限制【推荐性】

**规则描述**：URL长度**应该**不超过2000个字符

**检查方法**：在API设计时验证完整URL长度

#### R1.3 资源标识符规范【强制性】

**规则描述**：
- 资源标识符**必须**唯一且稳定
- **禁止**在标识符中使用"/"字符（如业务需要，必须使用"\"转义）
- **推荐**使用有意义的索引字段（IP、邮箱等）

### 3.2 版本控制规范

#### R1.4 版本格式【强制性】

**规则描述**：
- **必须**支持显式版本控制
- 版本号**必须**使用major.minor格式（如v1.0, v2.1）
- 版本定义**必须**在/api路径之后

**正确示例**：
```http
https://das.com/api/v1.0/agents
https://das.com/api/v2.1/devices
```

#### R1.5 版本兼容性【强制性】

**规则描述**：
- **必须**保留历史版本接口，除非确认无客户端使用
- 不更新版本的情况下**可以**新增属性
- **禁止**删除或修改现有属性

---

## 4. R2 - HTTP方法与状态码规则

### 4.1 HTTP方法使用规范

#### R2.1 方法语义映射【强制性】

**规则描述**：HTTP方法**必须**准确表达操作语义

| 方法 | 操作描述 | 成功状态码 | 幂等性 |
| ---- | -------- | ---------- | ------ |
| **GET** | 获取资源当前值 | 200 OK | ✓ |
| **POST** | 创建新资源或提交操作 | 201 Created | ✗ |
| **PUT** | 更新资源（完整替换） | 200 OK | ✓ |
| **PATCH** | 更新资源（部分更新） | 200 OK | ✗ |
| **DELETE** | 删除资源 | 204 No Content | ✓ |

**正确示例**：
```http
GET /api/v1.0/agents           # 获取探针列表
POST /api/v1.0/agents          # 创建新探针
GET /api/v1.0/agents/12345     # 获取特定探针
PUT /api/v1.0/agents/12345     # 完整更新探针
PATCH /api/v1.0/agents/12345   # 部分更新探针
DELETE /api/v1.0/agents/12345  # 删除探针
```

#### R2.2 PATCH方法特殊规范【强制性】

**规则描述**：对不存在资源的PATCH请求**必须**返回409 Conflict状态码

### 4.2 状态码使用规范

#### R2.3 标准状态码【强制性】

**规则描述**：**必须**使用标准HTTP状态码，**必须**返回精确对应的状态码

**2xx 成功状态码**：
- `200 OK` - GET、PUT、PATCH操作成功
- `201 Created` - POST创建资源成功  
- `202 Accepted` - 请求已接受，用于异步操作
- `204 No Content` - DELETE操作成功

**4xx 客户端错误**：
- `400 Bad Request` - 请求语法错误或参数无效
- `401 Unauthorized` - 未提供认证凭据或认证失败
- `403 Forbidden` - 已认证但无访问权限
- `404 Not Found` - 请求资源不存在
- `405 Method Not Allowed` - HTTP方法不被允许
- `409 Conflict` - 请求冲突（如PATCH不存在的资源）
- `422 Unprocessable Entity` - 请求格式正确但语义错误
- `429 Too Many Requests` - 请求频率超限

**5xx 服务器错误**：
- `500 Internal Server Error` - 服务器内部错误
- `503 Service Unavailable` - 服务暂时不可用

#### R2.4 异步操作状态码【推荐性】

**规则描述**：异步操作**应该**返回202状态码并提供任务跟踪信息

**示例**：
```http
HTTP/1.1 202 Accepted
{
  "task": {
    "href": "/api/v1.0/tasks/12345",
    "id": "12345"
  }
}
```

---

## 5. R3 - 请求响应格式规则

### 5.1 请求头规范

#### R3.1 标准请求头【推荐性】

**规则描述**：**推荐**使用以下标准请求头，使用时**必须**保持一致

| 请求头 | 类型 | 说明 |
| ------ | ---- | ---- |
| Accept | Content type | 请求的响应内容类型，推荐：application/json |
| Accept-Language | Language code | 响应语言偏好（如支持国际化） |  
| Accept-Charset | Charset type | 响应编码，默认UTF-8 |
| Content-Type | Content type | 请求体媒体类型（PUT/POST/PATCH） |

### 5.2 响应格式规范

#### R3.2 成功响应格式【强制性】

**规则描述**：
- 成功响应**必须**是单个JSON对象
- **必须**包含名为"data"的键
- 单个资源时data**必须**是JSON对象
- 多个资源时data**必须**是JSON数组（可为空数组）

**正确示例**：
```json
// 单个资源
{
  "data": {
    "agentType": "soc",
    "softVersion": "3.0.1",
    "config": "",
    "version": 1611054006000
  }
}

// 多个资源
{
  "data": [
    {
      "agentType": "soc",
      "softVersion": "3.0.1"
    },
    {
      "agentType": "ainta", 
      "softVersion": "1.1.3"
    }
  ]
}

// 空结果
{
  "data": []
}

// 操作结果
{
  "data": 1
}
```

#### R3.3 错误响应格式【强制性】

**规则描述**：
- 错误响应**必须**是单个JSON对象
- **必须**包含名为"error"的键
- error对象**必须**包含"code"和"message"字段

**Error对象结构**：
| 字段 | 类型 | 必需 | 说明 |
| ---- | ---- | ---- | ---- |
| code | String | ✓ | 服务端定义的错误代码 |
| message | String | ✓ | 用户可读的错误描述 |
| target | String | - | 错误的具体目标 |
| details | Error[] | - | 详细错误数组 |
| innerError | InnerError | - | 更具体的内部错误 |

**正确示例**：
```json
{
  "error": {
    "code": "BadArgument",
    "message": "Previous passwords may not be reused",
    "target": "password",
    "innerError": {
      "code": "PasswordError",
      "innerError": {
        "code": "PasswordReuseNotAllowed"
      }
    }
  }
}
```

#### R3.4 响应内容类型【强制性】

**规则描述**：
- 默认响应格式**必须**是application/json
- JSON属性名**必须**采用驼峰命名规范

---

## 6. R4 - 强类型设计规范

### 6.1 基础类型规范

#### R4.1 强类型设计【强制性】

**规则描述**：
- **禁止**使用Map<String, Object>作为参数或返回值
- **必须**使用强类型的DTO、Entity、VO类
- **必须**继承统一的基类（BaseRequest、PaginationRequest等）

**正确示例**：
```java
// ✅ 正确：使用强类型
public class UserRequest extends BaseRequest {
    private String username;
    private String email;
}

// ❌ 错误：使用Map
public Map<String, Object> getUserInfo(String userId) {
    Map<String, Object> result = new HashMap<>();
    result.put("username", "john");
    return result;
}
```

#### R4.2 统一基类【强制性】

**规则描述**：
- 所有请求**必须**继承BaseRequest
- 分页请求**必须**继承PaginationRequest
- 所有响应**必须**使用ApiResponse<T>接口

**正确示例**：
```java
// 基础请求
public class CreateUserRequest extends BaseRequest {
    private String username;
    private String email;
}

// 分页请求
public class UserListRequest extends PaginationRequest {
    private String keyword;
    private String status;
}

// 统一响应
public ApiResponse<User> createUser(CreateUserRequest request) {
    // 实现逻辑
}
```

### 6.2 错误码标准化

#### R4.3 错误码枚举【强制性】

**规则描述**：
- **必须**使用ErrorCode枚举定义所有错误码
- **必须**为每个错误码提供对应的HTTP状态码
- **必须**提供详细的错误描述

**标准错误码**：
```java
public enum ErrorCode implements ResponseCode {
    // 参数错误
    BadArgument(HttpStatus.BAD_REQUEST),
    BadUserArgument(HttpStatus.BAD_REQUEST),
    BadSignArgument(HttpStatus.BAD_REQUEST),
    BadTimeArgument(HttpStatus.BAD_REQUEST),
    
    // 认证错误
    AuthFailed(HttpStatus.UNAUTHORIZED),
    Unauthorized(HttpStatus.UNAUTHORIZED),
    InvalidApiKey(HttpStatus.UNAUTHORIZED),
    InvalidToken(HttpStatus.UNAUTHORIZED),
    
    // 权限错误
    Forbidden(HttpStatus.FORBIDDEN),
    NotAcceptable(HttpStatus.NOT_ACCEPTABLE),
    
    // 资源错误
    NotFound(HttpStatus.NOT_FOUND),
    RequestTimeout(HttpStatus.REQUEST_TIMEOUT),
    
    // 业务错误
    Conflict(HttpStatus.CONFLICT),
    Overlimit(HttpStatus.CONFLICT),
    InvalidOperation(HttpStatus.BAD_REQUEST),
    
    // 系统错误
    InternalError(HttpStatus.INTERNAL_SERVER_ERROR),
    ServiceUnavailable(HttpStatus.SERVICE_UNAVAILABLE),
    GatewayTimeout(HttpStatus.GATEWAY_TIMEOUT);
}
```

### 6.3 验证和工具方法

#### R4.4 参数验证【推荐性】

**规则描述**：
- **应该**为每个DTO类提供验证方法
- **应该**使用jakarta.validation注解
- **应该**提供业务逻辑验证方法

**正确示例**：
```java
@Data
@EqualsAndHashCode(callSuper = true)
public class UserRequest extends BaseRequest {
    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 20, message = "用户名长度必须在3-20字符之间")
    private String username;
    
    @Email(message = "邮箱格式不正确")
    private String email;

}
```

#### R4.5 工具方法【推荐性】

**规则描述**：
- **应该**提供便捷的工具方法
- **应该**提供类型转换方法
- **应该**提供业务判断方法

**正确示例**：
```java
public class AssetRequest extends BaseRequest {
    private String assetType;
    private String ipAddress;
}
```

---

## 6. R4 - 数据类型与序列化规则

### 6.1 JSON基础类型规范

#### R4.1 允许的数据类型【强制性】

**规则描述**：JSON值**必须**是以下类型之一，**禁止**超出此范畴

| 数据类型 | 描述 | 示例 |
| -------- | ---- | ---- |
| String | 双引号包围的字符串 | "hello", "你好" |
| Number | 十进制数字（不允许前导零） | 1234, 123.22 |
| Boolean | 布尔值 | true, false |
| Object | 花括号包围的键值对 | {"a":1, "b":true} |
| Array | 方括号包围的值列表 | [1,2,3,4] |
| Null | 空值 | null |

#### R4.2 大整数处理【强制性】

**规则描述**：超过JavaScript安全整数范围的数值**必须**作为字符串返回

```json
// 错误：可能被截断
{"largeId": 9007199254740992}

// 正确：使用字符串
{"largeId": "9007199254740992"}
```

### 6.2 空值处理规范

#### R4.3 空值约定【强制性】

**规则描述**：为保证接口一致性，各类型空值**必须**遵循以下约定

| 数据类型 | 空值表示 |
| -------- | -------- |
| String | "" |
| Number | 0 |
| Boolean | 不允许空值 |
| Object | {} |
| Array | [] |

### 6.3 时间类型规范

#### R4.4 时间格式【强制性】

**规则描述**：
- 接口层面**必须**使用Unix时间戳传递日期时间
- **必须**使用毫秒级精度
- 前端根据用户时区自行处理展示

**正确示例**：
```json
{
  "createTime": 1611054006000,
  "updateTime": 1611054006000
}
```

#### R4.5 持续时长【推荐性】

**规则描述**：持续时长**应该**按ISO 8601标准序列化

**示例**：
```json
{
  "duration": "P3Y6M4DT12H30M5S"
}
```

---

## 7. R5 - 命名与标识符规则

### 7.1 基本命名规范

#### R5.1 命名格式【推荐性】

**规则描述**：
- **应该**使用详细单词而非非通用缩写
- 缩写**应该**遵循驼峰命名法
- 所有标识符**应该**使用驼峰命名法（lowerCamelCase）
- HTTP头属性使用Capitalized-Hyphenated-Terms格式

**正确示例**：
```json
{
  "agentName": "sensor-agent",
  "createDateTime": 1611054006000,
  "softwareUrl": "https://example.com"
}
```

**错误示例**：
```json
{
  "agent_name": "sensor-agent",    // 错误：使用下划线
  "create_date_time": 1611054006000,  // 错误：使用下划线
  "sw_url": "https://example.com"  // 错误：非通用缩写
}
```

#### R5.2 禁用词汇【禁止性】

**规则描述**：接口**禁止**使用以下单词命名

- Context
- Scope  
- Resource

### 7.2 特殊字段命名规范

#### R5.3 标识字段【强制性】

**规则描述**：
- 标识字段**必须**使用字符串类型定义
- **可以**使用简单"id"表示资源主键
- **应该**使用后缀"Id"表示外键关系

**示例**：
```json
{
  "id": "JHF8UE6H5W34D",
  "userId": "12345",
  "subscriptionId": "sub-67890"  
}
```

#### R5.4 时间字段【强制性】

**规则描述**：时间字段**必须**使用规定后缀

- 日期+时间：**必须**使用后缀"DateTime"
- 仅日期：**必须**使用后缀"Date" 
- 仅时间：**必须**使用后缀"Time"

**示例**：
```json
{
  "createDateTime": 1611054006000,
  "birthDate": 1611054006000,
  "startTime": 1611054006000
}
```

#### R5.5 计数和集合字段【强制性】

**规则描述**：
- 集合命名**必须**为复数名词
- 计数字段**必须**使用后缀"Count"

**示例**：
```json
{
  "agents": [...],
  "agentCount": 25,
  "totalCount": 100
}
```

### 7.3 常用字段标准化

#### R5.6 标准字段名【强制性】

**规则描述**：以下常用字段**必须**使用统一命名

| 标准字段名 | 用途 |
| ---------- | ---- |
| id | 主键标识 |
| name | 通用名称 |
| displayName | 显示名称 |
| createTime | 创建时间 |
| updateTime | 更新时间 |
| startTime | 开始时间 |
| endTime | 结束时间 |

---

## 8. R6 - 资源与集合操作规则

### 8.1 集合操作规范

#### R6.1 分页参数【推荐性】

**规则描述**：分页**应该**使用$page和$size参数

**参数说明**：
- `$page` - 页码（正整数，从1开始）
- `$size` - 每页数量（正整数）

**请求示例**：
```http
GET /api/v1.0/agents?$page=5&$size=10
```

**响应示例**：
```json
{
  "$page": 5,
  "$size": 10,
  "data": [...],
  "total": 100
}
```

#### R6.2 排序参数【推荐性】  

**规则描述**：排序**应该**使用$orderBy参数

**格式说明**：
- 多字段用逗号分隔
- 排序方向用空格分隔（asc/desc，默认asc）
- 空值排序"小于"非空值

**示例**：
```http
GET /api/v1.0/agents?$orderBy=name desc,createTime
```

**响应示例**：
```json
{
  "$orderBy": "name desc,createTime",
  "data": [...],
  "total": 100
}
```

#### R6.3 不支持的请求【强制性】

**规则描述**：不支持的功能**必须**返回400状态码和相应错误码

**错误码对照**：
- `ErrorUnsupportedOrderBy` - 不支持的排序字段
- `ErrorUnsupportedPaging` - 不支持分页

### 8.2 批量操作规范

#### R6.4 批量操作理解【推荐性】

**规则描述**：批量操作**应该**理解为对资源集合的操作

**示例**：
```http
# 批量获取
GET /api/v1.0/agents

# 批量创建
POST /api/v1.0/agents
{"data": [...]}

# 批量更新（部分更新）
PATCH /api/v1.0/agents  
{"data": [...]}

# 批量替换（完整更新）
PUT /api/v1.0/agents
{"data": [...]}

# 批量删除
DELETE /api/v1.0/agents/id1,id2,id3
DELETE /api/v1.0/agents?agentType=ainta
```

### 8.3 通用子资源规范

#### R6.5 文件子资源【推荐性】

**规则描述**：文件相关操作**应该**使用files子资源

```http
# 导出资源集合
GET /api/v1.0/agents/files

# 导出特定资源  
GET /api/v1.0/agents/12345/files?type=docx

# 导入到资源集合
POST /api/v1.0/agents/files

# 导入到特定资源
POST /api/v1.0/agents/12345/files?type=docx
```

#### R6.6 其他通用子资源【可选性】

**选项子资源（options）**：
```http
GET /api/v1.0/menus/options
```

**连通性子资源（connectivities）**：
```http  
POST /api/v1.0/assets/12345/connectivities
```

**同步子资源（synchronizations）**：
```http
POST /api/v1.0/assets/synchronizations
```

---

## 9. R7 - 客户端兼容性规则

### 9.1 客户端要求

#### R7.1 字段忽略原则【强制性】

**规则描述**：客户端**必须**安全忽略未约定字段

**场景说明**：服务端可能在不更改版本号的情况下添加新字段，客户端必须能正确处理

#### R7.2 字段顺序忽略【禁止性】

**规则描述**：客户端**禁止**依赖JSON响应字段顺序

#### R7.3 无声失效原则【推荐性】  

**规则描述**：客户端请求可选功能时**应该**具备兼容性，**可以**忽略不支持的功能

### 9.2 服务端兼容性保证

#### R7.4 向后兼容性【强制性】

**规则描述**：
- **必须**保留历史版本接口
- 同版本内**可以**新增字段
- **禁止**删除或修改现有字段

#### R7.5 版本演进策略【推荐性】

**规则描述**：
- 重大变更**应该**发布新版本
- 兼容性变更**可以**在同版本内进行
- 废弃功能**应该**提前通知并保持一定时间

---

## 10. 附录

### 10.1 HTTP状态码对照表

#### 常用状态码

| 状态码 | 名称 | 使用场景 |
| ------ | ---- | -------- |
| 200 | OK | GET、PUT、PATCH成功 |
| 201 | Created | POST创建成功 |
| 202 | Accepted | 异步操作已接受 |
| 204 | No Content | DELETE成功 |
| 400 | Bad Request | 请求语法错误 |
| 401 | Unauthorized | 未认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 405 | Method Not Allowed | 方法不允许 |
| 409 | Conflict | 请求冲突 |
| 422 | Unprocessable Entity | 语义错误 |
| 429 | Too Many Requests | 请求过频 |
| 500 | Internal Server Error | 服务器内部错误 |
| 503 | Service Unavailable | 服务不可用 |

### 10.2 标准错误代码表

| 错误代码 | 状态码 | 说明 |
| -------- | ------ | ---- |
| BadArgument | 400 | 参数无效 |
| BadUserArgument | 400 | 用户参数无效 |
| ErrorUnsupportedOrderBy | 400 | 不支持排序 |
| ErrorUnsupportedPaging | 400 | 不支持分页 |
| InvalidOperation | 400 | 操作无效 |
| Unauthorized | 401 | 未授权 |
| InvalidApiKey | 401 | 无效API密钥 |
| NotFound | 404 | 资源未找到 |
| RequestTimeout | 408 | 请求超时 |
| Conflict | 409 | 资源冲突 |
| Overlimit | 409 | 超出限制 |
| InternalError | 500 | 内部错误 |
| ServiceUnavailable | 503 | 服务不可用 |
| GatewayTimeout | 504 | 网关超时 |

### 10.3 快速检查清单

#### 设计阶段检查项

- [ ] URL结构是否符合规范格式
- [ ] 是否使用了正确的HTTP方法
- [ ] 响应格式是否包含必需字段
- [ ] 错误处理是否完整
- [ ] 字段命名是否符合规范
- [ ] 是否使用了强类型设计（禁止Map<String, Object>）
- [ ] 是否继承了统一的基类（BaseRequest、PaginationRequest）
- [ ] 是否使用了统一的错误码枚举（ErrorCode）
- [ ] 是否提供了参数验证方法
- [ ] 是否提供了工具方法

#### 开发阶段检查项  

- [ ] 状态码是否准确对应操作结果
- [ ] JSON格式是否符合标准
- [ ] 时间字段是否使用Unix时间戳
- [ ] 大整数是否转为字符串
- [ ] 空值处理是否一致
- [ ] 是否使用了强类型参数和返回值
- [ ] 是否使用了统一的异常处理机制
- [ ] 是否添加了详细的Javadoc注释
- [ ] 是否使用了Lombok简化代码
- [ ] 是否遵循了命名规范

#### 测试阶段检查项

- [ ] 异常场景是否返回正确错误码
- [ ] 分页排序功能是否正常
- [ ] 客户端兼容性是否良好
- [ ] API文档是否与实现一致
- [ ] 参数验证是否正常工作
- [ ] 错误码映射是否正确
- [ ] 强类型转换是否正常
- [ ] 工具方法是否按预期工作
- [ ] 性能是否满足要求
- [ ] 安全性是否符合标准

---

**文档结束**

> 本规则基于DAS REST API指南v1.1生成，旨在为开发者提供清晰、可操作的API开发规范。如有疑问或建议，请及时反馈并持续完善。

---

## 11. 附录 - 最佳实践示例

### 11.1 完整的API实现示例

```java
@RestController
@RequestMapping("/api/v1.0/users")
@Slf4j
public class UserController {
    
    @Autowired
    private UserService userService;
    
    /**
     * 创建用户
     */
    @PostMapping
    public ApiResponse<User> createUser(@Valid @RequestBody CreateUserRequest request) {
        log.info("创建用户请求: {}", request);
        
        try {
            User user = userService.createUser(request);
            return ApiResponse.success(user).build();
        } catch (ValidationException e) {
            log.warn("参数验证失败: {}", e.getMessage());
            return ApiResponse.error(ErrorCode.BadArgument)
                    .message(e.getMessage())
                    .target("request")
                    .build();
        } catch (Exception e) {
            log.error("创建用户失败", e);
            return ApiResponse.error(ErrorCode.InternalError)
                    .message("系统内部错误")
                    .build();
        }
    }
    
    /**
     * 查询用户列表
     */
    @GetMapping
    public ApiResponse<List<User>> getUserList(@Valid UserListRequest request) {
        log.info("查询用户列表: {}", request);
        
        try {
            List<User> users = userService.getUserList(request);
            return ApiResponse.success(users).build();
        } catch (Exception e) {
            log.error("查询用户列表失败", e);
            return ApiResponse.error(ErrorCode.InternalError)
                    .message("系统内部错误")
                    .build();
        }
    }
}
```

### 11.2 完整的DTO示例

```java
@Data
@EqualsAndHashCode(callSuper = true)
@JsonInclude(JsonInclude.Include.NON_NULL)
public class CreateUserRequest extends BaseRequest {
    
    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 20, message = "用户名长度必须在3-20字符之间")
    private String username;
    
    @Email(message = "邮箱格式不正确")
    private String email;
    
    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phone;
    
    @Size(max = 500, message = "描述长度不能超过500字符")
    private String description;

}
```

### 11.3 完整的服务层示例

```java
@Service
@Slf4j
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    /**
     * 创建用户
     */
    public User createUser(CreateUserRequest request) {
        log.info("创建用户: {}", request);
        
        // 参数验证
        if (!request.isValidUsername()) {
            throw new ValidationException("用户名格式不正确");
        }
        
        if (!request.isValidEmail()) {
            throw new ValidationException("邮箱格式不正确");
        }
        
        // 检查用户是否已存在
        if (userRepository.existsByUsername(request.getUsername())) {
            throw new BusinessException("用户名已存在");
        }
        
        // 创建用户
        User user = new User();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());
        user.setPhone(request.getPhone());
        user.setDescription(request.getDescription());
        user.setCreateTime(System.currentTimeMillis());
        
        return userRepository.save(user);
    }
    
    /**
     * 查询用户列表
     */
    public List<User> getUserList(UserListRequest request) {
        log.info("查询用户列表: {}", request);
        

        
        // 构建查询条件
        UserQuery query = UserQuery.builder()
                .keyword(request.getKeyword())
                .status(request.getStatus())
                .page(request.getPage())
                .size(request.getSize())
                .orderBy(request.getOrderBy())
                .build();
        
        return userRepository.findByQuery(query);
    }
}
```

---

---
> Source: [wwj-git-rgb/spring-ai-code-demo](https://github.com/wwj-git-rgb/spring-ai-code-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
