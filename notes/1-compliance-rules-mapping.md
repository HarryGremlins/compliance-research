# 隐私合规规则技术映射表（产出一 — 维度一：转账/支付合规）

> **TL;DR**：从 FATF Travel Rule、OFAC 制裁框架、ISO 20022 三大来源中提取了 15 条对 Altius L1 有直接产品影响的合规规则。核心发现：(1) 所有转账必须携带发起人/受益人姓名+账号，超过 1,000 USD/EUR 须附加地址和身份信息 → Token Standard 必须内置这些字段；(2) 制裁名单筛查必须实时、支持模糊匹配、自动更新 → Transfer Hook 前置检查；(3) ISO 20022 的 Memo 字段（RmtInf）是银行间支付的标配 → Token Standard 需原生支持。
>
> **目标**：把转账/支付相关的合规规则翻译成 Altius L1 的具体产品需求。
>
> **日期**：2026-03-18
>
> **信息来源**：FATF Recommendations 2012 (Updated Oct 2025)、FATF VA/VASP Guidance 2021、OFAC Compliance Framework、ISO 20022 for Dummies (SWIFT 6th Ed)

---

## 一、FATF Travel Rule（建议 16）— 转账信息携带要求

### 1.1 核心规则

FATF 建议 16（Payment Transparency）要求所有支付或价值转移必须携带发起人（originator）和受益人（beneficiary）的身份信息，以确保执法机构能追踪资金流向。

**这是对 Altius Token Standard 和 Transfer Hooks 最直接的技术要求。**

### 1.2 跨境转账的信息要求

#### 低于 de minimis 阈值（≤ USD/EUR 1,000）

| 必须携带的字段 | 说明 | Altius Token Standard 字段 |
|---|---|---|
| 发起人姓名 | originator name | `originator_name` |
| 受益人姓名 | beneficiary name | `beneficiary_name` |
| 发起人账号 | 如用账户处理交易；若无账户，则需唯一交易引用号 | `originator_account` 或 `tx_reference` |
| 受益人账号 | 同上 | `beneficiary_account` 或 `tx_reference` |

**注**：低于阈值的信息不需要验证准确性，除非有洗钱/恐融嫌疑。

#### 高于 de minimis 阈值（> USD/EUR 1,000）

| 必须携带的字段 | 说明 | Altius Token Standard 字段 |
|---|---|---|
| 发起人姓名 | originator name | `originator_name` |
| 受益人姓名 | beneficiary name | `beneficiary_name` |
| 发起人账号 | 若无账户，则需唯一交易引用号 | `originator_account` |
| 受益人账号 | 同上 | `beneficiary_account` |
| 发起人地址 | 物理地址；受益人只需国家和城市名 | `originator_address` / `beneficiary_country_town` |
| 发起人出生日期 | 仅自然人；若无完整日期，出生年份即可 | `originator_dob` |
| 法人实体标识 | 若发起人/受益人为法人：BIC、LEI 或唯一官方标识符 | `originator_lei` / `beneficiary_lei` |

### 1.3 境内转账的信息要求

#### 低于 de minimis 阈值（≤ USD/EUR 1,000）

- 发起人姓名
- 发起人账号（或唯一交易引用号）

#### 高于 de minimis 阈值（> USD/EUR 1,000）

- 与跨境转账相同的发起人信息
- 但可以不随交易一起发送，只需在 3 个工作日内应请求提供给受益机构或主管当局

### 1.4 对虚拟资产（VA）转账的特别要求

来源：FATF Recommendations INR.15 第 7(b) 段 + VA/VASP Guidance 第 281-294 段

