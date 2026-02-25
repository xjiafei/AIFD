# REST API设计模板

## API端点命名规范

- 使用复数名词: `/api/v1/users`, `/api/v1/orders`
- 禁用path变量
- 使用小写字母和连字符: `/api/v1/user-profiles`
- 版本控制: `/api/v1/`, `/api/v2/`

## 标准CRUD端点

### 创建资源 (Create)
```
POST /api/v1/users
Content-Type: application/json

Request:
{
  "name": "张三",
  "email": "zhangsan@example.com"
}

Response: 201 Created
{
  "id": 1,
  "name": "张三",
  "email": "zhangsan@example.com",
  "createdAt": "2025-01-01T10:00:00Z"
}
```

### 查询资源列表 (Read List)
```
GET /api/v1/users?page=1&size=20&sort=createdAt,desc&search=张

Response: 200 OK
{
  "content": [...],
  "page": 1,
  "size": 20,
  "totalElements": 100,
  "totalPages": 5
}
```


### 更新资源 (Update)
```
PUT /api/v1/users/1
Content-Type: application/json

Request:
{
  "name": "李四",
  "email": "lisi@example.com"
}

Response: 200 OK
{
  "id": 1,
  "name": "李四",
  "email": "lisi@example.com",
  "updatedAt": "2025-01-02T10:00:00Z"
}
```

### 删除资源 (Delete)
```
DELETE /api/v1/users/1

Response: 204 No Content
```

## 错误响应格式

```json
{
  "timestamp": "2025-01-01T10:00:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "邮箱格式不正确",
  "path": "/api/v1/users"
}
```

## HTTP状态码使用

- **200 OK**: 成功（GET, PUT, PATCH）
- **201 Created**: 创建成功（POST）
- **204 No Content**: 删除成功（DELETE）
- **400 Bad Request**: 请求参数错误
- **401 Unauthorized**: 未认证
- **403 Forbidden**: 无权限
- **404 Not Found**: 资源不存在
- **409 Conflict**: 资源冲突
- **500 Internal Server Error**: 服务器错误
