# 隐私合规研究行动计划（修订版）

> **目标**：产出可直接指导 Altius L1 产品开发的隐私合规规则集，同时作为面向客户的专业展示材料。
>
> **聚焦**：隐私（Privacy）× 支付/银行场景
>
> **更新日期**：2026-03-17

---

## 背景

### 已完成的工作

| 产出 | 文件 | 状态 |
|---|---|---|
| ISO 27701:2025 深度解读 | `notes/iso-27701-analysis.md` | ✅ 完成 |
| 竞品深度分析（Kaleido vs Canton） | `notes/competitor-analysis-kaleido-canton.md` | ✅ 完成 |

### 核心发现（来自已完成工作）

1. **Canton** 有协议层隐私但无合规认证、无 ZKP、不兼容 EVM
2. **Kaleido** 有 SOC 2 + ISO 认证但隐私停留在应用层
3. **两者都没有 ISO 27701**，都没有原生同意管理、自动化决策透明、数据主体权利门户
4. **Altius 的机会**：合规原生（规则内置到 Middleware Layer）+ 可选隐私（ZK/TEE/Viewing Key）+ EVM 兼容

### Tony 在 3/16 会议中的关键指示

> "每家银行它都是要符合同样的规则的，我们把这些所有的规则都实现了，它就只需要上来填参数了。"

这决定了 research 的最终形态：**不是一篇论文，而是一张可编码的规则表。**

---

## 三个产出物

### 产出一：隐私合规规则技术映射表（核心交付物）

**是什么**：一张大表，把全球主要地区的隐私合规规则翻译成 Altius L1 的具体产品需求。

**表格结构**：

| 合规规则 | 适用地区 | 规则摘要 | 对链上产品的技术要求 | 对应 Altius 模块 | 优先级 |
|---|---|---|---|---|---|
| FATF Travel Rule | 全球 | 转账超阈值须附带发送方/接收方身份 | Transfer Hook: 金额阈值检查 + 身份字段附加 | Transfer Hooks | P0 |
| ISO 20022 Memo | 全球（银行间） | 支付消息须包含结构化附加信息 | Token Standard: 转账支持 Memo 字段 | Compliance Standards | P0 |
| ... | ... | ... | ... | ... | ... |

**需要调研的三个维度**（按 Tony 指定的场景聚焦）：

#### 维度一：转账/支付合规

这是和 Altius 产品最直接相关的维度，因为直接对应 Transfer Hooks 和 Token Standard。

需要逐条调研的规则：
- [ ] **FATF Travel Rule**：转账阈值是多少？需要哪些具体字段（发送方姓名、地址、账号等）？不同国家的阈值差异？
- [ ] **ISO 20022**：哪些数据字段涉及 PII？Token Standard 需要支持哪些字段？
- [ ] **EU PSD2 / PSD3**：支付服务商的数据保护义务有哪些？对 API 的具体要求？
- [ ] **新加坡 PSA（Payment Services Act）**：对 DPT（Digital Payment Token）服务商的隐私要求？
- [ ] **美国 MSB（Money Services Business）**：FinCEN 对虚拟货币交易商的记录保留要求？
- [ ] **制裁名单合规**：OFAC（美国）、EU Sanctions List — 技术上如何实现地址黑名单检查？需要多频繁更新？

#### 维度二：数据隐私合规

这对应 Altius 的 Opt-in Privacy 模块和整体架构设计原则。

需要调研的规则（只关注对链上产品的技术要求，不写法律论文）：
- [ ] **GDPR**：哪些条款直接影响链上产品设计？（重点：Art.17 删除权、Art.25 Privacy-by-Design、Art.44-49 跨境传输）→ 对 Altius 的具体要求是什么？
- [ ] **CCPA/CPRA**：和 GDPR 的差异在哪？对 Altius 有什么额外要求？
- [ ] **新加坡 PDPA**：对数据处理者的技术义务？跨境传输的具体条件？
- [ ] **数据本地化要求**：哪些国家强制要求数据不出境？（中国 PIPL、印度 DPDP、俄罗斯等）→ 这直接支持 Altius "一国一链"的架构决策