| 规则 | 具体要求 | Altius 实现 |
|---|---|---|
| **CDD 阈值** | 偶发交易超过 USD/EUR 1,000 须做客户尽职调查 | Transfer Hook：金额阈值检查 |
| **Travel Rule 适用** | VA 转账适用与传统电汇相同的 Travel Rule（建议 16）| Token Standard：转账需携带 originator/beneficiary 信息 |
| **信息提交时机** | 必须**立即**提交，与转账同时或同步进行 | Transfer Hook：转账前验证信息完整性 |
| **信息不必附在交易上** | 信息可以直接或间接提交，不必附加在 VA 转账本身上（脚注 47）| 可通过链下通道传递，链上存引用 |
| **冻结义务** | 必须对被制裁人员/实体执行冻结，禁止交易 | Transfer Hook：制裁名单黑名单检查 |
| **钱包地址筛查** | VASP 应筛查客户和对手方的钱包地址是否在黑名单上（第 273 段）| Transfer Hook：地址黑名单前置检查 |
| **非托管钱包** | 与非托管钱包交易时，VASP 需从自己的客户处获取发起人和受益人信息 | 链上标记或链下 KYC 关联 |
| **技术中立** | FATF 不指定特定技术方案。智能合约、多签、独立消息平台、API 均可（第 282 段）| Altius 可选择最适合的实现方式 |
| **低于阈值的 VA 转账** | 仍需：发起人姓名、受益人姓名、钱包地址、唯一交易引用号（第 294 段）| Token Standard 最小字段集 |

### 1.5 对 Altius 的技术要求总结

**Token Standard 必须支持的字段**：

```
Transfer {
    // Core transfer fields
    amount: uint256,
    token: address,
    from: address,
    to: address,

    // Travel Rule compliance fields (Memo/附加信息)
    originator_name: string,         // Required for all transfers
    beneficiary_name: string,        // Required for all transfers
    originator_account: string,      // Required; or tx_reference
    beneficiary_account: string,     // Required; or tx_reference
    tx_reference: string,            // Unique transaction reference

    // Additional fields for transfers > USD/EUR 1,000
    originator_address: string,      // Physical address
    beneficiary_country_town: string, // Country + town name
    originator_dob: string,          // Date of birth (natural person)
    originator_lei: string,          // LEI/BIC (legal person)
    beneficiary_lei: string,         // LEI/BIC (legal person)
}
```

**Transfer Hooks 必须实现的检查**：

| Hook | 触发时机 | 逻辑 |
|---|---|---|
| **金额阈值检查** | 每笔转账前 | if amount > threshold → 要求完整 Travel Rule 信息 |
| **信息完整性检查** | 每笔转账前 | 验证必填字段是否齐全 |
| **制裁名单检查** | 每笔转账前 | 筛查 from/to 地址是否在黑名单上 |
| **转账限额检查** | 每笔转账前 | 检查是否超过用户的单笔/日累计限额 |
| **冻结检查** | 每笔转账前 | 检查账户是否被冻结 |

---

## 二、制裁名单筛查（OFAC / EU Sanctions）

### 2.1 OFAC 合规框架的五大组成部分

来源：A Framework for OFAC Compliance Commitments (U.S. Department of the Treasury)

| 组成部分 | 描述 | 对 Altius 的影响 |
|---|---|---|
| **管理层承诺** | 高管必须审核和批准合规计划 | Dashboard：合规政策配置和审批流程 |
| **风险评估** | 持续评估客户、产品、地理位置的制裁风险 | Dashboard：风险评估模块 |
| **内部控制** | 制定识别、拦截、上报、记录的政策和流程 | Transfer Hook：自动拦截 + 记录 |
| **测试与审计** | 定期测试合规程序的有效性 | Dashboard：审计日志 + 测试报告 |
| **培训** | 所有相关人员的制裁合规培训 | 非链上需求，属于客户组织层面 |

### 2.2 制裁筛查的技术要求

来源：OFAC Framework 第 5 页 "Internal Controls" + 附录 "Root Causes" 第 VI 节

| 要求 | 具体描述 | Altius 实现 |
|---|---|---|
| **名单同步** | 必须及时更新 SDN List（特别制裁名单）和 SSI List（行业制裁名单）| Transfer Hook：定期从 OFAC 数据源同步黑名单 |
| **筛查覆盖** | 筛查客户、交易对手、中介的姓名和标识符 | Transfer Hook：地址 + 姓名匹配 |
| **模糊匹配** | 必须考虑替代拼写（如 Havana/Habana, Cuba/Kuba）| 筛查算法需支持模糊匹配 |
| **BIC 代码筛查** | 须包含被制裁金融机构的 SWIFT BIC 代码 | Transfer Hook：BIC 黑名单 |
| **实时 + 批量** | 交易筛查应实时进行；客户库可批量定期筛查 | Transfer Hook 实时 + Dashboard 批量 |
| **记录保留** | 所有筛查记录必须保留，供审计使用 | Dashboard：审计日志 |

### 2.3 常见违规根因（OFAC 附录提取）

