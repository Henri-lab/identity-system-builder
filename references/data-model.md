# 数据模型目录（身份域，按档位裁剪）

## 约定

- 主键 `uuid default gen_random_uuid()`；时间 `timestamptz`（UTC）；状态 `text + check`；
- 唯一索引是业务不变量的载体，不要省略；PII 与事实分离（ASP A-1/A-11）。

## 核心表

| 表 | 关键字段 | 关键约束 |
| --- | --- | --- |
| users | user_ref、display_name、status、created_at | `status in (active, disabled, deleted)`；**无**登录标识 |
| user_identities | user_id、provider、provider_subject、email/phone、verified | `unique(provider, provider_subject)` |
| password_credentials | user_id、password_hash、changed_at | 每用户至多一条活跃凭据；hash 为 Argon2id 串 |
| sessions | user_id、token_hash、created_at、expires_at、revoked_at、last_seen_at、device_meta | `unique(token_hash)`；索引 `(user_id, revoked_at)` |
| verification_codes | purpose、principal_ref、code_hash、expires_at、attempts、consumed_at | `purpose in (email_verify, phone_verify, password_reset)`；短 TTL |
| login_attempts | principal_ref?、provider_subject_masked、success、reason、ip_masked、at | 只追加，审计事实 |

## P2 扩展表（按档位加）

| 表 | 关键字段 | 关键约束 |
| --- | --- | --- |
| totp_secrets | user_id、secret_encrypted、verified_at | 加密存储；未验证不生效 |
| totp_backup_codes | user_id、code_hash、used_at | 一次性 |
| oidc_federations | user_id、issuer、subject | `unique(issuer, subject)` |

## 与会员系统（MSP）的衔接

- MSP 的 `users` 表退化为"业务主体档案"，登录标识与凭据移入本域；
- 两个域通过 `principal`（= user_ref）关联：MSP 订单/权益/账本只存 principal 引用；
- 账户合并（MSP OP-M2）时：身份域做 Identity 迁移（重绑 user_ref），权益/积分仍按 MSP 规则走段并集与账本 transfer。

## 反模式

- 在 users 表加 `email UNIQUE + password_hash`（身份与凭据焊死，多登录方式无解）；
- 明文或可逆加密存口令/验证码/token；把完整 IP 长期存审计表（只留掩码）。
