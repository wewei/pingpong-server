# PingPong API 概览

## 概述

PingPong 是一个以人为本的轻量任务管理系统。
PingPong 尊重人的主体性和自主性。
主体性体现在，人的身份先于组织、服务存在，每个用户的身份由自签的公私密钥对确定，无需在任何一个服务上注册。人与组织的关系也不是从属，而是授权：即组织赋予某个用户在组织内发起任务的权限。
自主性体现在，任何任务的分配都应得到接收用户的确认才能生效，且系统应尽量保护用户不超量任务、信息淹没，每个人只应关注与自己相关的任务与信息。

鉴于此，PingPong 采用 Federated Service 架构。每个用户可以和多个 Service 互动。PingPong Protocol 就是用户客户端与 Service 之间的通讯协议。

PingPong 协议定义的 API 均通过统一的 endpoint URL 进行访问。
客户端请求操作统一使用 HTTP POST 方法访问 endpoint URL。
服务端事件推送统一使用 HTTP GET 方法配合 Server-Sent Events (SSE) 访问 endpoint URL。

## 实体概念

每个服务实例 (Service) 可以服务一个或多个组织 (Org)，每个组织可以授权一个或多个用户，在组织中发起任务，任务归属于组织。
每个服务有一个或多个管理员 (admin)，每个组织也有一个或多个管理员。
每个任务有一个固定的请求者 (requester)，和一个可以改变的响应者 (respondent)。
实体关系如下：

```text
┌─────────┐                ┌─────┐                ┌──────┐
│ Service │──owns [1..n]──→│ Org │──owns [0..n]──→│ Task │
└────┬────┘                └──┬──┘                └───┬──┘
     │                        │                       │
     │ ref admin [1..n]       │ ref admin [1..n]      │ ref requester [1]
     │                        │ ref member [0..n]     │ ref respondent [1]
     │                        │                       │
     └────────────────────────┼───────────────────────┘
                              │
                              ▼
                          ┌──────┐
                          │ User │
                          └──────┘
```

## API 分类

从功能层面，所有的 API 分成 3 个类别：

1. **服务管理**：负责服务实例级别的管理操作，包括管理员的增删查改，确保服务的安全性和可维护性。
2. **组织管理**：处理服务内租户组织的管理操作，包括组织的创建、信息维护，以及组织内管理员和成员的权限管理，实现组织层面的访问控制。
3. **任务管理**：核心业务功能，涵盖任务的完整生命周期管理，从发起、接收到响应、关闭，以及通过 SSE 实现任务事件的实时推送和监听。

## 目录结构

### [Auth](./Auth/)
认证与鉴权机制
- 用户 Id 生成
- SSE Token
- POST Token

### [ServiceManagement](./ServiceManagement/)
服务实例级别的管理操作
- 📖 获取管理员
- ✏️ 增加管理员
- ✏️ 撤销管理员

### [OrgManagement](./OrgManagement/)
服务内租户组织的管理操作
- ✏️ 创建组织
- 📖 获取组织信息
- ✏️ 修改组织信息
- 📖 获取组织管理员
- ✏️ 增加组织管理员
- ✏️ 撤销组织管理员
- 📖 获取组织成员
- ✏️ 增加组织成员
- ✏️ 撤销组织成员

### [TaskManagement](./TaskManagement/)
核心业务功能，任务生命周期管理
- ✏️ 发起任务
- ✏️ 接收任务
- ✏️ 回应任务
- ✏️ 关闭任务

## 认证与鉴权

详细的认证与鉴权机制请参考 [Auth 文档](./Auth/Overview.md)，包括：

- **用户 Id 生成** - Ed25519 公私钥对和 RIPEMD-160(SHA-256) 哈希
- **SSE Token** - 用于长连接认证的轻量级 Token
- **POST Token** - 用于请求认证的双重验证 Token

所有 API 调用都需要在 Authorization Header 中携带相应的 Bearer Token。
