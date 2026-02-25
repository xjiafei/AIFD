# API 设计规范

> 本规范定义RESTful API设计标准和OpenAPI 3.0文档规范

## 目录

- [RESTful API设计原则](#restful-api设计原则)
- [API命名规范](#api命名规范)
- [HTTP方法](#http方法)
- [HTTP状态码](#http状态码)
- [请求和响应格式](#请求和响应格式)
- [版本控制](#版本控制)
- [分页和过滤](#分页和过滤)
- [错误处理](#错误处理)
- [安全性](#安全性)
- [OpenAPI 3.0规范](#openapi-30规范)

---

## RESTful API设计原则

### 1. 资源导向

API应该围绕**资源（Resources）**设计，而非操作。

```
✅ 好的设计（资源导向）：
GET    /users          - 获取用户列表
GET    /users/123      - 获取单个用户
POST   /users          - 创建用户
PUT    /users/123      - 更新用户
DELETE /users/123      - 删除用户

❌ 不好的设计（操作导向）：
GET    /getUsers
POST   /createUser
POST   /updateUser
POST   /deleteUser
```

### 2. 使用名词而非动词

资源使用**名词复数形式**，避免使用动词。

```
✅ 好的设计：
/users
/orders
/products

❌ 不好的设计：
/getUsers
/createOrder
/deleteProduct
```

### 3. 层次化结构

使用URL层次结构表示资源之间的关系。

```
✅ 好的设计：
GET /users/123/orders          - 获取用户123的订单列表
GET /users/123/orders/456      - 获取用户123的订单456
POST /users/123/orders         - 为用户123创建订单

❌ 不好的设计：
GET /getUserOrders?userId=123
GET /orders?userId=123&orderId=456
```

### 4. 无状态

每个请求应包含所有必要信息，服务器不应依赖之前的请求。

```
✅ 好的设计：
GET /users/123
Authorization: Bearer eyJhbGc...

❌ 不好的设计：
登录后依赖服务器端session
```

---

## API命名规范

### 1. URL路径规范

#### 1.1 使用小写字母
```
✅ 好的：/api/users
❌ 不好：/API/Users
```

#### 1.2 使用连字符分隔单词
```
✅ 好的：/api/user-profiles
✅ 好的：/api/order-items
❌ 不好：/api/userProfiles
❌ 不好：/api/user_profiles
```

#### 1.3 使用复数形式
```
✅ 好的：/api/users
✅ 好的：/api/orders
❌ 不好：/api/user
❌ 不好：/api/order
```

#### 1.4 避免深层嵌套
嵌套层级不超过3层。

```
✅ 好的：
/api/users/123/orders
/api/orders/456/items

❌ 不好（太深）：
/api/organizations/123/departments/456/teams/789/members/012
```

如果嵌套太深，考虑使用查询参数：
```
✅ 更好的设计：
/api/members?teamId=789
```

### 2. 查询参数规范

#### 2.1 使用驼峰命名
```
✅ 好的：
GET /api/users?pageSize=20&sortBy=createdAt

❌ 不好：
GET /api/users?page_size=20&sort_by=created_at
```

#### 2.2 常用查询参数
```
# 分页
page=1           # 页码
pageSize=20      # 每页大小
limit=20         # 限制数量
offset=0         # 偏移量

# 排序
sortBy=createdAt    # 排序字段
sortOrder=desc      # 排序方向（asc/desc）

# 过滤
status=active       # 状态过滤
createdAfter=2026-01-01  # 时间过滤
keyword=search      # 关键词搜索

# 字段选择
fields=id,name,email  # 只返回指定字段
```

---

## HTTP方法

### 1. 标准HTTP方法

| 方法 | 说明 | 幂等性 | 安全性 |
|------|------|--------|--------|
| GET | 获取资源 | ✓ | ✓ |
| POST | 创建资源 | ✗ | ✗ |
| PUT | 更新资源（全量） | ✓ | ✗ |
| PATCH | 更新资源（部分） | ✗ | ✗ |
| DELETE | 删除资源 | ✓ | ✗ |
| HEAD | 获取资源元信息 | ✓ | ✓ |
| OPTIONS | 获取支持的方法 | ✓ | ✓ |

### 2. 使用场景

#### 2.1 GET - 获取资源
```
GET /api/users              # 获取用户列表
GET /api/users/123          # 获取单个用户
GET /api/users/123/orders   # 获取用户的订单

特点：
- 不修改服务器状态
- 可缓存
- 可书签
- 参数在URL中
```

#### 2.2 POST - 创建资源
```
POST /api/users
Content-Type: application/json

{
  "username": "john",
  "email": "john@example.com"
}

响应：
201 Created
Location: /api/users/123

{
  "id": 123,
  "username": "john",
  "email": "john@example.com",
  "createdAt": "2026-02-13T10:00:00Z"
}

特点：
- 创建新资源
- 非幂等（多次调用创建多个资源）
- 返回201状态码
- 响应中包含资源Location
```

#### 2.3 PUT - 完全更新资源
```
PUT /api/users/123
Content-Type: application/json

{
  "username": "john_updated",
  "email": "john.new@example.com",
  "age": 30
}

响应：
200 OK

{
  "id": 123,
  "username": "john_updated",
  "email": "john.new@example.com",
  "age": 30,
  "updatedAt": "2026-02-13T11:00:00Z"
}

特点：
- 完全替换资源
- 幂等（多次调用结果相同）
- 必须提供所有字段
```

#### 2.4 PATCH - 部分更新资源
```
PATCH /api/users/123
Content-Type: application/json

{
  "email": "john.new@example.com"
}

响应：
200 OK

{
  "id": 123,
  "username": "john",
  "email": "john.new@example.com",
  "age": 25,
  "updatedAt": "2026-02-13T11:00:00Z"
}

特点：
- 部分更新资源
- 只需提供要更新的字段
- 更灵活
```

#### 2.5 DELETE - 删除资源
```
DELETE /api/users/123

响应：
204 No Content

或

200 OK
{
  "message": "User deleted successfully"
}

特点：
- 删除资源
- 幂等（多次删除结果相同）
- 返回204或200
```

---

## HTTP状态码

### 1. 状态码分类

#### 1xx - 信息响应
很少使用，通常由服务器自动处理。

#### 2xx - 成功
| 状态码 | 说明 | 使用场景 |
|--------|------|----------|
| 200 OK | 成功 | GET、PUT、PATCH成功 |
| 201 Created | 已创建 | POST创建资源成功 |
| 202 Accepted | 已接受 | 异步任务已接受 |
| 204 No Content | 无内容 | DELETE成功，不返回内容 |

#### 3xx - 重定向
| 状态码 | 说明 | 使用场景 |
|--------|------|----------|
| 301 Moved Permanently | 永久重定向 | 资源永久移动 |
| 302 Found | 临时重定向 | 资源临时移动 |
| 304 Not Modified | 未修改 | 缓存有效 |

#### 4xx - 客户端错误
| 状态码 | 说明 | 使用场景 |
|--------|------|----------|
| 400 Bad Request | 错误请求 | 参数验证失败 |
| 401 Unauthorized | 未认证 | 需要登录 |
| 403 Forbidden | 禁止访问 | 权限不足 |
| 404 Not Found | 未找到 | 资源不存在 |
| 405 Method Not Allowed | 方法不允许 | HTTP方法不支持 |
| 409 Conflict | 冲突 | 资源冲突（如用户名重复） |
| 422 Unprocessable Entity | 无法处理 | 语义错误（如业务规则验证失败） |
| 429 Too Many Requests | 请求过多 | 限流 |

#### 5xx - 服务器错误
| 状态码 | 说明 | 使用场景 |
|--------|------|----------|
| 500 Internal Server Error | 服务器错误 | 服务器内部错误 |
| 501 Not Implemented | 未实现 | 功能未实现 |
| 502 Bad Gateway | 网关错误 | 网关或代理错误 |
| 503 Service Unavailable | 服务不可用 | 服务器维护或过载 |
| 504 Gateway Timeout | 网关超时 | 网关超时 |

### 2. 状态码选择指南

```
成功场景：
GET     /api/users/123     → 200 OK
POST    /api/users         → 201 Created
PUT     /api/users/123     → 200 OK
PATCH   /api/users/123     → 200 OK
DELETE  /api/users/123     → 204 No Content

失败场景：
GET     /api/users/999     → 404 Not Found（资源不存在）
POST    /api/users         → 400 Bad Request（参数错误）
POST    /api/users         → 409 Conflict（用户名重复）
PUT     /api/users/123     → 403 Forbidden（权限不足）
DELETE  /api/users/123     → 422 Unprocessable Entity（有关联数据无法删除）
```

---

## 请求和响应格式

### 1. 统一响应格式

#### 1.1 成功响应
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 123,
    "username": "john",
    "email": "john@example.com"
  },
  "timestamp": "2026-02-13T10:00:00Z"
}
```

#### 1.2 列表响应（带分页）
```json
{
  "code": 200,
  "message": "success",
  "data": [
    {"id": 1, "username": "john"},
    {"id": 2, "username": "jane"}
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 100,
    "totalPages": 5
  },
  "timestamp": "2026-02-13T10:00:00Z"
}
```

#### 1.3 错误响应
```json
{
  "code": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Email format is invalid"
    },
    {
      "field": "password",
      "message": "Password must be at least 8 characters"
    }
  ],
  "timestamp": "2026-02-13T10:00:00Z",
  "path": "/api/users",
  "traceId": "abc123"  // 用于问题追踪
}
```

### 2. 请求格式规范

#### 2.1 Content-Type
```
application/json          # 默认使用JSON
application/xml           # 如需支持XML
multipart/form-data       # 文件上传
application/x-www-form-urlencoded  # 表单提交
```

#### 2.2 请求头规范
```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer eyJhbGc...
User-Agent: MyApp/1.0
X-Request-ID: abc123
```

### 3. 字段命名规范

#### 3.1 使用驼峰命名
```json
✅ 好的：
{
  "userId": 123,
  "userName": "john",
  "createdAt": "2026-02-13T10:00:00Z"
}

❌ 不好：
{
  "user_id": 123,
  "UserName": "john",
  "created_at": "2026-02-13T10:00:00Z"
}
```

#### 3.2 时间格式
使用 **ISO 8601** 格式：
```json
{
  "createdAt": "2026-02-13T10:00:00Z",        // UTC时间
  "updatedAt": "2026-02-13T10:00:00+08:00"    // 带时区
}
```

#### 3.3 布尔值
使用 `true/false`，不使用 `1/0` 或字符串：
```json
✅ 好的：
{
  "isActive": true,
  "hasPermission": false
}

❌ 不好：
{
  "isActive": 1,
  "hasPermission": "false"
}
```

---

## 版本控制

### 1. 版本控制策略

#### 方案1：URL路径版本（推荐）
```
GET /api/v1/users
GET /api/v2/users
```

**优点**：
- 清晰明了
- 易于理解和使用
- 便于缓存

**缺点**：
- URL冗余

#### 方案2：请求头版本
```
GET /api/users
Accept: application/vnd.example.v1+json
```

**优点**：
- URL简洁

**缺点**：
- 不够直观
- 调试困难

#### 方案3：查询参数版本（不推荐）
```
GET /api/users?version=1
```

**缺点**：
- 容易被忽略
- 缓存问题

### 2. 版本演进规则

- **主版本号（v1, v2）**：不兼容的API变更
- **次版本号（v1.1, v1.2）**：向后兼容的功能添加
- **修订号（v1.1.1）**：向后兼容的bug修复

```
v1.0.0 → v1.1.0  # 添加新字段（兼容）
v1.1.0 → v2.0.0  # 删除字段或改变字段类型（不兼容）
```

### 3. 弃用策略

```json
{
  "deprecated": true,
  "deprecationDate": "2026-12-31",
  "migrateTo": "/api/v2/users",
  "message": "This endpoint will be removed on 2026-12-31"
}
```

---

## 分页和过滤

### 1. 基于页码的分页

```
GET /api/users?page=1&pageSize=20

响应：
{
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 100,
    "totalPages": 5,
    "hasNext": true,
    "hasPrev": false
  }
}
```

### 2. 基于游标的分页

```
GET /api/users?cursor=eyJpZCI6MTAwfQ&limit=20

响应：
{
  "data": [...],
  "pagination": {
    "nextCursor": "eyJpZCI6MTIwfQ",
    "prevCursor": "eyJpZCI6ODAwfQ",
    "hasNext": true
  }
}
```

### 3. 过滤

```
GET /api/users?status=active&role=admin&createdAfter=2026-01-01

支持的操作符：
- 等于：status=active
- 不等于：status!=inactive
- 大于：age>18
- 小于：age<60
- 包含：name~john
- 范围：createdAt>=2026-01-01&createdAt<=2026-12-31
```

### 4. 排序

```
GET /api/users?sortBy=createdAt&sortOrder=desc

支持多字段排序：
GET /api/users?sortBy=createdAt,username&sortOrder=desc,asc
```

### 5. 字段选择

```
# 只返回指定字段
GET /api/users?fields=id,username,email

# 排除某些字段
GET /api/users?exclude=password,salt
```

---

## 错误处理

### 1. 错误响应结构

```json
{
  "code": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "code": "INVALID_FORMAT",
      "message": "Email format is invalid"
    }
  ],
  "timestamp": "2026-02-13T10:00:00Z",
  "path": "/api/users",
  "method": "POST",
  "traceId": "abc-123-def-456"
}
```

### 2. 错误码设计

#### 2.1 错误码格式
```
格式：HTTP状态码 + 业务错误码

示例：
400001 - 参数缺失
400002 - 参数格式错误
401001 - 未登录
403001 - 权限不足
404001 - 资源不存在
409001 - 资源冲突
```

#### 2.2 常见错误码
```json
{
  "400001": "参数缺失",
  "400002": "参数格式错误",
  "400003": "参数值非法",
  "401001": "未登录",
  "401002": "Token已过期",
  "401003": "Token无效",
  "403001": "权限不足",
  "404001": "资源不存在",
  "409001": "资源已存在",
  "409002": "资源冲突",
  "422001": "业务规则验证失败",
  "429001": "请求过于频繁",
  "500001": "服务器内部错误",
  "503001": "服务暂时不可用"
}
```

### 3. 字段验证错误

```json
{
  "code": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "username",
      "code": "REQUIRED",
      "message": "Username is required"
    },
    {
      "field": "email",
      "code": "INVALID_FORMAT",
      "message": "Email format is invalid"
    },
    {
      "field": "age",
      "code": "OUT_OF_RANGE",
      "message": "Age must be between 18 and 120"
    }
  ]
}
```

---

## 安全性

### 1. 认证

#### 1.1 Bearer Token（推荐）
```http
GET /api/users
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### 1.2 API Key
```http
GET /api/users
X-API-Key: your-api-key
```

### 2. HTTPS

- **强制使用HTTPS**
- 自动将HTTP重定向到HTTPS

### 3. CORS

```javascript
// 服务器端配置
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

### 4. 限流

```http
# 响应头
X-RateLimit-Limit: 100          # 限制数
X-RateLimit-Remaining: 95       # 剩余数
X-RateLimit-Reset: 1638360000   # 重置时间

# 超过限制
429 Too Many Requests
{
  "code": 429,
  "message": "Rate limit exceeded",
  "retryAfter": 60
}
```

### 5. 输入验证

- 验证所有输入参数
- 使用白名单而非黑名单
- 防止SQL注入
- 防止XSS攻击
- 防止CSRF攻击

---

## OpenAPI 3.0规范

### 1. OpenAPI文档结构

```yaml
openapi: 3.0.0
info:
  title: User Management API
  version: 1.0.0
  description: API for managing users
  contact:
    name: API Support
    email: support@example.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: https://staging-api.example.com/v1
    description: Staging server

paths:
  /users:
    get:
      summary: Get user list
      description: Retrieve a list of users with pagination
      operationId: getUsers
      tags:
        - Users
      parameters:
        - name: page
          in: query
          description: Page number
          schema:
            type: integer
            default: 1
        - name: pageSize
          in: query
          description: Page size
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserListResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'

    post:
      summary: Create a new user
      description: Create a new user with the provided data
      operationId: createUser
      tags:
        - Users
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: User created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          $ref: '#/components/responses/Conflict'

  /users/{id}:
    get:
      summary: Get user by ID
      description: Retrieve a single user by their ID
      operationId: getUserById
      tags:
        - Users
      parameters:
        - name: id
          in: path
          required: true
          description: User ID
          schema:
            type: integer
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
          example: 123
        username:
          type: string
          example: john
        email:
          type: string
          format: email
          example: john@example.com
        createdAt:
          type: string
          format: date-time
          example: 2026-02-13T10:00:00Z

    CreateUserRequest:
      type: object
      required:
        - username
        - email
        - password
      properties:
        username:
          type: string
          minLength: 4
          maxLength: 20
          example: john
        email:
          type: string
          format: email
          example: john@example.com
        password:
          type: string
          format: password
          minLength: 8
          example: Pass1234

    UserListResponse:
      type: object
      properties:
        code:
          type: integer
          example: 200
        message:
          type: string
          example: success
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        pagination:
          $ref: '#/components/schemas/Pagination'

    Pagination:
      type: object
      properties:
        page:
          type: integer
        pageSize:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer

    ErrorResponse:
      type: object
      properties:
        code:
          type: integer
        message:
          type: string
        errors:
          type: array
          items:
            type: object
            properties:
              field:
                type: string
              message:
                type: string

  responses:
    BadRequest:
      description: Bad Request
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

    Unauthorized:
      description: Unauthorized
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

    NotFound:
      description: Not Found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

    Conflict:
      description: Conflict
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

### 2. OpenAPI文档编写规范

#### 2.1 描述要清晰
- **summary**：简短描述（一句话）
- **description**：详细描述（可多行）
- **operationId**：唯一标识符（用于代码生成）

#### 2.2 使用标签分组
```yaml
tags:
  - name: Users
    description: User management operations
  - name: Orders
    description: Order management operations
```

#### 2.3 使用$ref复用
```yaml
# 复用Schema
$ref: '#/components/schemas/User'

# 复用Response
$ref: '#/components/responses/NotFound'

# 复用Parameter
$ref: '#/components/parameters/PageParam'
```

#### 2.4 提供示例
```yaml
schema:
  type: object
  properties:
    username:
      type: string
      example: john      # 提供示例值
```

---

## 总结

### 核心原则

1. ✅ **一致性**：API设计保持一致的风格和规范
2. ✅ **简洁性**：URL清晰简洁，易于理解
3. ✅ **可预测性**：遵循RESTful约定，行为可预测
4. ✅ **安全性**：使用HTTPS，认证授权，输入验证
5. ✅ **文档化**：使用OpenAPI 3.0编写详细文档
6. ✅ **版本控制**：合理的版本管理和演进策略
7. ✅ **错误处理**：清晰的错误信息和状态码
8. ✅ **性能**：支持分页、过滤、字段选择

### 快速检查清单

- [ ] 使用名词复数形式的资源路径
- [ ] 使用正确的HTTP方法
- [ ] 返回正确的HTTP状态码
- [ ] 统一的请求响应格式
- [ ] 完整的错误处理
- [ ] API版本控制
- [ ] 完整的OpenAPI文档
- [ ] 认证和授权
- [ ] 输入验证
- [ ] 分页和过滤支持
- [ ] HTTPS强制使用
- [ ] 限流机制

遵循本规范，可以设计出**一致、易用、安全、可维护**的API。
