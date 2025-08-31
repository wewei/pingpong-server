# 📖 获取组织信息

## 功能

获取指定组织的详细信息

## 请求 Body 类型

```typescript
type GetOrgInfoRequest = {
  type: 'GET_ORG_INFO';  // API 操作类型标识
  orgId: string;  // 组织 Id
}
```

## 鉴权规则

调用者需要是该组织的成员或管理员

## 返回类型

**成功响应 (HTTP 200)：**
```typescript
type GetOrgInfoResult = {
  orgId: string;
  name: string;
  description?: string;
  createdAt: number;  // Unix 时间戳
  updatedAt: number;  // Unix 时间戳
}
```

**错误响应：**
```typescript
type GetOrgInfoError = 
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
  "type": "GET_ORG_INFO",
  "orgId": "org123"
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
  "updatedAt": 1640995200000
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
