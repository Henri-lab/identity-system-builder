# identity-system-builder 🔐 身份认证系统构建器

> A **Codex skill** that turns "user authentication" into a reviewed, step-by-step,
> verifiable build plan — independent of language and framework. Docs are in Chinese;
> the core protocol (ASP) is tool-neutral.

<p align="center">
  <a href="LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-blue.svg"></a>
  <img alt="ASP v0.1.0" src="https://img.shields.io/badge/protocol-ASP%20v0.1.0-blueviolet">
  <img alt="Conformance" src="https://img.shields.io/badge/conformance-S01--S20_%2B_D1--D5-success">
  <img alt="Version" src="https://img.shields.io/badge/version-v0.1.0-informational">
</p>

把「用户认证」做成一份**可直接执行的构建技能**：任何代码能力达标的 agent 读它就能按档位
（内嵌最小 → 单产品完整 → 多产品共享身份 → 统一身份服务）从零搭出可理解、可维护、可演进的认证系统。

它只回答「你是谁、你怎么证明、你的会话是否有效」；**不回答「你能看什么」**——认证、授权、权益三分离，
权益判权归会员协议 MSP。

## 快速开始

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/Henri-lab/identity-system-builder.git ~/.codex/skills/identity-system-builder
```

之后在任意 Codex 会话中按 skill 名（`identity-system-builder`）触发即可。

## 目录

```text
SKILL.md                 主入口：选型访谈 → 档位 I0~I3 → 按分支读参考 → 生成项目技能
agents/openai.yaml       agent 元数据
assets/                  生成产物模板
references/
├── asp/                 ASP 协议 v0.1.0（规范性）：spec / events / conformance / prior-art
├── data-model.md        身份域表目录（身份/凭据/会话分离）
├── solution-tiers.md    选型访谈与档位矩阵 I0~I3
├── lessons-log.md       经验账本（L-001 起）
└── maintenance.md       技能持续进化纪律
```

## 三个不寻常之处

1. **协议而非模板**：ASP 只规定外部可观察的行为与不变量，用 20 条测试向量（S-01~S-20）
   + 5 条设计判据（D-1~D-5）判定「可用」；
2. **范围诚实**：只做人类身份；机器身份（API key / 私钥 / SSH）是 v0.2+ 的独立模块，不混入用户登录；
3. **安全即验收**：防枚举、防固定、验证码一次性、改密撤会话都不是"建议"，是 MUST 不变量。

## 起源与署名

本技能由 [membership-system-builder](https://github.com/Henri-lab/membership-system-builder) 的认证域独立而来，
2026-10-14 独立为公开仓库。作者清单见 [AUTHORS.md](AUTHORS.md)；二次分发必须保留 [LICENSE](LICENSE) 版权声明。

```text
identity-system-builder（ASP v0.1.0）
作者：henrifox
仓库：https://github.com/Henri-lab/identity-system-builder
首发：2026-10-14｜许可：MIT（保留署名）
```

## License

[MIT](LICENSE) © 2026 henrifox
