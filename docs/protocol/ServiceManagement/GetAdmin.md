# 📖 获取管理员

## 功能

获取服务实例的所有管理员列表

## 请求 Body 类型

```typescript
type GetAdminRequest = {
  type: 'GET_ADMIN';  // API 操作类型标识
}
```

## 鉴权规则

调用者需要是现有管理员

## 返回类型

**成功响应 (HTTP 200)：**
```typescript
type GetAdminResult = {
  admins: string[];  // 管理员用户 Id 列表
}
```

**错误响应：**
```typescript
type GetAdminError = 
  | { code: 'UNAUTHORIZED'; message: string }        // HTTP 401
  | { code: 'FORBIDDEN'; message: string }           // HTTP 403
  | { code: 'INTERNAL_ERROR'; message: string };     // HTTP 500
```

## 使用示例

### 请求
```http
POST /api
Authorization: Bearer <token>
Content-Type: application/json

{
  "type": "GET_ADMIN"
}
```

### 成功响应
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "admins": ["user1", "user2", "user3"]
}
```

### 错误响应
```http
HTTP/1.1 403 Forbidden
Content-Type: application/json

{
  "code": "FORBIDDEN",
  "message": "权限不足，需要管理员权限"
}
```
