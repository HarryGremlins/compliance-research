# 地区合规速查卡（产出三：EU / US / SG）

> **TL;DR**：三张一页纸速查卡，分别覆盖 EU、US、SG 三个重点地区。每张卡包含：核心法规、对区块链支付产品的具体技术要求、Altius 如何满足、特殊注意事项。新增内容：(1) EU PSD2 的支付同意/撤回机制、小额支付豁免、资金确认的"是/否"回答（ZKP 用例）、第三方支付发起服务的数据限制；(2) ISO 20022 全球迁移窗口（2025 年底 11,000+ 银行完成迁移）对 Altius 的市场定位意义。
>
> **用途**：快速参考 + Tony 销售材料的补充内容
>
> **日期**：2026-03-25

---

## 速查卡一：欧盟（EU）

### 核心法规

| 法规 | 一句话说明 |
|---|---|
| **GDPR** | 全球最严个人数据保护法，违规最高罚全球年收入 4% |
| **PSD2** (Directive 2015/2366) | 欧盟支付服务指令，规范支付服务提供商的义务和用户权利 |
| **MiCA** (Regulation 2023/1114) | 欧盟加密资产市场法规，2024 年生效 |
| **FATF Travel Rule** | 通过欧盟 Funds Transfer Regulation 在欧盟落地 |
| **EU Sanctions Regime** | 欧盟理事会制裁条例，须筛查合并制裁名单 |

### 对区块链支付产品的具体技术要求

| # | 要求 | 法规来源 | 对 Altius 的技术要求 | 对应模块 |
|---|---|---|---|---|
| 1 | 转账须携带发起人/受益人完整信息（超过 EUR 1,000） | FATF R.16 / EU FTR | Token Standard 内置 Travel Rule 字段 | Compliance Standards |
| 2 | PII 可被删除（被遗忘权） | GDPR Art.17 | 链上零 PII；链下可删除 | 架构设计 |
| 3 | Privacy-by-Design + 默认隐私 | GDPR Art.25 | Opt-in Privacy 作为架构级模块 | Opt-in Privacy |
| 4 | 跨境传输须有充分性认定或 SCCs | GDPR Art.44-49 | 一国一链：EU 链数据留在 EU/EEA | 架构设计 |
| 5 | 支付须经付款人同意；同意可随时撤回 | PSD2 Art.64 | Transfer Hook：授权验证 + 撤回机制 | Transfer Hooks |
| 6 | 资金确认只能回答"是/否"，不得透露余额 | PSD2 Art.65 | **ZKP 完美用例**：证明余额足够但不暴露金额 | Opt-in Privacy |
| 7 | 第三方支付发起服务不得存储敏感数据、不得请求超出必要范围的数据 | PSD2 Art.66 | Transfer Hook：数据最小化检查；链下敏感数据不持久化 | Transfer Hooks |
| 8 | 小额支付豁免（单笔 ≤ EUR 30 或累计 ≤ EUR 150） | PSD2 Art.63 | Transfer Hook：小额支付可跳过部分合规检查 | Transfer Hooks |
| 9 | 支付工具可设定支出限额 | PSD2 Art.68 | Transfer Hook：可配置的单笔/日累计限额 | Transfer Hooks |
| 10 | 自动化决策须提供人工干预渠道 | GDPR Art.22 | Dashboard：被拦截交易的申诉入口 | Dashboard |
| 11 | 数据泄露须 72 小时内通知监管 | GDPR Art.33 | Dashboard：泄露检测 + 通知流程 | Dashboard |
| 12 | EU 制裁名单实时筛查 | EU Council Regulations | Transfer Hook：EU 制裁名单筛查 | Transfer Hooks |

### Altius 在 EU 市场的独特优势

- **一国一链**天然满足 GDPR 跨境传输要求 — 无需 SCCs
- **PSD2 Art.65 的资金确认**是 ZKP 的杀手级用例 — Altius 的 Opt-in Privacy 模块可用 ZK 证明"余额足够"而不暴露具体金额，Canton 和 Kaleido 都做不到
- **小额支付豁免**意味着 Transfer Hook 可以分层 — 低于 EUR 30 的交易走快速通道

### 特殊注意事项