这些是 OFAC 执法行动中最常见的违规原因，直接指导 Altius 应该避免什么：

1. **缺乏正式合规计划** → Altius Dashboard 必须提供合规计划模板
2. **误解 OFAC 规则的适用范围** → 涉及美国人、美元、美国技术均受管辖
3. **筛查软件未及时更新** → 名单同步机制必须自动化
4. **模糊匹配不足** → 筛查算法不能仅做精确匹配
5. **客户尽职调查不足** → KYC 信息必须完整

---

## 三、ISO 20022 支付消息中的 PII 字段

### 3.1 ISO 20022 消息结构

ISO 20022 支付消息（Customer Credit Transfer = pacs.008）包含以下涉及 PII 的数据组件：

| XML 标签 | 字段含义 | PII 类别 | 对应 FATF Travel Rule |
|---|---|---|---|
| `<Dbtr><Nm>` | 付款人姓名 (Debtor Name) | 姓名 | originator_name |
| `<Dbtr><PstlAdr>` | 付款人地址 (Postal Address) | 地址 | originator_address |
| `<Dbtr><PstlAdr><StrtNm>` | 街道名 | 地址 | originator_address |
| `<Dbtr><PstlAdr><BldgNb>` | 楼号 | 地址 | originator_address |
| `<Dbtr><PstlAdr><TwnNm>` | 城市 | 地址 | originator_address |
| `<Dbtr><PstlAdr><Ctry>` | 国家 | 地址 | originator_address |
| `<DbtrAcct><Id>` | 付款人账号 | 账户 | originator_account |
| `<DbtrAgt><FinInstnId><BIC>` | 付款人银行 BIC | 机构标识 | originator_lei |
| `<Cdtr><Nm>` | 收款人姓名 | 姓名 | beneficiary_name |
| `<CdtrAcct><Id>` | 收款人账号 | 账户 | beneficiary_account |
| `<CdtrAgt><FinInstnId><BIC>` | 收款人银行 BIC | 机构标识 | beneficiary_lei |
| `<IntrBkSttlmAmt>` | 结算金额和币种 | 交易数据 | amount + token |
| `<IntrBkSttlmDt>` | 结算日期 | 交易数据 | timestamp |
| `<RmtInf>` | 汇款附言/Memo | 附加信息 | memo 字段 |

### 3.2 ISO 20022 对 Altius Token Standard 的启示

1. **FATF 建议 16 第 4 段明确提到 ISO 20022**："转账伴随的信息应尽可能按照 ISO 20022 等标准进行结构化"
2. **字段命名应与 ISO 20022 对齐**：Debtor = Originator = 发起人；Creditor = Beneficiary = 受益人
3. **Memo/汇款附言（RmtInf）是必需的**：Tony 在会议中提到的 ISO 20022 Memo 字段就是 `<RmtInf>`，用于携带汇款目的等结构化附加信息
4. **BIC/LEI 标识符**：对法人实体，需要携带银行的 BIC 代码或 LEI 标识

---

## 四、综合技术映射表

