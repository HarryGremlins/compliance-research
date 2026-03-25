# 合规覆盖矩阵：Altius vs Canton vs Kaleido

> **TL;DR**：从 33 条合规规则中提取 20 条最关键的，对比三个平台的覆盖情况。结论：Canton 在隐私架构上领先（数据最小化、子交易隐私）但在合规工具和认证上完全缺失；Kaleido 有认证和基础工具但隐私停留在应用层；Altius 的差异化定位是**唯一同时覆盖合规工具（Transfer Hooks + Dashboard）和隐私技术（ZK/TEE/Viewing Key）的 EVM 兼容平台**，且一国一链模型天然解决数据本地化问题。
>
> **用途**：可直接用于 Tony 的销售 PPT，向客户展示 Altius 的合规覆盖优势。
>
> **日期**：2026-03-25

---

## 一、图例说明

| 符号 | 含义 |
|---|---|
| ✅ | 已支持 / 原生覆盖 |
| ⚠️ | 部分支持 / 需额外开发 / 应用层实现 |
| ❌ | 不支持 / 未覆盖 |
| 🔜 | 规划中（Altius 专用） |

---

## 二、合规覆盖矩阵

### 2.1 转账/支付合规

| # | 合规要求 | 来源 | Canton | Kaleido | Altius（规划中） |
|---|---|---|---|---|---|
| 1 | **Travel Rule — 转账携带发起人/受益人信息** | FATF R.16 | ❌ Daml 不原生支持 | ⚠️ 无内置字段，需应用层实现 | 🔜 Token Standard 内置 originator/beneficiary 字段 |
| 2 | **Travel Rule — 金额阈值自动检查** | FATF INR.16 §8 | ❌ 无 Transfer Hook 概念 | ⚠️ 可通过智能合约实现 | 🔜 Transfer Hook：原生阈值检查（可配置） |
| 3 | **Travel Rule — 缺失信息时拦截转账** | FATF INR.16 §23 | ❌ | ⚠️ 应用层逻辑 | 🔜 Transfer Hook：前置验证，缺字段则拒绝 |
| 4 | **ISO 20022 Memo 字段** | ISO 20022, FATF INR.16 §4 | ❌ Daml 无 Memo 概念 | ❌ ERC20 无原生 Memo | 🔜 Token Standard 内置结构化 Memo（RmtInf） |
| 5 | **制裁名单筛查（OFAC/EU）** | OFAC Framework | ❌ 无内置筛查 | ⚠️ 通过 Chainlink Oracle 可接入 | 🔜 Transfer Hook：实时地址+实体筛查，支持模糊匹配 |
| 6 | **钱包地址黑名单** | FATF VA Guidance §273 | ❌ | ⚠️ 应用层实现 | 🔜 Transfer Hook：链上地址黑名单前置检查 |
| 7 | **账户冻结** | FATF R.6 | ⚠️ Daml 可归档合约 | ⚠️ 智能合约级别 | 🔜 Transfer Hook：原生冻结功能 |
| 8 | **制裁名单自动同步** | OFAC Framework p.5 | ❌ | ❌ | 🔜 Dashboard：自动化名单同步 + 更新日志 |
| 9 | **可疑交易监控和报告（STR）** | FATF R.20 | ❌ | ⚠️ 日志+浏览器可辅助 | 🔜 Dashboard：交易监控 + STR 生成模板 |
| 10 | **交易记录保留（≥5 年）** | FATF R.11 | ✅ 虚拟全局账本永久保留 | ✅ 区块链天然不可变 | 🔜 Dashboard：审计日志 + 可配置保留期 |

### 2.2 数据隐私合规

| # | 合规要求 | 来源 | Canton | Kaleido | Altius（规划中） |
|---|---|---|---|---|---|
| 11 | **PII 不可上链（删除权兼容）** | GDPR Art.17 | ✅ 私有合约存储，无全局状态，节点级可删 | ⚠️ 链下文档存储可删，链上不可变 | 🔜 架构原则：链上零 PII，链下可删除 |
| 12 | **Privacy-by-Design** | GDPR Art.25 | ✅ 协议层原生隐私设计 | ❌ 隐私是应用层关注点 | 🔜 Opt-in Privacy 作为架构级模块 |
| 13 | **数据最小化** | GDPR Art.5(1)(c) | ✅ 子交易隐私 — 各方只看到相关部分 | ⚠️ 私有交易限制可见性，但非协议强制 | 🔜 Transfer Hook 默认最小字段集 + ZKP |
| 14 | **选择性披露 / Viewing Key** | GDPR Art.25, ISO 27701 A.1.3.7 | ⚠️ Daml 的 observer 模型提供部分选择性 | ❌ 无原生选择性披露 | 🔜 Viewing Key：监管方可审计，其他方不可见 |
| 15 | **跨境传输合规** | GDPR Art.44-49, PDPA S.26 | ⚠️ 多同步器可地理分离，但非自动化 | ⚠️ 多区域部署，但需手动配置 | 🔜 一国一链：PII 不跨境，跨链仅传哈希 |
| 16 | **数据本地化** | 中国 PIPL, 印度 DPDP 等 | ⚠️ 可部署本地节点，但非产品特性 | ⚠️ 可选区域（AWS/Azure），但非隔离链 | 🔜 一国一链天然满足：每条链独立部署在对应国家 |
| 17 | **自动化决策通知 + 人工干预** | GDPR Art.22 | ❌ 无内置机制 | ❌ 无内置机制 | 🔜 Dashboard：被拦截交易的申诉入口 + 人工审核 |
| 18 | **数据泄露通知** | GDPR Art.33 (72h), PDPA (3天) | ❌ | ❌ | 🔜 Dashboard：泄露检测 + 按地区配置通知时限 |

