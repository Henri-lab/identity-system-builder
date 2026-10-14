---
name: identity-system-builder
description: 为「用户认证」设计并生成可直接执行的身份系统构建技能：按目的与规模选档（内嵌最小认证 → 单产品完整认证 → 多产品共享身份 → 统一身份服务），覆盖注册登录、服务端会话、验证码、密码重置、封禁注销、限流审计，P2 含 TOTP 与 OIDC 联合登录；也用于对既有认证实现做 ASP 协议符合性审计或安全评审。不适用于机器身份（API key / 私钥 / SSH 运维接入）或只接现成托管认证（Auth0/Clerk/Supabase Auth）的小改动。
---

# 身份认证系统构建器

## 产出物

一次运行产出两份东西：

1. 一份**选型结论**（档位 + 理由，写进产出技能的「分支决策」章节）；
2. 一份针对该项目的**构建技能目录**（`SKILL.md` + `references/`），任何 agent 读它就能从零实现、按里程碑验收。

不产出业务代码本身。认证属于安全敏感基础设施：**只用成熟密码学与协议实现（Argon2id、RFC 6238、OIDC），绝不自研**。

## 第一步：选型访谈 → 方案档位（必做，禁止静默默认）

按 [references/solution-tiers.md](references/solution-tiers.md) 一次问清三件事——**目的**（验证想法 / 自用经营 / 商业产品 / 多产品基础设施）、
**规模**（用户量级、产品数、团队）、**硬约束**（合规、技术栈、是否已有账号库）——然后选档并写清理由。
用户未回答的项按最保守假设处理，并在产出中显式标注"未确认"。

| 档位 | 目的 × 规模 | 典型方案 |
| --- | --- | --- |
| I0 内嵌最小 | 单产品验证，<100 用户 | `password` + 服务端会话 Cookie + 邮箱验证 |
| I1 单产品完整 | 单产品经营，<1 万用户 | + 手机验证、封禁注销、限流审计、可选 TOTP |
| I2 多产品共享身份 | 多产品复用同一账号 | 共享账号库或 OIDC SSO + RBAC 分离 |
| I3 统一身份服务 | 多租户/高安全 | 自建 identity 服务或接 Keycloak/Auth0/Clerk |

> 档位判据、换档触发器与理由模板见 solution-tiers.md；下表是分支取值全集，落档后逐项确认。

| 分支 | 取值 | 说明 |
| --- | --- | --- |
| 凭据方式 | `password` / `phone_code` / `oauth_oidc` / `password+oauth` | I0=password |
| 会话形态 | `server_cookie` / `jwt_short_lived` | 默认 server_cookie，JWT 只在多客户端场景评估 |
| 验证渠道 | `email` / `phone` / `both` | 重置密码必须经验证渠道 |
| MFA | `none` / `totp`（含备份码） | I1 起可选 |
| 联合登录 | `none` / `oidc` | I2 起 |
| 多产品单点 | `none` / `oidc_sso` / `shared_account_db` | I3 起 |
| 权限模型 | `none` / `rbac`（P3 后置） | 与业务权益分离，判权在业务层 |
| 账号生命周期 | `basic`（禁用/注销）/ `full`（+删除请求） | full 对接 data-retention |

## 第二步：按分支读取参考

**始终读取：**

- [references/solution-tiers.md](references/solution-tiers.md) —— **选型访谈与档位矩阵 I0~I3**（第一步必读）
- [references/asp/README.md](references/asp/README.md) —— **ASP 协议总览**（先读它决定深入哪一篇）
- [references/asp/spec.md](references/asp/spec.md) —— 对象/状态机/操作/不变量 A-1~A-12
- [references/asp/conformance.md](references/asp/conformance.md) —— 等级、MUST 清单、S-01~S-20 与设计判据 D-1~D-5
- [references/asp/events.md](references/asp/events.md) —— 事件信封与必需事件
- [references/asp/prior-art.md](references/asp/prior-art.md) —— **成熟先例映射**（OWASP / NIST 800-63B / RFC 6238 / OIDC / Auth0 / Keycloak）
- [references/data-model.md](references/data-model.md) —— 身份域表目录（身份/凭据/会话分离）
- [references/lessons-log.md](references/lessons-log.md) —— 经验账本（使用前扫最近条目）
- [references/maintenance.md](references/maintenance.md) —— 技能维护手册