| # | 合规规则 | 来源 | 适用地区 | 规则摘要 | 对 Altius 的技术要求 | 对应模块 | 优先级 |
|---|---|---|---|---|---|---|---|
| 1 | **Travel Rule — 基本信息** | FATF R.16, INR.16 §8-9 | 全球 | 所有转账须携带发起人和受益人姓名+账号 | Token Standard：originator/beneficiary name + account 字段 | Compliance Standards | P0 |
| 2 | **Travel Rule — 扩展信息** | FATF INR.16 §9(c-e) | 全球 | 超过 1,000 USD/EUR 须携带地址、出生日期（自然人）或 LEI/BIC（法人）| Token Standard：address, dob, LEI 字段 | Compliance Standards | P0 |
| 3 | **Travel Rule — 金额阈值** | FATF INR.16 §8, INR.15 §7(a) | 全球（各国可自定义） | De minimis 阈值不超过 1,000 USD/EUR；超过此阈值须做 CDD | Transfer Hook：金额阈值检查 + CDD 触发 | Transfer Hooks | P0 |
| 4 | **Travel Rule — VA 转账** | FATF INR.15 §7(b), VA Guidance §281-294 | 全球 | VA 转账适用与传统电汇相同的 Travel Rule；信息须同步提交 | Transfer Hook：转账前验证信息完整性 | Transfer Hooks | P0 |
| 5 | **Travel Rule — 信息拒绝** | FATF INR.16 §23, §27, §31 | 全球 | 若转账缺少必要信息，发起机构不得执行；受益/中介机构应有执行/拒绝/暂停的政策 | Transfer Hook：缺少必要字段时阻止转账 | Transfer Hooks | P0 |
| 6 | **制裁名单筛查 — OFAC** | OFAC Framework, 31 CFR Part 501 | 美国（涉及美元/美国人的全球交易） | 必须筛查 SDN List 和 SSI List；须支持模糊匹配和替代拼写 | Transfer Hook：实时地址+实体名称筛查 | Transfer Hooks | P0 |
| 7 | **制裁名单筛查 — EU** | EU Council Regulations | 欧盟 | 须筛查欧盟合并制裁名单 | Transfer Hook：EU 制裁名单筛查 | Transfer Hooks | P0 |
| 8 | **名单同步** | OFAC Framework p.5 | 全球 | 制裁名单必须及时更新；未更新是常见违规根因 | Dashboard：自动化名单同步 + 更新日志 | Dashboard | P0 |
| 9 | **ISO 20022 Memo 字段** | ISO 20022 (pacs/pain), FATF INR.16 §4 | 全球（银行间） | 转账应按 ISO 20022 结构化；须支持汇款附言（RmtInf）| Token Standard：结构化 memo 字段 | Compliance Standards | P0 |
| 10 | **ISO 20022 字段对齐** | ISO 20022, FATF INR.16 脚注 48, 58 | 全球 | FATF 术语与 ISO 20022 可互换：Originator=Debtor, Beneficiary=Creditor | Token Standard 字段命名对齐 ISO 20022 | Compliance Standards | P1 |
| 11 | **钱包地址黑名单** | FATF VA Guidance §273 | 全球 | VASP 应维护和筛查黑名单钱包地址 | Transfer Hook：链上地址黑名单前置检查 | Transfer Hooks | P0 |
| 12 | **冻结义务** | FATF R.6, INR.16 §1(c) | 全球 | 须执行针对被制裁人员/实体的冻结行动，禁止交易 | Transfer Hook：冻结账户检查 | Transfer Hooks | P0 |
| 13 | **可疑交易报告（STR）** | FATF R.20, VA Guidance §275-278 | 全球 | 交易监控须基于风险，识别可疑交易并报告给 FIU | Dashboard：交易监控 + STR 生成 | Dashboard | P1 |
| 14 | **交易记录保留** | FATF R.11, OFAC Framework p.6 | 全球 | 所有 originator/beneficiary 信息须保留至少 5 年 | Dashboard：审计日志（5 年保留期）| Dashboard | P1 |
| 15 | **合规计划框架** | OFAC Framework 5 Components | 美国（最佳实践全球适用） | 须有：管理层承诺、风险评估、内部控制、测试审计、培训 | Dashboard：合规计划配置和审计模块 | Dashboard | P2 |

---

## 五、下一步

此映射表目前覆盖了**维度一：转账/支付合规**。接下来需要：

1. **维度二：数据隐私合规**（GDPR、CCPA、PDPA）— 对 Opt-in Privacy 模块和架构设计原则的要求
2. **维度三：AML/KYC 合规** — 对 Transfer Hooks 和 Dashboard 的扩展要求
3. **整合三个维度** → 完整映射表 → 产出二（合规覆盖矩阵）→ 产出三（地区速查卡）

---

*创建日期：2026-03-18*
*项目：Tabula — 机构隐私合规研究*

### 信息来源

| 来源 | URL |
|---|---|
| FATF Recommendations 2012 (Updated Oct 2025) | https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Fatf-recommendations.html |
| FATF Updated Guidance for VA/VASP (2021) | https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Guidance-rba-virtual-assets-2021.html |
| OFAC Compliance Framework | https://ofac.treasury.gov/media/16331/download?inline |
| OFAC SDN List (数据下载) | https://ofac.treasury.gov/specially-designated-nationals-and-blocked-persons-list-sdn-human-readable-lists |
| EU Consolidated Sanctions List | https://www.consilium.europa.eu/en/policies/sanctions/consolidated-list/ |
| ISO 20022 消息目录 | https://www.iso20022.org/catalogue-messages |
| ISO 20022 for Dummies (SWIFT 6th Ed) | 本地文件 context/ISO20022.pdf |