- EU 正在推进 **PSD3**（预计 2025-2026），可能引入更严格的强客户认证（SCA）要求和开放银行规则
- **MiCA** 对稳定币发行商有额外要求（储备金、白皮书披露等），如果 Altius 链上的稳定币由银行发行，需确认是否触发 MiCA 义务
- **eIDAS 2.0** 推动欧盟数字身份钱包，可能成为 KYC 的新基础设施

---

## 速查卡二：美国（US）

### 核心法规

| 法规 | 一句话说明 |
|---|---|
| **CCPA/CPRA** | 加州隐私法，美国目前最严的州级隐私法规 |
| **BSA/AML** | 银行保密法 + 反洗钱法规，FinCEN 执行 |
| **OFAC Sanctions** | 美国财政部制裁合规，全球涉及美元的交易均受管辖 |
| **FATF Travel Rule** | 通过 FinCEN 的 31 CFR 1010.410 在美国落地 |
| **State MTL** | 各州货币传输牌照，要求因州而异 |

### 对区块链支付产品的具体技术要求

| # | 要求 | 法规来源 | 对 Altius 的技术要求 | 对应模块 |
|---|---|---|---|---|
| 1 | 转账须携带发起人/受益人信息 | FATF R.16 / FinCEN 31 CFR 1010.410 | Token Standard 内置 Travel Rule 字段 | Compliance Standards |
| 2 | OFAC SDN/SSI 名单实时筛查 + 模糊匹配 | OFAC Framework | Transfer Hook：实时筛查 + 模糊匹配算法 | Transfer Hooks |
| 3 | 制裁名单自动同步更新 | OFAC Framework p.5 | Dashboard：自动化名单同步 | Dashboard |
| 4 | 可疑交易报告（SAR/STR）| BSA, FinCEN | Dashboard：交易监控 + SAR 生成 | Dashboard |
| 5 | 交易记录保留 ≥ 5 年 | BSA 31 CFR 1010.430 | Dashboard：审计日志（5 年保留期） | Dashboard |
| 6 | 消费者可要求"停止出售/共享个人信息" | CCPA §1798.120 | Dashboard：Opt-out of Sale 配置 | Dashboard |
| 7 | 敏感个人信息（含金融账户）使用限制 | CPRA §1798.121 | Transfer Hook：敏感信息使用范围限制 | Transfer Hooks |
| 8 | 未加密 PII 泄露可被索赔 $750/事件 | CCPA §1798.150 | 所有链下 PII 必须加密存储 | Opt-in Privacy |
| 9 | 消费者请求须 45 天内响应 | CCPA §1798.105 | Dashboard：数据主体请求 SLA 追踪 | Dashboard |
| 10 | 钱包地址黑名单筛查 | FATF VA Guidance §273 | Transfer Hook：链上地址黑名单 | Transfer Hooks |

### Altius 在 US 市场的独特优势

- **无跨境传输限制** — CCPA 不限制数据跨境，架构设计更灵活
- **OFAC 制裁筛查**是美国市场的硬性要求 — Altius 的 Transfer Hook 原生支持实时筛查 + 模糊匹配，这是 Canton 和 Kaleido 都不内置的
- **CCPA 的"出售"概念**独特 — Altius Dashboard 支持 opt-out 配置，竞品没有

### 特殊注意事项

- 美国**无联邦统一隐私法** — 但多个州正在立法（Colorado, Virginia, Connecticut 等），趋势是向 CCPA 看齐
- **FinCEN 2019 虚拟货币指引**（FIN-2019-G001）明确了哪些活动需要 MSB 牌照 — Altius 链上的稳定币转账大概率触发 MSB 义务
- **纽约 BitLicense** 有额外要求（网络安全计划、资本充足率等）
- 美国的制裁合规是**域外管辖** — 任何涉及美元、美国技术或美国人的交易都受 OFAC 管辖，即使双方都在美国境外

---

## 速查卡三：新加坡（SG）

### 核心法规

| 法规 | 一句话说明 |
|---|---|
| **PDPA** | 新加坡个人数据保护法，11 项数据保护义务 |
| **PSA** (Payment Services Act 2019) | 规范支付服务和数字支付代币（DPT）服务商的牌照和义务 |
| **MAS Notices/Guidelines** | 新加坡金融管理局的技术风险管理指引 |
| **FATF Travel Rule** | 通过 MAS Notice PSN02 在新加坡落地 |

### 对区块链支付产品的具体技术要求

