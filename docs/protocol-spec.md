# PingPong 协议规范 (v1)

版本: 1.0

概述
---
PingPong 是一个基于“限时消息（Ping）”与“回应（Pong）”的联邦式协作协议。协议目标：简单、可审计、支持去中心化身份（DID），并采用中继路由模式（relay）。

传输与模式
---
- 所有 Client → Server、Server → Server 消息均使用 HTTP POST。
- Server → Client 使用 Server-Sent Events (SSE)，即客户端通过 GET 建立持久 SSE 连接接收事件推送。
- 协议仅支持中继模式（`relay`）：发件人的 home 服务器负责存档、策略检查并转发到目标服务器。组织可以通过策略要求所有交互必须经过其 home 服务器以保证审计与同步。

规范性约定
---
- 所有传输必须使用 TLS（HTTPS）。
- 消息使用 JSON 编码，Content-Type: application/json。
- 身份采用 DID（Decentralized Identifier）；消息签名采用发件人私钥（推荐 Ed25519 / Ed448 或其他适合的椭圆签名算法）。
- 请求附带防重放与完整性字段（Digest、Nonce、Date）。

HTTP Header 约定
---
常用 Header：
- Content-Type: application/json
- Date: RFC 7231 时间戳（或同时提供 X-PingPong-Timestamp: ISO8601）
- Digest: SHA-256=<base64 of body digest>
- Signature: keyId="<did>#<key-id>",algorithm="Ed25519",signature="<base64>"
  - Signature 为对请求的规范化字符串或请求体（见签名规范）签名后的 base64 编码。
- X-PingPong-Nonce: <随机串> （防重放）
- Idempotency-Key: <client-provided-id>（幂等）

签名规范（建议）
---
- 推荐使用 HTTP Signatures / JWS Detached 签名：对以下串联内容做签名（字符串顺序固定）：
  1. HTTP 方法（大写）
  2. Request-Target（如: POST /v1/messages）
  3. Date (RFC7231)
  4. Digest (SHA-256 base64)
  5. X-PingPong-Nonce

示例：
```
POST\n/v1/messages\nTue, 02 Sep 2025 10:00:00 GMT\nSHA-256=...\nnonce-12345
```
对该串进行签名，结果放入 `Signature` header 的 signature 字段，并把 keyId 指向签名者的 DID key（例如 `did:pingpong:alice#key-1`）。

消息模型（JSON）
---
核心消息体（message）示例：

```json
{
  "messageId": "msg-123",
  "threadId": "thread-1",
  "from": "did:pingpong:alice",
  "to": "did:pingpong:bob",
  "mode": "relay",
  "createdAt": "2025-09-02T10:00:00Z",
  "deadline": "2025-09-02T12:00:00Z",
  "content": "请在2小时内回复",
  "attachments": [],
  "metadata": {"sensitivity":"normal"}
}
```

备注：
- `messageId` 由发送方生成并应在全局内尽可能唯一（例如使用 UUIDv4）。
- `threadId` 用于将属于同一往返的消息聚合。

服务间信封（Envelope）
---
当消息经过中继服务器时，服务器应使用 `envelope` 将原始消息与中继元数据打包并为该跳次签名：

```json
{
  "envelope": {
    "original": { /* message */ },
    "originServer": "https://server-a.example.com",
    "returnPath": ["https://server-a.example.com"],
    "signatures": [
      {
        "keyId": "did:pingpong:server-a#key-1",
        "signature": "base64...",
        "timestamp": "2025-09-02T10:00:01Z"
      }
    ]
  }
}
```

接收方服务器在验证 inbound 请求时，应：
1. 验证 `Digest` 与请求体一致。
2. 验证 `original` 中 `from` 的签名（如果 `original` 内包含发件人签名）。
3. 验证 envelope.signatures（每一跳的服务器签名）以确保 returnPath 的完整性和可审计性。

端点规范
---
以下列出常见 API 端点（基于 /v1）：

Client → Server

- POST /v1/messages
  - 描述：发送新消息（Ping）。客户端签名请求。
  - 请求 body：message JSON（见上文）。
  - 必要 Header：Date、Digest、Signature、X-PingPong-Nonce
  - 响应：
    - 201 Created
      - Location: /v1/messages/{messageId}
      - Body: { "messageId": "...", "status": "accepted" }
    - 400 Bad Request / 401 / 409

- POST /v1/messages/{messageId}/supplement
  - 描述：在目标尚未回应前追加补充（Supplement）。
  - 请求 body：{ "content": "补充内容", "timestamp": "..." }
  - 如果消息已被回应，返回 409 Conflict，body 中包含当前回应的摘要。

- POST /v1/messages/{messageId}/respond
  - 描述：接收方对指定 message 的正式回应（Pong）。
  - 请求 body：{ "responseId": "resp-1", "content": "....", "timestamp": "..." }
  - 服务器在接收到回应后：更新状态为 responded，阻止后续对该 message 的 supplement 成功。

Server → Server

