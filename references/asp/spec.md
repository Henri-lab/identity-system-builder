# ASP 协议规范正文

> 本文使用 RFC 2119 语义：**必须（MUST）**、**禁止（MUST NOT）**、**应当（SHOULD）**、**可以（MAY）**。
> 协议面向"外部可观察行为"，不规定表名、字段名、进程拓扑、语言与中间件。

## 1. 通用约定

- **标识符**：所有对象有不可变、全局唯一、不透明的引用 `*_ref`；调用方禁止解析其内部结构。
- **principal**：认证层的唯一产出。下游系统只消费 `principal`，**不得自造**；本协议不要求 `principal` 等于任何业务主键。
- **时间**：所有持久化时间戳为 UTC；过期/时间窗以**服务端时间**为准（对齐 MSP INV-21）。
- **幂等键**：改变状态的操作必须接受 `idempotency_key`（或天然业务键）；同一键重复调用返回相同结果、无二次副作用。
- **密钥纪律**：签名/加密密钥独立管理、可轮换、永不进仓库/镜像/日志（对齐 MSP 凭据纪律）。
- **扩展**：扩展只能新增字段/类型枚举/事件类型，禁止改变既有语义。

## 2. 范围与边界

- 本协议只管**人类身份**。机器身份（API key、mTLS、私钥/证书登录）与 SSH 运维接入是
  **ServiceIdentity 模块**（v0.2+），不在 v0.1 范围内；
- 认证层不判业务权限：登录成功 ≠ 有访问权（三分离）；业务判权在业务层，权益判权在 MSP（INV-3）；
- 不规定前端形态（Cookie/Header 由实现决定），只规定会话的**服务端可撤销性**与安全属性。

## 3. 对象模型（最小必需字段）

| 对象 | 关键字段 | 约束 |
| --- | --- | --- |
| User | `user_ref, display_name, status, created_at` | **不存**登录标识与凭据（A-1） |
| Identity | `user_ref, provider, provider_subject, email?, phone?, verified` | `unique(provider, provider_subject)`；1 user : N identities |
| Credential | `user_ref, password_hash, changed_at` | 只存哈希；算法参数随哈希串（A-2） |
| Session | `session_ref, user_ref, token_hash, created_at, expires_at, revoked_at, last_seen_at, device_meta` | 服务端只存哈希；可撤销（A-3） |
| VerificationCode | `purpose, principal_ref, code_hash, expires_at, attempts, consumed_at` | 一次性、绑定目的（A-6） |
| LoginAttempt | `principal_ref?, success, reason, ip_masked, at` | 审计事实，不可变 |

## 4. 状态机（允许的迁移是封闭集合）

```
User:    active ⇄ disabled → deleted
Session: active → revoked（过期等价 revoked，由时间判定）
Identity: unverified → verified（不得逆向；变更标识 = 新 Identity）
VerificationCode: issued → consumed | expired | exhausted
```

未列出的迁移必须被拒绝且无副作用（对齐 MSP INV-4/5）。

## 5. 操作定义

### OP-A1 `register`
- 输入：`provider, provider_subject, credential?, profile?, idempotency_key`
- 副作用（同事务）：创建 User + Identity +（可选）Credential。 `[EVT] user.registered`
- 错误：`IDENTITY_TAKEN`（同 provider+subject 已存在）/ `INVALID_CREDENTIAL`
- 防枚举：对"已存在"与"非法输入"返回相同语义与耗时（A-5）。

### OP-A2 `login`
- 输入：`provider, provider_subject, credential? | id_token?`
- 前置：账号未禁用；失败计数未触发锁定。
- 输出：新会话（**登录后必须轮换会话标识**，防固定攻击，A-4）。
- 副作用：记录 LoginAttempt（成败皆记）。 `[EVT] auth.login_succeeded | auth.login_failed`
- 错误：统一返回 `INVALID_CREDENTIALS`（不区分"用户不存在/密码错"）；锁定窗口内 → `RATE_LIMITED`。

### OP-A3 `logout` / OP-A4 `revokeSession`
- `logout` 撤销当前会话；`revokeSession`（本人/管理员）撤销指定会话或该用户全部会话。
- 副作用：`revoked_at` 置位（条件更新，幂等）。 `[EVT] session.revoked`

### OP-A5 `verifyIdentity`（邮箱/手机验证码）
- 发码：限频（同 principal 同 purpose 每分钟 ≤1）；校验：次数限制、短 TTL、**消费即失效**（A-6）。
- 通过后 Identity `verified`。 `[EVT] identity.verified`

