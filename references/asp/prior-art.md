# ASP 成熟先例映射（prior art）

> ASP 不自研密码学与协议；每个对象/操作对齐成熟标准，差异必须写"差分声明"。

## 映射表

| ASP 概念 | 成熟先例 | 对齐方式与吸收内容 |
| --- | --- | --- |
| User / Identity / Credential 分离 | Auth0 users + identities；Keycloak user + federated identity | `(provider, provider_subject)` 唯一；不把邮箱当主键 |
| 口令存储 | OWASP Password Storage Cheat Sheet；NIST SP 800-63B §5.1.1.2 | Argon2id；参数随哈希串；不用明文/可逆加密 |
| 服务端会话 | OWASP Session Management Cheat Sheet | 随机不可预测、只存哈希、登录轮换、Secure/HttpOnly/SameSite |
| 会话固定 | OWASP Session Fixation | 登录后换发会话标识（A-4） |
| 防枚举/爆破 | OWASP Authentication Cheat Sheet；NIST 800-63B §5.2.2 | 统一响应 + 恒定耗时 + 限速锁定（A-5） |
| 验证码 | OWASP Forgot Password Cheat Sheet | 一次性、短 TTL、限次、绑定目的（A-6） |
| TOTP | RFC 6238 | P2 MFA；备份码；未过 2FA 不发会话 |
| 联合登录 | OAuth 2.0 / OpenID Connect Core | `state` 防 CSRF；以 subject 绑定，不凭邮箱自动合并 |
| 账号生命周期 | Auth0/Keycloak 状态机 | active/disabled/deleted；禁用即撤会话 |
| 审计 | OWASP Logging Cheat Sheet | 成败皆记、掩码 IP、不含明文 |
| 三分离 | Auth0 的 authN vs authZ；MSP INV-3 | 认证层只发 principal，权益判权在 MSP |

## 禁止事项

- 不自研哈希/KDF/加密协议；不自研 OIDC；
- 不照抄私有表结构；协议约束行为；
- 每个差异必须"出处 + 差分 + 测试向量"三件套。
