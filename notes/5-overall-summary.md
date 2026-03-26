# Altius L1 隐私合规研究 — 总结报告

> **TL;DR**
>
> 本研究从竞品分析、转账/支付合规、数据隐私合规、合规覆盖对比、地区速查五个维度，系统性地调研了区块链支付平台在全球主要市场面临的隐私合规要求，并将其翻译为 Altius L1 的具体产品需求。
>
> **核心结论：**
>
> 1. **竞品均有明显短板**：Canton 有最强的协议层隐私但无合规认证、无 ZKP、不兼容 EVM；Kaleido 有 SOC 2 + ISO 认证但隐私停留在应用层。两者都没有 ISO 27701、没有原生 Travel Rule 支持、没有内置制裁名单筛查。
>
> 2. **FATF Travel Rule 是最硬性的技术要求**：所有转账必须携带发起人/受益人姓名+账号；超过 USD/EUR 1,000 须附加地址、出生日期（自然人）或 LEI/BIC（法人）。缺少必要信息时发起机构不得执行转账。这直接决定了 Altius Token Standard 必须内置这些字段，Transfer Hook 必须做前置验证。
>
> 3. **"链上零 PII"是不可妥协的架构原则**：GDPR 删除权（Art.17）、CCPA 泄露赔偿（$750/事件）、新加坡 PDPA 保留限制三部法律共同要求 PII 可被删除。但 GDPR Art.17(3)(b) 的"法律义务"例外为 AML 记录保留 5 年提供了合法空间。解决方案：链上只存哈希引用，PII 全部链下可删除。
>
> 4. **一国一链是天然的合规优势**：GDPR 跨境传输限制（Art.44-49）、中国 PIPL 数据本地化、新加坡 PDPA S.26 等问题在全球化区块链上都是难题，但 Altius 的一国一链模型从架构层面直接解决 — 每条链独立部署在对应国家，PII 不跨境。
>
> 5. **Altius 的差异化定位**：唯一同时覆盖合规工具（Transfer Hooks + Dashboard）和隐私技术（ZK/TEE/Viewing Key）的 EVM 兼容平台。三个销售卖点：合规开箱即用、一国一链 = 天然合规、隐私 + 合规在 EVM 上的首次结合。
>
> 6. **EU PSD2 提供了新的 ZKP 用例**：Art.65 要求资金确认只能回答"是/否"而不透露余额 — 这是 ZK 证明的完美应用场景。Art.63 的小额支付豁免（≤ EUR 30）允许 Transfer Hook 分层处理。
>
> 7. **ISO 20022 全球迁移窗口**：到 2025 年底 11,000+ 银行完成迁移。Altius Token Standard 天然对齐 ISO 20022（Debtor=Originator, RmtInf=Memo），可定位为"已经 ISO 20022 ready 的区块链"。
>
> **项目**：Tabula — 机构隐私合规研究
>
> **日期**：2026-03-26

---

## 研究文档索引