### OP-A6 `resetPassword`
- 前置：必须经已验证渠道（邮箱/手机验证码）确认主体（A-7）。
- 副作用：更新 Credential；**撤销该用户全部其它会话**。 `[EVT] password.reset`

### OP-A7 `changePassword`（登录态）
- 前置：提供旧密码；副作用同 A-6（撤销其它会话）。

### OP-A8 `linkIdentity`
- 输入：`user_ref, provider, provider_subject`（经该 provider 认证后）
- 若 subject 已被他人绑定 → `LINK_CONFLICT`，**不得自动合并**（合并是显式流程，见 MSP OP-M2）。 `[EVT] identity.linked`

### OP-A9 `disableAccount` / `enableAccount`（管理员）
- 禁用 → 该用户**所有会话立即失效**（A-3）；启用 → 重置锁定计数。 `[EVT] account.disabled/enabled`

### OP-A10 `requestErasure`（P2，对接 data-retention）
- 掩码/删除 PII；审计事件保留匿名引用；会话全部撤销。 `[EVT] account.erasure_requested`

### P2 可选操作

| 操作 | 语义 | 关键点 |
| --- | --- | --- |
| OP-M1 `enrollTOTP` / `verifyTOTP` | TOTP（RFC 6238）+ 备份码 | 未通过 2FA **不得**发会话；secret 加密存储 |
| OP-F1 `startOIDC` / `oidcCallback` | 联合登录 | `state` 防 CSRF；以 `provider_subject` 绑定，禁止仅凭邮箱自动合并 |

## 6. 全局不变量（A）

- **A-1（身份与凭据分离）** User 不存登录标识与凭据；Identity 唯一键 `(provider, provider_subject)`。
- **A-2（口令存储）** 口令只用 Argon2id（或同强度 KDF）；明文永不落库/日志/缓存；哈希参数随哈希串。
- **A-3（会话可撤销）** 会话 token 随机不可预测、服务端只存哈希；改密码、封禁、强制下线必须撤销相关会话。
- **A-4（登录轮换）** 登录成功后必须换发新会话标识，防会话固定。
- **A-5（防枚举与限流）** 登录/注册对"不存在 vs 密码错"返回统一结果与恒定耗时；连续失败触发限速/锁定；成败皆审计（不含明文）。
- **A-6（验证码一次性）** 验证码绑定 purpose + principal、短 TTL、次数限制、消费即失效。
- **A-7（重置经验证渠道）** 密码重置必须经已验证邮箱/手机确认主体；成功后撤销其它会话。
- **A-8（权限与权益分离）** 认证层只发 `principal`；业务授权与权益判权在业务层（MSP INV-3），认证层不得代判。
- **A-9（时钟与密钥权威）** 服务端 UTC；密钥独立管理可轮换、不进仓库。
- **A-10（审计）** 注册/登录成败/会话撤销/账号状态变更必须留 actor + reason + 时间（含掩码后的 IP/设备）。
- **A-11（注销与事实分离）** 删除请求走掩码/匿名化；审计事实保留匿名引用，不得破坏审计链（对齐 MSP INV-34）。
- **A-12（会话边界校验）** 每次请求在服务端校验会话（存在、未过期、未撤销、账号未禁用）；前端隐藏按钮不构成授权。

## 7. 错误模型（最小集合）

| 代码 | 语义 | 约束 |
| --- | --- | --- |
| `INVALID_CREDENTIALS` | 认证失败（统一，不区分原因） | 恒定耗时，无副作用 |
| `RATE_LIMITED` | 触发限速/锁定 | 无副作用 |
| `IDENTITY_TAKEN` / `LINK_CONFLICT` | 标识已被占用/绑定冲突 | 无副作用 |
| `NOT_VERIFIED` / `CODE_INVALID` / `CODE_EXHAUSTED` | 验证前置不满足 | 无副作用 |
| `SESSION_EXPIRED` / `SESSION_REVOKED` / `ACCOUNT_DISABLED` | 会话/账号状态不满足 | 401/403，无副作用 |

## 8. 并发与限流

- MUST：验证码消费、会话撤销、锁定计数递增为条件更新/原子操作；
- MUST：登录失败限流按 `principal`（含不存在的 principal 按 `provider_subject` 维度）计数；
- SHOULD：对注册/发码接口做整体限流，防短信/邮件轰炸。

## 9. v0.2+ 候选（按加法原则追加）

- ServiceIdentity：API key（哈希存储、作用域、轮换）、mTLS、私钥/证书登录——**机器身份**；
- Passkey/WebAuthn；OIDC 深化（PKCE、登出回传）；多租户隔离（每次访问服务端校验 scope）。
