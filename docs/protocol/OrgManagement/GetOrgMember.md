# 📖 获取组织成员

## 功能

获取指定组织的所有成员列表

## 请求 Body 类型

```typescript
type GetOrgMemberRequest = {
  type: 'GET_ORG_MEMBER';  // API 操作类型标识
  orgId: string;  // 组织 Id
}
```

## 鉴权规则

调用者需要是该组织的成员或管理员

## 返回类型

**成功响应 (HTTP 200)：**
```typescript
type GetOrgMemberResult = {
  orgId: string;
  members: Array<{
    userId: string;
    role: 'ADMIN' | 'MEMBER';
    joinedAt: number;  // Unix 时间戳
  }>;
}
```

**错误响应：**
```typescript
type GetOrgMemberError = 
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
  "type": "GET_ORG_MEMBER",
  "orgId": "org123"
}
```

### 成功响应
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "orgId": "org123",
  "members": [
    {
      "userId": "user1",
      "role": "ADMIN",
      "joinedAt": 1640995200000
    },
    {
      "userId": "user2",
      "role": "MEMBER",
      "joinedAt": 1640995200000
    }
  ]
}
```

### 错误响应
```http
HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "code": "ORG_NOT_FOUND",
  "message": "组织不存在"
}
```