| # | 文档 | 核心内容 | 飞书链接 |
|---|---|---|---|
| 0 | **竞品深度分析：Kaleido vs Canton** | Canton 和 Kaleido 的隐私架构、合规认证、机构采用情况对比；ISO 27701 对标；Gap 分析；Altius 五个差异化方向 | [查看文档](https://fg9w9yu3odc.sg.larksuite.com/docx/W8bSdmP5oo7ZLxxHY0Pl3LnHgNM) |
| 1 | **合规规则映射表（转账/支付）** | 从 FATF Travel Rule、OFAC 制裁框架、ISO 20022 中提取 15 条规则 → Token Standard 字段定义 + Transfer Hook 检查逻辑 | [查看文档](https://fg9w9yu3odc.sg.larksuite.com/docx/NZGZdhMxuoWJ4kxmJ9blnbvPgig) |
| 2 | **隐私合规规则映射表（数据隐私）** | 从 GDPR、CCPA/CPRA、新加坡 PDPA 中提取 18 条规则 → "链上零 PII"架构原则 + Opt-in Privacy 法律正当性 + 数据本地化汇总 | [查看文档](https://fg9w9yu3odc.sg.larksuite.com/docx/EJpUdOb0coGcfRxcRfRlzAdlg5d) |
| 3 | **合规覆盖矩阵：Altius vs 竞品** | 20 条关键合规要求的三方对比（Canton 4项覆盖/Kaleido 3项覆盖/Altius 规划全覆盖）；三个销售卖点；PPT 视觉化方案 | [查看文档](https://fg9w9yu3odc.sg.larksuite.com/docx/CA1LduXchoATQ2xLRTzl6md9gCc) |
| 4 | **地区合规速查卡（EU/US/SG）** | 三个重点地区各一页速查卡；新增 PSD2 条款（资金确认 ZKP 用例、小额支付豁免）；ISO 20022 全球迁移窗口的市场定位意义 | [查看文档](https://fg9w9yu3odc.sg.larksuite.com/docx/K9XAdhYuUoghaXx0bt7lKfYOgOh) |

---

## 一、术语速查表

### 法规与标准

| 缩写 | 全称 | 中文 | 来源 |
|---|---|---|---|
| **GDPR** | General Data Protection Regulation | 通用数据保护条例 | 欧盟，2018 年生效 |
| **CCPA** | California Consumer Privacy Act | 加州消费者隐私法 | 美国加州，2020 年生效 |
| **CPRA** | California Privacy Rights Act | 加州隐私权利法 | 美国加州，2023 年生效（CCPA 修正案） |
| **PDPA** | Personal Data Protection Act | 个人数据保护法 | 新加坡，2014 年生效（2021 修订） |
| **PIPL** | Personal Information Protection Law | 个人信息保护法 | 中国，2021 年生效 |
| **PSD2** | Payment Services Directive 2 | 第二支付服务指令 | 欧盟，2018 年生效 |
| **MiCA** | Markets in Crypto-Assets Regulation | 加密资产市场法规 | 欧盟，2024 年生效 |
| **PSA** | Payment Services Act | 支付服务法 | 新加坡，2019 年生效 |
| **BSA** | Bank Secrecy Act | 银行保密法 | 美国联邦 |
| **FATF** | Financial Action Task Force | 金融行动特别工作组 | 国际组织（OECD 下属） |
| **ISO 20022** | — | 金融消息标准 | ISO（国际标准化组织） |
| **ISO 27701** | — | 隐私信息管理系统 | ISO |

### 合规术语

| 缩写 | 全称 | 中文 | 说明 |
|---|---|---|---|
| **PII** | Personally Identifiable Information | 个人可识别信息 | 可直接或间接识别到自然人的信息 |
| **KYC** | Know Your Customer | 了解你的客户 | 客户身份验证流程 |
| **CDD** | Customer Due Diligence | 客户尽职调查 | KYC 的正式合规术语 |
| **AML** | Anti-Money Laundering | 反洗钱 | — |
| **CFT** | Combating the Financing of Terrorism | 反恐融资 | — |
| **STR/SAR** | Suspicious Transaction/Activity Report | 可疑交易/活动报告 | 向金融情报单位（FIU）提交的报告 |
| **DPO** | Data Protection Officer | 数据保护官 | 组织内负责隐私合规的专职人员 |
| **DPIA** | Data Protection Impact Assessment | 数据保护影响评估 | 高风险数据处理的预评估 |
| **SCCs** | Standard Contractual Clauses | 标准合同条款 | EU 跨境传输的合同保障机制 |
| **BCRs** | Binding Corporate Rules | 约束性公司规则 | 跨国企业集团的内部数据传输框架 |
| **OFAC** | Office of Foreign Assets Control | 外国资产控制办公室 | 美国财政部下属，管理制裁名单 |
| **SDN List** | Specially Designated Nationals List | 特别制裁名单 | OFAC 维护的被制裁实体名单 |
| **LEI** | Legal Entity Identifier | 法人实体标识符 | 全球唯一的法人标识，ISO 17442 标准 |
| **BIC** | Business Identifier Code | 商业标识代码 | 即 SWIFT 代码，ISO 9362 标准 |
| **VASP** | Virtual Asset Service Provider | 虚拟资产服务提供商 | FATF 定义的虚拟资产相关业务实体 |

### 技术术语

| 术语 | 全称 | 说明 |
|---|---|---|
| **ZKP/ZK** | Zero-Knowledge Proof | 零知识证明：在不暴露数据的前提下证明某个陈述为真 |
| **TEE** | Trusted Execution Environment | 可信执行环境：在硬件安全飞地中执行安全计算 |
| **FHE** | Fully Homomorphic Encryption | 全同态加密：在加密数据上直接计算 |
| **MPC** | Multi-Party Computation | 安全多方计算：多方联合计算，互不共享输入 |
| **Viewing Key** | — | 查看密钥：允许特定方（如监管者）查看加密交易细节的密钥 |
| **Transfer Hook** | — | 转账钩子：在转账执行前/后触发的合规检查逻辑（Altius 架构概念） |

### Altius 架构术语

| 术语 | 说明 |
|---|---|
| **Middleware Layer** | Altius L1 的中间件层，包含 Compliance Standards、Opt-in Privacy、Transfer Hooks、Native Stablecoin as Gas 四个模块 |
| **Transfer Hooks** | 转账前置检查：黑名单筛查、金额阈值检查、信息完整性验证、冻结检查、限额检查 |
| **Compliance Standards** | 合规标准模块：Token Standard 内置 ISO 20022 对齐字段、Travel Rule 字段、Memo 字段 |
| **Opt-in Privacy** | 可选隐私模块：ZK、TEE、FHE、Viewing Key |
| **Enterprise Layer** | 企业应用层：区块浏览器、合规 Dashboard、Custody API |
| **一国一链** | Altius 的部署模型：每个国家/地区部署独立的链，天然满足数据本地化和跨境传输合规 |

---

## 二、关键数字速查

| 数字 | 含义 | 来源 |
|---|---|---|
| **USD/EUR 1,000** | FATF Travel Rule 的 de minimis 阈值：超过此金额的转账须携带完整发起人/受益人信息 | FATF INR.16 §8 |
| **EUR 30 / EUR 150** | EU PSD2 小额支付豁免阈值：单笔 ≤30 或累计 ≤150 可豁免部分合规检查 | PSD2 Art.63 |
| **5 年** | 交易记录最低保留期限 | FATF R.11, BSA |
| **72 小时** | GDPR 数据泄露通知时限（通知监管机构） | GDPR Art.33 |
| **3 天** | 新加坡 PDPA 数据泄露通知时限（通知 PDPC） | PDPA |
| **45 天** | CCPA 消费者请求响应时限（可延长 45 天） | CCPA §1798.105 |
| **$750/事件** | CCPA 未加密 PII 泄露的法定赔偿金额 | CCPA §1798.150 |
| **4% 或 2000 万欧元** | GDPR 最高罚款（取较高者） | GDPR Art.83 |
| **11,000+** | 到 2025 年底完成 ISO 20022 迁移的银行数量 | ISO 20022 for Dummies Ch.2 |
| **200+** | Canton Network 的生态参与方数量 | Canton 官网 |
| **33 条** | 本研究提取的合规规则总数（15 条转账/支付 + 18 条数据隐私） | 文档 1 + 文档 2 |

---

## 三、Altius 的三个销售卖点（总结）

### 卖点 1：合规开箱即用

Canton 和 Kaleido 都要求客户自己建设合规基础设施。Altius 是唯一一个把 Travel Rule、制裁名单筛查、交易监控、审计日志**内置在链的 Middleware 层**的企业级区块链。客户只需要配置参数 — 阈值、黑名单来源、保留期限 — 而不需要从零开发。

### 卖点 2：一国一链 = 天然合规

GDPR 的跨境传输限制、中国 PIPL 的数据本地化、新加坡 PDPA 的等同保护要求 — 这些在全球化区块链上都是难题。Altius 的一国一链模型**从架构层面解决**了这些问题：每条链独立部署在对应国家，PII 不跨境，跨链交互只传哈希引用。

### 卖点 3：隐私 + 合规 = EVM 上的首次结合

Canton 有协议级隐私但不兼容 EVM。Kaleido 兼容 EVM 但隐私是应用层的。Altius 是**第一个在 EVM 兼容链上提供内置隐私（ZK/TEE/Viewing Key）和内置合规（Transfer Hooks）的平台**。PSD2 Art.65 的资金确认（只能回答"是/否"不透露余额）就是 ZKP 的杀手级用例。

---

## 四、建议的下一步行动

| 优先级 | 行动 | 说明 |
|---|---|---|
| P0 | **Token Standard 字段定义** | 基于文档 1 的 Transfer struct，正式定义 Altius Token Standard 中的 Travel Rule + ISO 20022 字段 |
| P0 | **Transfer Hook 规则引擎设计** | 基于 33 条规则，设计可配置的规则引擎（阈值检查、黑名单筛查、信息完整性验证、冻结检查） |
| P1 | **启动 ISO 27701 认证路径** | 两个竞品都没有，这是最直接的差异化认证 |
| P1 | **Dashboard 原型设计** | 合规配置界面：黑白名单管理、阈值设置、审计日志、STR 生成、数据主体请求管理 |
| P2 | **跨链合规协议设计** | 一国一链模型下，跨链交互如何只传哈希引用而不传 PII |

---

## 五、参考资料汇总

### 国际标准与组织

| 来源 | URL |
|---|---|
| FATF Recommendations 2012 (Updated Oct 2025) | https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Fatf-recommendations.html |
| FATF VA/VASP Guidance 2021 | https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Guidance-rba-virtual-assets-2021.html |
| ISO 20022 消息目录 | https://www.iso20022.org/catalogue-messages |
| ISO 20022 SWIFT 迁移计划 | https://www.swift.com/standards/iso-20022/get-ready-iso-20022-cbpr |
| ISO 20022 for Dummies (SWIFT 6th Ed) | 本地文件 context/ISO20022.pdf |
| ISO/IEC 27701:2025 | 本地文件 context/ISO-IEC-27701-2025.pdf |

### 欧盟法规

| 来源 | URL |
|---|---|
| GDPR 全文（按条款浏览） | https://gdpr-info.eu/ |
| EU PSD2 Directive 2015/2366 全文 | https://eur-lex.europa.eu/legal-content/EN/TXT/PDF/?uri=CELEX:32015L2366 |
| EU 合并制裁名单 | https://www.consilium.europa.eu/en/policies/sanctions/consolidated-list/ |

### 美国法规

| 来源 | URL |
|---|---|
| CCPA/CPRA (加州司法部) | https://oag.ca.gov/privacy/ccpa |
| OFAC Compliance Framework | https://ofac.treasury.gov/media/16331/download?inline |
| OFAC SDN List 数据下载 | https://ofac.treasury.gov/specially-designated-nationals-and-blocked-persons-list-sdn-human-readable-lists |

### 新加坡法规

| 来源 | URL |
|---|---|
| PDPA 法律原文 | https://sso.agc.gov.sg/Act/PDPA2012 |
| PSA 2019 法律原文 | https://sso.agc.gov.sg/Acts-Supp/2-2019/Published/20190220?DocDate=20190220 |
| MAS DPT 服务商监管通知 | https://www.mas.gov.sg/regulation/notices/psn02-aml-cft |
| PDPC 官网 | https://www.pdpc.gov.sg/overview-of-pdpa/the-legislation/personal-data-protection-act |

### 竞品相关

| 来源 | URL |
|---|---|
| Canton Network 官网 | https://www.canton.network/ |
| Canton Network 技术文档 | https://docs.digitalasset.com/build/3.4/ |
| Canton 生态系统参与者 | https://www.cantonecosystem.com/ |
| Kaleido 官网 | https://www.kaleido.io/ |
| Kaleido 技术文档 | https://docs.kaleido.io/baas/ |
| Kaleido 安全与认证 | https://docs.kaleido.io/baas/kaleido-platform/security/ |
| Paladin (EVM 可编程隐私) | https://www.kaleido.io/blockchain-platform/paladin |
| RootData — Kaleido | https://www.rootdata.com/Projects/detail/Kaleido?k=MjE1Mg== |
| RootData — Canton Network | https://www.rootdata.com/Projects/detail/Canton%20Network?k=NzgyOA== |

---

*创建日期：2026-03-26*
*项目：Tabula — 机构隐私合规研究*