#### 维度三：AML/KYC 合规

这对应 Transfer Hooks（黑白名单）和 Enterprise Layer Dashboard（审计）。

- [ ] **各国 AML 法规对交易监控的技术要求**：实时 vs. 批量？阈值报告 vs. 可疑行为报告？
- [ ] **KYC 等级与交易限额的关系**：不同 KYC 等级能做多大金额的交易？→ Transfer Hook 参数
- [ ] **审计追踪要求**：监管机构要求保留多长时间的交易记录？什么格式？→ Dashboard + Viewing Key

### 产出二：合规覆盖矩阵（销售材料）

**是什么**：一张简洁的对比图，展示 Altius vs. Canton vs. Kaleido 在关键合规能力上的差异。

**数据来源**：从产出一中提取 top 15-20 条最重要的规则，加上已完成的竞品分析结果。

**表格结构**：

| 合规要求 | Canton | Kaleido | Altius（规划中） |
|---|---|---|---|
| FATF Travel Rule 支持 | ❌ | ⚠️ 应用层 | ✅ Transfer Hook 原生支持 |
| ISO 20022 Memo 字段 | ❌ | ❌ | ✅ Token Standard 内置 |
| GDPR 删除权 | ⚠️ | ⚠️ | ✅ PII 链下 + 架构设计 |
| ... | ... | ... | ... |

**用途**：直接放进 Tony 的销售 PPT。

### 产出三：重点地区合规速查卡

**是什么**：EU、US、SG 三个地区各一页，用最精炼的方式总结"在这个地区做区块链支付，隐私方面你需要知道什么"。

**每张卡的结构**：
1. 核心法规（一句话概括）
2. 对区块链支付产品的 3-5 条具体技术要求
3. Altius 如何满足（对应到 Middleware 模块）
4. 注意事项 / 特殊要求

---

## 执行计划

### 第一步：转账/支付合规调研（产出一的维度一）

**为什么先做这个**：和 Altius 的 Transfer Hooks + Token Standard 最直接相关，Tony 在会上举的例子（ISO 20022 Memo、黑名单）都属于这个维度。

具体任务：
1. 调研 FATF Travel Rule 的具体字段要求和各国阈值差异
2. 调研 ISO 20022 支付消息中的 PII 字段
3. 调研 OFAC/EU 制裁名单的技术接入方式
4. 调研 EU PSD2、新加坡 PSA、美国 MSB 的关键技术要求
5. 整理成映射表格式

### 第二步：数据隐私合规调研（产出一的维度二）

具体任务：
1. GDPR 对链上产品的具体技术要求（不是全文解读，而是提取可编码的规则）
2. CCPA vs. GDPR 的差异项
3. 新加坡 PDPA 的技术要求
4. 汇总数据本地化要求（哪些国家、什么条件）

### 第三步：整合产出

1. 合并产出一的三个维度 → 完整的隐私合规规则技术映射表
2. 从映射表中提取关键项 → 产出二（合规覆盖矩阵）
3. 按地区重组 → 产出三（地区速查卡）

---

## 参考资料

### 已有的内部资料
- ISO 27701:2025 原文（`context/ISO-IEC-27701-2025.pdf`）
- Canton Network 完整文档（`docs/canton-docs.md`）
- Kaleido 完整文档（`docs/kaleido-docs.md`）

### 需要查阅的外部资料
- FATF 建议 16 及虚拟资产更新指引（2021、2023）
- ISO 20022 消息规范（支付类：pain、pacs）
- OFAC SDN List 技术接入文档
- EU PSD2 / PSD3 法规文本
- 新加坡 PSA 及 MAS 指引
- GDPR 全文（重点：Art.17, 25, 44-49）
- CCPA/CPRA 法规文本

---

*创建日期：2026-03-17*
*项目：Tabula — 机构隐私合规研究*
