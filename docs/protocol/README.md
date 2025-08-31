# PingPong Protocol 文档说明

## 文档结构

本目录包含了 PingPong 协议的所有文档，按照功能类别进行了组织：

```
docs/protocol/
├── Overview.md                    # 顶层协议概览
├── README.md                      # 本文档
├── Auth/                          # 认证与鉴权
│   ├── Overview.md               # 认证鉴权概览
│   ├── SSEToken.md               # SSE Token 详情
│   └── POSTToken.md              # POST Token 详情
├── ServiceManagement/             # 服务管理 API
│   ├── Overview.md               # 服务管理概览
│   ├── GetAdmin.md               # 📖 获取管理员
│   ├── AddAdmin.md               # ✏️ 增加管理员
│   └── RevokeAdmin.md            # ✏️ 撤销管理员
├── OrgManagement/                 # 组织管理 API
│   ├── Overview.md               # 组织管理概览
│   ├── CreateOrg.md              # ✏️ 创建组织
│   ├── GetOrgInfo.md             # 📖 获取组织信息
│   ├── GetOrgAdmin.md            # 📖 获取组织管理员
│   └── GetOrgMember.md           # 📖 获取组织成员
└── TaskManagement/                # 任务管理 API
    └── Overview.md               # 任务管理概览
```

## 文档规范

### 文件命名
- 每个 API 一个独立的 `.md` 文件
- 文件名使用 PascalCase 格式
- 文件名反映 API 的主要功能

### 内容结构
每个 API 文档都包含：
1. **功能描述** - API 的主要用途
2. **请求类型** - TypeScript 类型定义
3. **鉴权规则** - 权限要求
4. **返回类型** - 成功和错误的响应格式
5. **使用示例** - 请求和响应的具体示例

### 类型标签
- 📖 = READ（只读请求）
- ✏️ = WRITE（修改请求）

### 返回格式
- 成功时返回 HTTP 200 和对应的 Result 类型
- 失败时返回对应的 HTTP 错误状态码和 Error 类型

## 快速导航

- [协议概览](./Overview.md) - 查看协议概述、实体概念和 API 分类
- [认证鉴权](./Auth/Overview.md) - 用户身份验证和 Token 机制
- [服务管理](./ServiceManagement/Overview.md) - 服务级别的管理操作
- [组织管理](./OrgManagement/Overview.md) - 组织架构和权限管理
- [任务管理](./TaskManagement/Overview.md) - 核心业务功能

## 贡献指南

添加新的 API 文档时：
1. 在对应类别目录下创建新的 `.md` 文件
2. 更新对应类别的 `Overview.md` 文件
3. 更新顶层的 `Overview.md` 文件
4. 遵循现有的文档格式和规范