只读与当前档位相关的部分，不要把全部内容塞进产出技能。

## 第三步：生成构建技能

用 [assets/generated-skill-template/SKILL.md.tmpl](assets/generated-skill-template/SKILL.md.tmpl) 作骨架，产出目标项目的技能目录。
生成内容必须包含：

1. **目标与非目标**：明确"不做机器身份/SSH、不做权限引擎（P3 前）"；
2. **分支决策表**：访谈结论、档位与理由（含为什么不是相邻档位）、未确认假设；
3. **数据模型**：users / user_identities / password_credentials / sessions / verification_codes（按档位裁剪）；
4. **认证链路**：注册 → 验证 → 登录 → 会话 → 重置/封禁/注销的完整流程与安全属性（Cookie 属性、限流、轮换）；
5. **里程碑 + 验收**：对齐 ASP 等级与 conformance 的 S 向量 + D 判据；每阶段以真实数据库端到端冒烟验收；
6. **踩坑清单**：从 `references/lessons-log.md` 挑与本档位相关的条目；
7. **运维手册**：密钥管理、备份、会话撤销入口、锁定排查；
8. **与业务系统的边界**：只发 `principal`；MSP 消费 principal，权益判权在 MSP。

## 不可妥协项（任何产出都必须写入）

1. **身份与凭据分离**：User 不存登录标识；`(provider, provider_subject)` 唯一（A-1）。
2. **口令 Argon2id**：明文永不落库/日志/缓存；算法参数随哈希串（A-2）。
3. **会话可撤销**：token 随机、服务端只存哈希；登录轮换、改密码/封禁/强制下线即撤销（A-3/4）。
4. **防枚举**：统一错误 + 恒定耗时 + 限速锁定；成败皆审计（A-5）。
5. **验证码一次性**：绑定目的、短 TTL、限次、消费即失效（A-6）。
6. **密码重置经验证渠道**，成功后撤销其它会话（A-7）。
7. **三分离**：认证层只发 `principal`，不代判权限；权益判权在 MSP 的 Entitlement（A-8）。
8. **密钥不进仓库**；时钟以服务端 UTC 为准（A-9）。
9. **审计事实可留、个人标识可删**：注销走掩码/匿名化，不破坏审计链（A-10/11 + data-retention）。
10. **机器身份不混入用户登录**：API key/mTLS/私钥/SSH 属 ServiceIdentity（v0.2+）或主机安全，另立模块。
11. **声明 ASP 符合性等级**：MUST 与 D-1~D-5 全过才算可用；偏差必须记录。

## 必须显式禁止的反面模式

- 把 JWT 裸存当"不可撤销会话"；把邮箱/手机直接当 User 主键；
- 登录错误区分"用户不存在"与"密码错"；
- 验证码永不过期或可复用；密码重置不撤销其它会话；
- 前端隐藏按钮代替服务端校验；凭据/密钥进仓库或日志；
- 认证层直接判"是否会员"（权益判权必须在 MSP）；
- 自研哈希算法、KDF 或 OIDC 协议；
- 在核心代码写产品专属分支/switch（用注册表与中间件扩展点）。

## 完成后

- 用 skill-creator 的 `scripts/quick_validate.py` 校验产出技能目录；
- 条件允许时用独立 subagent 前向测试：只给产出技能 + 一句真实需求，看它能否直接开工；
- 把本轮新经验按 [references/lessons-log.md](references/lessons-log.md) 的格式追加（唯一进化入口）。

## 持续进化（每次使用都必须执行）

1. **使用前**：读 lessons-log 最近条目；与 MFA/联合登录相关时对照 `asp/prior-art.md`；
2. **使用中/后**：新经验按"日期｜来源｜分支 / 结论 / 为什么＋落地"追加，注明真实项目或推理；
3. **提升与归档**：同主题 ≥2 条或触及不可妥协项 → 提升进 `asp/spec.md` / `data-model.md` / `pitfalls`；
   被证伪的标注 `[deprecated]` 保留溯源，不删除。