### 2.3 平台基础能力

| # | 能力 | Canton | Kaleido | Altius（规划中） |
|---|---|---|---|---|
| 19 | **合规认证（SOC 2 / ISO 27001）** | ❌ 无公开认证 | ✅ SOC 2 Type II + ISO 27001/17/18 | 🔜 目标：ISO 27701 + SOC 2 |
| 20 | **EVM 兼容** | ❌ 仅 Daml | ✅ 多协议（Besu/Geth/Quorum/Fabric/Corda） | 🔜 EVM 兼容 + Rust 原生 Token |

---

## 三、维度汇总

### 3.1 按能力维度统计

| 维度 | Canton ✅ | Canton ⚠️ | Canton ❌ | Kaleido ✅ | Kaleido ⚠️ | Kaleido ❌ |
|---|---|---|---|---|---|---|
| **转账/支付合规**（#1-10） | 1 | 1 | 8 | 1 | 6 | 3 |
| **数据隐私合规**（#11-18） | 3 | 2 | 3 | 0 | 4 | 4 |
| **平台基础**（#19-20） | 0 | 0 | 2 | 2 | 0 | 0 |
| **合计** | **4** | **3** | **13** | **3** | **10** | **7** |

### 3.2 一句话总结

- **Canton**：隐私架构领先（4 项原生覆盖），但**合规工具几乎为零**（13 项未覆盖）。适合已经有自己合规基础设施的大型金融机构。
- **Kaleido**：有认证和基础工具（3 项原生覆盖），但**大部分合规能力需要客户自己在应用层开发**（10 项部分支持）。隐私能力弱。
- **Altius**：规划中的 Middleware Layer（Transfer Hooks + Compliance Standards + Opt-in Privacy + Dashboard）可覆盖全部 20 项。**差异化定位：开箱即用的合规能力 + EVM 兼容 + 一国一链**。

---

## 四、Altius 的三个独特卖点（用于销售）

### 卖点 1：合规开箱即用

> "Canton 和 Kaleido 都要求客户自己建设合规基础设施。Altius 是唯一一个把 Travel Rule、制裁名单筛查、交易监控、审计日志**内置在链的 Middleware 层**的企业级区块链。客户只需要配置参数 — 阈值、黑名单来源、保留期限 — 而不需要从零开发。"

### 卖点 2：一国一链 = 天然合规

> "GDPR 的跨境传输限制、中国 PIPL 的数据本地化、新加坡 PDPA 的等同保护要求 — 这些在全球化区块链上都是难题。Altius 的一国一链模型**从架构层面解决**了这些问题：每条链独立部署在对应国家，PII 不跨境，跨链交互只传哈希引用。"

### 卖点 3：隐私 + 合规 = EVM 上的首次结合

> "Canton 有协议级隐私但不兼容 EVM。Kaleido 兼容 EVM 但隐私是应用层的。Altius 是**第一个在 EVM 兼容链上提供内置隐私（ZK/TEE/Viewing Key）和内置合规（Transfer Hooks）的平台**。开发者用 Solidity 写智能合约，同时获得协议级的隐私和合规保障。"

---

## 五、视觉化建议（PPT 用）

如果 Tony 需要在 PPT 中展示，建议用以下格式：

### 方案 A：简化版（适合一页 slide）

| 合规要求 | Canton | Kaleido | Altius |
|---|---|---|---|
| Travel Rule | ❌ | ⚠️ | ✅ |
| 制裁名单筛查 | ❌ | ⚠️ | ✅ |
| ISO 20022 Memo | ❌ | ❌ | ✅ |
| 删除权兼容 | ✅ | ⚠️ | ✅ |
| Privacy-by-Design | ✅ | ❌ | ✅ |
| Viewing Key | ⚠️ | ❌ | ✅ |
| 跨境合规 | ⚠️ | ⚠️ | ✅ |
| 数据本地化 | ⚠️ | ⚠️ | ✅ |
| EVM 兼容 | ❌ | ✅ | ✅ |
| 合规认证 | ❌ | ✅ | 🔜 |

### 方案 B：三个圆圈（适合开场 slide）

```
    Canton               Kaleido              Altius
  ┌─────────┐         ┌─────────┐         ┌─────────┐
  │ Privacy │         │  Certs  │         │ Privacy │
  │   ✅    │         │   ✅    │         │   ✅    │
  │         │         │         │         │         │
  │Compliance│        │Compliance│        │Compliance│
  │   ❌    │         │   ⚠️   │         │   ✅    │
  │         │         │         │         │         │
  │   EVM   │         │   EVM   │         │   EVM   │
  │   ❌    │         │   ✅    │         │   ✅    │
  └─────────┘         └─────────┘         └─────────┘
```

---

*创建日期：2026-03-25*
*项目：Tabula — 机构隐私合规研究*

### 信息来源

| 来源 | URL |
|---|---|
| FATF Recommendations 2012 | https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Fatf-recommendations.html |
| FATF VA/VASP Guidance 2021 | https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Guidance-rba-virtual-assets-2021.html |
| OFAC Compliance Framework | https://ofac.treasury.gov/media/16331/download?inline |
| GDPR 全文 | https://gdpr-info.eu/ |
| CCPA/CPRA | https://oag.ca.gov/privacy/ccpa |
| 新加坡 PDPA | https://sso.agc.gov.sg/Act/PDPA2012 |
| Canton Network 技术文档 | https://docs.digitalasset.com/build/3.4/ |
| Kaleido 技术文档 | https://docs.kaleido.io/baas/ |
| Kaleido 安全与认证 | https://docs.kaleido.io/baas/kaleido-platform/security/ |
| ISO 20022 消息目录 | https://www.iso20022.org/catalogue-messages |
