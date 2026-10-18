# 技能维护手册（identity-system-builder）

本技能目标是"每次使用都变准"。维护动作四类：追加、合并、提升、废弃。

## 1. 什么时候写

满足任一条件就在 `lessons-log.md` 追加一条：

- 踩到一个认证/安全新坑（尤其是运行时才暴露的）；
- 一个判断被推翻（例：以为要上 JWT，实际服务端会话就够）；
- 拿到真实规模/攻击数据（爆破频率、验证码成本）；
- 工具/框架/标准变化（Argon2 参数建议、OIDC 规范演进）；
- 用户明确表达的偏好或约束（合规、预算）。

不写：通用编程常识、官方文档能查到的、项目机密（密钥/token/客户数据）。

## 2. 怎么写

```markdown
### L-编号 一句话结论（动词开头）
- 日期：YYYY-MM-DD｜来源：真实项目（哪次事故）/ 推理｜分支：任意
- 结论：<一句话，能直接指导决策>
- 为什么＋落地：<原因 + 落地，3 行以内>
- 外部参考：<OWASP / NIST / RFC 编号等>
```

## 3. 去重与合并 / 4. 提升

- 同主题条目 → 在原条目补充，不新开；
- 结论冲突 → 新条目覆盖旧结论，旧条目标 `[deprecated by L-xxx]`；
- 同主题 ≥2 条或触及不可妥协项 → 提升进 `asp/spec.md` / `data-model.md`，保留日志溯源。

## 5. 同步与版本

**唯一上游：独立开源仓库（identity-system-builder）。** `platform-kit/skills/` 为 vendored 副本，
`~/.codex/skills/` 为安装副本——编辑只在上游，单向同步（上游 → vendored → 安装副本）。

## 6. 变更记录

| 版本 | 日期 | 变更 |
| --- | --- | --- |
| v0.1.0 | 2026-10-14 | 初始版本：ASP v0.1.0（spec/events/conformance/prior-art）+ 数据模型 + 选型访谈 I0~I3 + L-001~L-005 |

## 7. 自检清单

1. frontmatter `name` 与目录名一致、`description` 能区分适用场景；
2. SKILL.md 引用的文件都存在；
3. 没有未完成的待办标记；新增条目格式合规、未含机密；
4. 副本已同步到 `~/.codex/skills/`。
5. 新增原则/概念前查 agent-skills-index 的 `concept-map.md`：已有则引用不复制，新概念先登记权威位置。
6. 决策回填与提升规则见 agent-skills-index 的 `conventions/knowledge-evolution.md`（证据等级：真实反例 > 推理）。