- POST /v1/inbound
  - 描述：其他服务器将经过包装的消息转发到本服务器。
  - 请求 body：{ "envelope": { ... } }
  - 验证通过后，存储并将消息推送至本地目标用户（或在离线时缓存）。
  - 响应：200 OK（或 4xx / 5xx）。

Server → Client (SSE)

- GET /v1/events
  - 描述：客户端建立 SSE 连接以接收事件。
  - 认证：建议使用短期 token（OAuth2 Bearer 或基于 DID 的会话 token）用于鉴权。
  - 可订阅事件类型：message.created, message.supplemented, message.responded, message.expired, message.status

SSE 事件示例：

```
event: message.created
id: msg-123
data: {"messageId":"msg-123","from":"did:pingpong:alice","threadId":"thread-1","deadline":"..."}

event: message.responded
id: resp-1
data: {"messageId":"msg-123","responseId":"resp-1","responder":"did:pingpong:alice","timestamp":"..."}
```

消息状态机
---
状态：
- pending：已发送，等待回应（可追加 supplement）
- supplemented：发送方追加过信息，仍等待回应
- responded：接收方已回应（禁止进一步追加到该轮）
- expired：截止时间到，未回应
- cancelled：被发送方或系统取消

转移规则：
- pending → supplemented (发送方追加)
- pending|supplemented → responded (接收方回应)
- pending|supplemented → expired (到期)
- 任意 → cancelled（显式取消）

受限追加逻辑
---
- 在目标 `message` 尚未被回应之前，原发送方可以多次调用 `/supplement` 为该 message 增补信息。
- 一旦接收方提交 `respond`，服务器应将该 message 标记为 `responded`，并对后续 `/supplement` 请求返回 409 Conflict，响应体应包含当前的回应（或其摘要）并提示发送方是否要基于新的回应开启新一轮 Ping。

事件与回执（MessageStatus）
---
服务器应向订阅方和来源方发出以下状态事件：
- message.delivered（目标服务器已接受并存储）
- message.presented（目标用户在线并已推送）
- message.read（如果客户端支持“已读”回执）
- message.responded
- message.expired

回执格式（示例）:
```json
{
  "messageId": "msg-123",
  "status": "delivered",
  "timestamp": "2025-09-02T10:01:00Z",
  "by": "https://server-b.example.com"
}
```

幂等与去重
---
- 客户端在发送创建或补充请求时应提供 `Idempotency-Key`，服务器基于此保证幂等。
- Server→Server 转发需要保留 `messageId` 与 `envelope.returnPath`，避免循环转发。服务器收到已经见过的 `messageId` 应返回 200 并给出现有状态而非再次处理。

发现（Discovery）
---
- 接收方的服务器地址与 capabilities 应由 DID 的 serviceEndpoints 字段提供（优先）。
- 如果 DID 无公开 serviceEndpoint，可通过公共注册器或目录服务查找。

错误格式
---
错误统一返回 JSON：
```json
{
  "error": "MESSAGE_ALREADY_RESPONDED",
  "code": 409,
  "message": "The message has already been responded by the recipient.",
  "detail": { /* optional */ }
}
```

安全与运维考量
---
- 强制 HTTPS
- 检查 `Date` 与服务器时间差，允许合理时间窗口（例如 ±5 分钟）
- 非法请求或签名失败必须返回 401/403
- 对于高风险敏感字段（attachments），建议端到端加密
- 记录并验证 Nonce 以抵抗重放攻击（服务器保留短期 Nonce 列表）
- 速率限制与反垃圾策略（Spam）

拓展与兼容性
---
- 建议使用 JSON-LD 或在未来用更高效的二进制格式（Protobuf）替代 JSON 保持向后兼容
- 可以为企业提供可选审计扩展、合规导出 API

示例流程（Relay 模式）
---
1. Alice Client -> POST /v1/messages (ServerA)
   - ServerA 验证签名并存储消息（状态 pending）
   - ServerA 找到 Bob 的 server（ServerB），构造 envelope 并对 envelope 签名
   - ServerA POST https://server-b.example.com/v1/inbound
2. ServerB 验证原始签名与 envelope 签名，存储并推送给 Bob（若在线则通过 SSE）
3. Bob 在客户端回复：POST /v1/messages/{msg}/respond 到 ServerB
   - ServerB 标记 responded，并将回应回递到 ServerA（依据 returnPath）
   - ServerA 更新本地消息状态并通过 SSE 通知 Alice

附录：术语表
---
- Ping：限时消息（发送方期望在 deadline 前获得回应）
- Pong：回应（接收方对 Ping 的回复）
- Envelope：服务器间转发时的包装结构，包含原始消息与中继元数据
- ReturnPath：消息在中继路径上的服务器列表（可用于回传与审计）

---

此规范为初版草案，后续可补充细化：签名具体的规范（JWS/HTTP Signatures）、DID 发现细节、SSE 权限模型、以及 API 的完整 OpenAPI/JSON Schema 描述。