| # | 要求 | 法规来源 | 对 Altius 的技术要求 | 对应模块 |
|---|---|---|---|---|
| 1 | DPT 服务须持牌 | PSA Part 3 | 客户须获得 MAS DPT 牌照 | 非链上需求 |
| 2 | 转账须携带发起人/受益人信息 | FATF R.16 / MAS Notice PSN02 | Token Standard 内置 Travel Rule 字段 | Compliance Standards |
| 3 | 收集/使用/披露个人数据前须获得同意 | PDPA 同意义务 | Transfer Hook：KYC 同意记录 | Transfer Hooks |
| 4 | 跨境传输须确保接收方提供等同保护 | PDPA S.26 | 跨境交易合同中须包含数据保护条款 | Transfer Hooks |
| 5 | 数据泄露 3 天内通知 PDPC | PDPA 泄露通知义务 | Dashboard：泄露检测 + 3 天通知流程 | Dashboard |
| 6 | 须指定数据保护官（DPO）| PDPA 问责义务 | Dashboard：DPO 配置 | Dashboard |
| 7 | 不再需要的数据须删除或匿名化 | PDPA 保留限制义务 | 链下数据自动过期机制 | Dashboard |
| 8 | 须公开数据保护政策 | PDPA 公开义务 | Dashboard：对外隐私政策页面 | Dashboard |
| 9 | 交易监控和可疑交易报告 | MAS Notice PSN02 | Dashboard：交易监控 + STR 生成 | Dashboard |
| 10 | AML/CFT 合规计划 | PSA + MAS Guidelines | Dashboard：合规计划模板 | Dashboard |

### Altius 在 SG 市场的独特优势

- **MAS Project Guardian** 的参与者（如 Kaleido）已经证明了新加坡市场对企业区块链的接受度 — Altius 可对标
- **PDPA S.26 跨境传输**要求等同保护 — 一国一链模型使新加坡链的数据天然留在新加坡
- 新加坡是**亚太金融中心** — 覆盖新加坡 = 对接东南亚市场的桥头堡

### 特殊注意事项

- **PSA 2019** 将 DPT 服务纳入监管 — 经营 DPT 服务须向 MAS 申请牌照（Standard Payment Institution 或 Major Payment Institution）
- **MAS 技术风险管理指引**（TRM Guidelines）对金融机构的 IT 安全有详细要求 — 可能影响链的部署架构
- 新加坡的 **PDPA 罚款上限为 SGD 1,000,000**（或组织年营业额的 10%，取较高者，2020 修订后）— 相对 GDPR 较低，但足以产生合规压力
- **MAS 正在积极推动代币化资产** — Project Guardian（资产代币化）和 Project Orchid（CBDC）是两个关键试点

---

## 附：ISO 20022 全球迁移窗口 — Altius 的市场定位机会

### 迁移时间线

到 2025 年底，全球主要支付基础设施将完成向 ISO 20022 的迁移：
- **美联储** + The Clearing House（美国）
- **英格兰银行**（英国）
- **Eurosystem**（欧元区）
- **HKICL**（香港）
- 全球 **11,000+ 银行**通过 SWIFT 网络

### 对 Altius 的战略意义

1. **市场窗口**：银行正在为 ISO 20022 改造系统 — 这是引入区块链支付基础设施的最佳时机。Altius 的 Token Standard 天然对齐 ISO 20022（Debtor=Originator, Creditor=Beneficiary, RmtInf=Memo），可以作为"已经 ISO 20022 ready 的区块链"来定位。

2. **互操作性卖点**：ISO 20022 是跨标准互操作的基础。Altius 的 Middleware Layer 可以定位为"区块链世界的 ISO 20022 中间件" — 把传统银行系统（MT103）和链上交易（Token Transfer）通过 ISO 20022 的公共数据模型桥接起来。

3. **监管友好**：FATF 建议 16 第 4 段明确提到应按 ISO 20022 结构化转账信息。采用 ISO 20022 对齐的 Token Standard = 天然满足监管期望。

---

*创建日期：2026-03-25*
*信息来源：EU PSD2 Directive 2015/2366 Art.63-72、ISO 20022 for Dummies Ch.2、FATF Recommendations/VA Guidance、GDPR、CCPA/CPRA、新加坡 PDPA/PSA*
*项目：Tabula — 机构隐私合规研究*
