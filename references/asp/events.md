# ASP 事件契约

> 事件是协议的一部分；传输方式（outbox/MQ/HTTP）由实现决定，不改变契约。
> 信封为 CloudEvents v1.0 的受限子集：`id` 与 `source` 组合全局唯一（对齐 MSP events.md §1）。

## 必需事件（按声明范围）

| 事件 | 触发 | 关键字段 |
| --- | --- | --- |
| `user.registered` | 注册成功 | `user_ref, provider, created_at` |
| `identity.verified` | 邮箱/手机验证通过 | `user_ref, provider, verified_at` |
| `auth.login_succeeded` | 登录成功 | `user_ref, session_ref, ip_masked, device_meta` |
| `auth.login_failed` | 登录失败 | `principal_ref?, provider_subject_masked, reason, ip_masked` |
| `session.revoked` | 登出/强制下线/改密码撤销 | `session_ref, user_ref, reason` |
| `password.reset` | 密码重置完成 | `user_ref, sessions_revoked` |
| `account.disabled` / `account.enabled` | 封禁/解封 | `user_ref, operator, reason` |
| `identity.linked` | 多登录方式绑定 | `user_ref, provider` |
| `account.erasure_requested` | 删除请求 | `user_ref, status`（不含被删内容） |
| `mfa.enrolled` / `federation.login_succeeded` | P2 可选 | `user_ref, provider / method` |

## 语义（MUST）

1. 状态变更与事件写入同事务（对齐 MSP INV-18）；
2. 消费者按 `event_id` 幂等；至少一次投递；
3. **安全事件（登录失败/封禁/撤销）永不丢弃**——是审计事实，遵循 data-retention 的"事实可留、标识可删"；
4. 事件负载不含明文密码、完整 token、完整 IP（只留掩码）。
