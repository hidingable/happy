# Happy Coder 数据模型与 API

## 数据库模型（Prisma Schema）

Schema 位置: `packages/happy-server/prisma/schema.prisma`

### 核心模型

```
┌──────────────────────────────────────────────────────────┐
│                      Account（用户）                      │
├──────────────────────────────────────────────────────────┤
│ id            String    @id @default(cuid())             │
│ publicKey     String    @unique   ← 加密公钥              │
│ name          String?             ← 显示名               │
│ avatar        String?             ← 头像                 │
│ encryptedData String?             ← 加密的用户数据       │
│ githubId      String?             ← 关联 GitHub          │
│ sessions      Session[]           ← 关联会话             │
│ machines      Machine[]           ← 关联设备             │
│ artifacts     Artifact[]          ← 关联 Artifacts       │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│                   Session（AI 会话）                      │
├──────────────────────────────────────────────────────────┤
│ id                  String    @id                        │
│ accountId           String              ← 所属用户       │
│ encryptedData       String?             ← 加密的会话元数据│
│ encryptedDataKey    String?             ← 加密的 DEK     │
│ deleted             Boolean   @default(false)            │
│ messages            SessionMessage[]    ← 消息列表       │
│ createdAt / updatedAt                                    │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│               SessionMessage（会话消息）                   │
├──────────────────────────────────────────────────────────┤
│ id                  String    @id                        │
│ sessionId           String              ← 所属会话       │
│ seq                 Int                 ← 消息序号       │
│ encryptedData       String              ← 加密消息内容   │
│ kind                String?             ← 消息类型       │
│ createdAt                                                │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│                   Machine（设备）                         │
├──────────────────────────────────────────────────────────┤
│ id                  String    @id                        │
│ accountId           String              ← 所属用户       │
│ publicKey           String    @unique   ← 设备公钥       │
│ encryptedData       String?             ← 加密设备信息   │
│ encryptedDataKey    String?             ← 加密的 DEK     │
│ version             String?             ← CLI 版本       │
│ lastSeen            DateTime?           ← 最后在线时间   │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│                  Artifact（用户产物）                      │
├──────────────────────────────────────────────────────────┤
│ id                  String    @id                        │
│ accountId           String              ← 所属用户       │
│ encryptedData       String              ← 加密内容       │
│ encryptedDataKey    String              ← 加密的 DEK     │
│ deleted             Boolean   @default(false)            │
│ createdAt / updatedAt                                    │
└──────────────────────────────────────────────────────────┘
```

### 辅助模型

| 模型 | 用途 |
|------|------|
| `UserRelationship` | 用户间社交关系（好友请求/已添加） |
| `UserFeedItem` | 活动 Feed 条目 |
| `UserKVStore` | 加密的键值存储（用户设置同步等） |
| `AccountPushToken` | 推送通知 Token（APNs/FCM） |
| `TerminalAuthRequest` | 终端 QR 码认证请求 |
| `AccountAuthRequest` | 账户认证请求 |
| `ServiceAccountToken` | OAuth Token 存储 |
| `UploadedFile` | 上传文件记录 |
| `GithubUser` / `GithubOrganization` | GitHub 集成 |
| `UsageReport` | Token 使用量统计 |
| `AccessKey` | 会话访问密钥 |
| `GlobalLock` / `RepeatKey` / `SimpleCache` | 基础设施 |

---

## API 路由

路由注册中心: `packages/happy-server/sources/app/api/api.ts`

### 认证

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/v1/auth/request` | 创建终端认证请求（QR 码） |
| POST | `/v1/auth/response` | 手机端响应认证 |
| POST | `/v1/auth/check` | 检查认证状态 |
| POST | `/v1/auth/account/request` | 创建账户认证请求 |
| POST | `/v1/auth/account/response` | 响应账户认证 |

### 会话

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/v1/sessions` | 获取会话列表 |
| POST | `/v1/sessions` | 创建会话 |
| GET | `/v1/sessions/:id` | 获取会话详情 |
| PUT | `/v1/sessions/:id` | 更新会话 |
| DELETE | `/v1/sessions/:id` | 删除会话 |
| GET | `/v1/sessions/:id/messages` | 获取会话消息 |
| POST | `/v3/sessions/:id/messages` | 发送消息（V3 协议） |

### 设备

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/v1/machines` | 获取设备列表 |
| POST | `/v1/machines` | 注册设备 |
| PUT | `/v1/machines/:id` | 更新设备信息 |
| DELETE | `/v1/machines/:id` | 删除设备 |

### 其他

| 模块 | 路径前缀 | 说明 |
|------|---------|------|
| 推送通知 | `/v1/push/` | Token 注册 |
| 账户 | `/v1/account/` | Profile CRUD |
| 连接 | `/v1/connect/` | OAuth 集成 |
| 语音 | `/v1/voice/` | LiveKit Token |
| Artifacts | `/v1/artifacts/` | 加密 Artifacts CRUD |
| Access Keys | `/v1/access-keys/` | 会话访问密钥 |
| Feed | `/v1/feed/` | 活动流 |
| KV Store | `/v1/kv/` | 加密键值存储 |
| 用户 | `/v1/users/` | 用户搜索/Profile |
| 版本 | `/v1/version/` | CLI 版本检查 |

---

## 实时通信（Socket.IO）

服务端: `packages/happy-server/sources/app/api/socket.ts`

### Socket 事件

| 事件 | 方向 | 说明 |
|------|------|------|
| `usage` | Client → Server | 上报 Token 使用量 |
| `rpc` | Client → Server | RPC 调用 |
| `ping` | 双向 | 心跳 |
| `session-update` | Server → Client | 会话变更通知 |
| `machine-update` | Server → Client | 设备变更通知 |
| `artifact-update` | Server → Client | Artifact 变更通知 |
| `access-key-update` | Server → Client | 访问密钥变更通知 |

---

## Wire Protocol（共享类型）

位置: `packages/happy-wire/src/`

### 核心 Schema

```typescript
// messages.ts - 消息类型
SessionMessage: { id, seq, encryptedData, kind, createdAt }

// MessageContent 解密后的消息内容结构
MessageContent: { role, content, toolCalls, ... }

// UpdateContainer - 实时更新容器
UpdateContainer:
  | { type: 'new-message', ... }
  | { type: 'update-session', ... }
  | { type: 'update-machine', ... }
```

所有 Schema 使用 **Zod** 定义，在 CLI、App、Server 间共享，保证类型一致性。
