# ✏️ 撤销管理员

## 功能

撤销指定用户的管理员权限

## 请求 Body 类型

```typescript
type RevokeAdminRequest = {
  type: 'REVOKE_ADMIN';  // API 操作类型标识
  adminIds: string[];  // 要撤销的管理员用户 Id 列表
}
```

## 鉴权规则

调用者需要是现有管理员

## 返回类型

**成功响应 (HTTP 200)：**
```typescript
type RevokeAdminResult = {
  revoked: string[];  // 成功撤销的管理员用户 Id 列表
  skipped: string[];  // 跳过撤销的用户 Id 列表（不是管理员）
}
```

**错误响应：**
```typescript
type RevokeAdminError = 
  | { code: 'UNAUTHORIZED'; message: string }        // HTTP 401
  | { code: 'FORBIDDEN'; message: string }           // HTTP 403
  | { code: 'INVALID_INPUT'; message: string }       // HTTP 400
  | { code: 'INTERNAL_ERROR'; message: string };     // HTTP 500
```

## 使用示例

### 请求
```http
POST /api
Authorization: Bearer <token>
Content-Type: application/json

{
  "type": "REVOKE_ADMIN",
  "adminIds": ["user1", "user2"]
}
```

### 成功响应
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "revoked": ["user1"],
  "skipped": ["user2"]
}
```

### 错误响应
```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "code": "INVALID_INPUT",
  "message": "不能撤销最后一个管理员"
}
```
