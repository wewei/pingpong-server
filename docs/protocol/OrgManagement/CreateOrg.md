# ✏️ 创建组织

## 功能

创建新的组织实体

## 请求 Body 类型

```typescript
type CreateOrgRequest = {
  type: 'CREATE_ORG';  // API 操作类型标识
  name: string;  // 组织名称
  description?: string;  // 组织描述（可选）
}
```

## 鉴权规则

调用者需要是服务管理员

## 返回类型

**成功响应 (HTTP 200)：**
```typescript
type CreateOrgResult = {
  orgId: string;  // 新创建的组织 Id
  name: string;
  description?: string;
  createdAt: number;  // Unix 时间戳
  creatorId: string;  // 创建者用户 Id
}
```

**错误响应：**
```typescript
type CreateOrgError = 
  | { code: 'UNAUTHORIZED'; message: string }        // HTTP 401
  | { code: 'FORBIDDEN'; message: string }           // HTTP 403
  | { code: 'INVALID_INPUT'; message: string }       // HTTP 400
  | { code: 'DUPLICATE_NAME'; message: string }      // HTTP 409
  | { code: 'INTERNAL_ERROR'; message: string };     // HTTP 500
```

## 使用示例

### 请求
```http
POST /api
Authorization: Bearer <token>
Content-Type: application/json

{
  "type": "CREATE_ORG",
  "name": "开发团队",
  "description": "负责产品开发的核心团队"
}
```

### 成功响应
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "orgId": "org123",
  "name": "开发团队",
  "description": "负责产品开发的核心团队",
  "createdAt": 1640995200000,
  "creatorId": "user1"
}
```

### 错误响应
```http
HTTP/1.1 409 Conflict
Content-Type: application/json

{
  "code": "DUPLICATE_NAME",
  "message": "组织名称已存在"
}
```
