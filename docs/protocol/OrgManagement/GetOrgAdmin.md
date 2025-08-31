# 📖 获取组织管理员

## 功能

获取指定组织的所有管理员列表

## 请求 Body 类型

```typescript
type GetOrgAdminRequest = {
  type: 'GET_ORG_ADMIN';  // API 操作类型标识
  orgId: string;  // 组织 Id
}
```

## 鉴权规则

调用者需要是该组织的成员或管理员

## 返回类型

**成功响应 (HTTP 200)：**
```typescript
type GetOrgAdminResult = {
  orgId: string;
  admins: string[];  // 组织管理员用户 Id 列表
}
```

**错误响应：**
```typescript
type GetOrgAdminError = 
  | { code: 'UNAUTHORIZED'; message: string }        // HTTP 401
  | { code: 'FORBIDDEN'; message: string }           // HTTP 403
  | { code: 'ORG_NOT_FOUND'; message: string }       // HTTP 404
  | { code: 'INTERNAL_ERROR'; message: string };     // HTTP 500
```

## 使用示例

### 请求
```http
POST /api
Authorization: Bearer <token>
Content-Type: application/json

{
  "type": "GET_ORG_ADMIN",
  "orgId": "org123"
}
```

### 成功响应
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "orgId": "org123",
  "admins": ["user1", "user2"]
}
```

### 错误响应
```http
HTTP/1.1 403 Forbidden
Content-Type: application/json

{
  "code": "FORBIDDEN",
  "message": "权限不足，需要组织成员权限"
}
```
