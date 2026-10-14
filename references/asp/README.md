# ASP：认证与会话协议（Authentication & Session Protocol）

> 版本：v0.1.0（2026-10-14）｜性质：**规范性协议**（normative），与语言/框架/中间件无关。
> 作者：henrifox｜许可：MIT（见仓库 LICENSE）。引用请注明：ASP v0.1.0 + 作者 + 仓库地址。

## 这是什么

把「用户认证」的**行为契约**抽成一组可验证的规则：注册/登录/会话/验证/密码重置/账号生命周期，
以及 P2 的 MFA 与联合登录。它只回答「你是谁、你怎么证明、你的会话是否有效」——
**不回答「你能看什么」**（那是业务授权，见三分离）。

## 三分离（先于一切）

| 层 | 问题 | 归属 |
| --- | --- | --- |
| 认证 Authentication | 你是谁 | ASP（本协议） |
| 授权 Authorization | 你能做什么（角色/权限） | 业务层（RBAC），P3 再建权限引擎 |
| 权益 Entitlement | 你买没买 | 会员协议 MSP 的 Entitlement 是唯一判权源 |

ASP 的产物只有一个：**`principal`（不透明引用）**。下游系统消费它，不自造它。

## 范围边界

- **在范围内（v0.1）**：人类身份——注册/登录、密码凭据（Argon2id）、服务端会话、邮箱/手机验证、
  密码重置、封禁注销、限流与审计。
- **P2 可选**：TOTP/备份码（MFA）、OIDC 联合登录。
- **明确不做（v0.2+ 候选）**：机器身份（API key / mTLS / 私钥登录）、SSH 运维接入、
  Passkey/WebAuthn、多租户隔离、企业 SSO。私钥/SSH 属于「机器与运维身份」，不是终端用户登录——
  普通 SaaS 先用 API key 解决服务间调用，SSH 归主机安全（server-hardening 技能）。

## 文件

| 文件 | 内容 | 何时读 |
| --- | --- | --- |
| [spec.md](spec.md) | 对象/状态机/操作/不变量 | 实现前必读 |
| [events.md](events.md) | 事件信封与必需事件 | 设计审计/埋点时 |
| [conformance.md](conformance.md) | 等级、MUST 清单、S-01~S-20 与设计判据 D-1~D-5 | 验收时必读 |
| [prior-art.md](prior-art.md) | OWASP/NIST/Auth0/Keycloak/Clerk 映射 | 设计决策时 |

## 用法（三步）

1. 按 `solution-tiers.md` 访谈（目的 × 规模 × 硬约束）选档 I0~I3；
2. 按 spec 实现；语言与存储自由；
3. 跑 conformance 的 S 向量 + D 判据，产出符合性报告；MUST 全过才算可用。
