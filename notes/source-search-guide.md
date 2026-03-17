# 外部资料搜索指南

> 以下资料按优先级排列，全部是免费公开文档。
>
> 日期：2026-03-17

---

## 优先级 P0：转账/支付合规（最先需要）

### 1. FATF 建议 16 + 虚拟资产指引

这是 Travel Rule 的源头文件，规定了转账时必须携带哪些身份信息。

| 文档 | 搜索关键词 | 说明 |
|---|---|---|
| FATF 40 条建议全文 | `FATF Recommendations 2012 updated PDF site:fatf-gafi.org` | 建议 16 在其中，定义了电汇中必须携带的发起人/受益人信息 |
| 虚拟资产更新指引 | `FATF Updated Guidance Virtual Assets VASPs 2021 PDF site:fatf-gafi.org` | 2021 年版，把 Travel Rule 扩展到虚拟资产，定义了 VASP 的义务 |
| INR.15 解释性说明 | `FATF Interpretive Note Recommendation 15 Virtual Assets` | 专门针对虚拟资产/VASP 的解释性说明 |

**我们需要从中提取的信息**：
- 转账阈值是多少（USD/EUR 1,000）？
- 必须携带的具体字段（发起人姓名、账号、地址等）
- VASP 之间的信息传递要求
- 不同金额区间的不同要求

**直接 URL 尝试**：`https://www.fatf-gafi.org/en/publications/Fatfrecommendations/Fatf-recommendations.html`

### 2. OFAC 制裁名单技术文档

美国财政部的制裁名单，所有金融系统都必须做地址/实体筛查。

| 文档 | 搜索关键词 | 说明 |
|---|---|---|
| SDN List 说明 | `OFAC SDN List FAQ site:treasury.gov` | 制裁名单的基本说明和更新频率 |
| 数据格式文档 | `OFAC SDN Advanced XML Data Formats site:treasury.gov` | 技术接入：数据的 XML/CSV 格式说明 |
| 合规指引 | `OFAC Compliance Framework Guidance 2019 site:treasury.gov` | 合规框架指引，讲如何建立制裁筛查程序 |

**我们需要从中提取的信息**：
- 名单的数据格式（用于 Transfer Hook 的黑名单检查）
- 更新频率（多久需要同步一次）
- 筛查的技术标准（精确匹配 vs. 模糊匹配）

**直接 URL 尝试**：`https://ofac.treasury.gov/specially-designated-nationals-and-blocked-persons-list-sdn-human-readable-lists`

### 3. EU 制裁名单

和 OFAC 类似，是欧盟版本。

| 文档 | 搜索关键词 | 说明 |
|---|---|---|
| EU 制裁数据库 | `EU Consolidated Financial Sanctions List site:europa.eu` | 欧盟合并制裁名单 |
| 技术接入 | `EU sanctions list API XML download site:europa.eu` | 数据下载和接口说明 |

---

## 优先级 P1：数据隐私法规

### 4. GDPR 全文

欧盟通用数据保护条例。我们不需要读全部 99 条，只需要重点读几条。

| 文档 | 搜索关键词 | 说明 |
|---|---|---|
| GDPR 官方全文 | `GDPR full text regulation EU 2016/679 site:eur-lex.europa.eu` | 官方法律文本，免费 |
| GDPR 可读版本 | `GDPR full text gdpr-info.eu` | gdpr-info.eu 提供按条款浏览的版本，比 PDF 方便 |

**我们只需要重点读的条款**：
- Art.5（数据处理原则 — 数据最小化、目的限制）
- Art.6（处理的合法依据）
- Art.17（删除权 / 被遗忘权）
- Art.22（自动化决策）
- Art.25（Privacy by Design）
- Art.44-49（跨境数据传输）

**直接 URL 尝试**：`https://gdpr-info.eu/`（最方便的按条款浏览版本）

### 5. CCPA/CPRA

加州消费者隐私法。和 GDPR 有很多重叠但也有独特要求。

| 文档 | 搜索关键词 | 说明 |
|---|---|---|
| CCPA 法规文本 | `California Consumer Privacy Act full text site:oag.ca.gov` | 加州司法部官网 |
| CPRA 修正案 | `CPRA California Privacy Rights Act 2020 full text` | 2020 年对 CCPA 的修正 |
| CCPA vs GDPR 对比 | `CCPA GDPR comparison differences chart` | 找一份差异对比表，比读两份全文高效得多 |

**我们重点关注的差异**：
- CCPA 的"出售个人信息"概念（GDPR 没有）
- Opt-out vs. Opt-in 的区别
- 对区块链数据的适用性

### 6. 新加坡 PDPA

| 文档 | 搜索关键词 | 说明 |
|---|---|---|
| PDPA 全文 | `Singapore Personal Data Protection Act PDPA full text site:sso.agc.gov.sg` | 新加坡法律官网 |
| PDPC 指引 | `PDPC Advisory Guidelines Key Concepts site:pdpc.gov.sg` | 新加坡个人数据保护委员会的实务指引，比法律原文更有用 |

---

## 优先级 P2：支付服务法规

### 7. EU PSD2

欧盟支付服务指令，规定支付服务提供商的义务。

| 文档 | 搜索关键词 | 说明 |
|---|---|---|
| PSD2 全文 | `PSD2 Directive 2015/2366 full text site:eur-lex.europa.eu` | 官方法律文本 |
| PSD2 简要指南 | `PSD2 summary key requirements payment services` | 找一份总结性的指南比读完整法律文本更高效 |

**我们重点关注**：Strong Customer Authentication (SCA)、支付数据的保护义务、第三方支付服务商（TPP）的数据访问权

### 8. 新加坡 PSA（Payment Services Act）

| 文档 | 搜索关键词 | 说明 |
|---|---|---|
| PSA 全文 | `Singapore Payment Services Act 2019 full text site:sso.agc.gov.sg` | 法律原文 |
| MAS 指引 | `MAS Notice PSN01 PSN02 Digital Payment Token site:mas.gov.sg` | MAS 针对 DPT 服务商的具体合规通知，比法律原文更实用 |

### 9. 美国 FinCEN/MSB 要求

| 文档 | 搜索关键词 | 说明 |
|---|---|---|
| MSB 合规指引 | `FinCEN MSB Money Services Business compliance guide site:fincen.gov` | FinCEN 对货币服务商的合规指引 |
| 虚拟货币指引 | `FinCEN Guidance Convertible Virtual Currency 2019 FIN-2019-G001` | 2019 年的虚拟货币指引，定义了哪些活动需要 MSB 牌照 |

---

## 搜索技巧

1. **优先用 `site:` 限定官方网站**：如 `site:fatf-gafi.org`、`site:eur-lex.europa.eu`、`site:treasury.gov`
2. **加 `PDF` 或 `filetype:pdf`**：如果想直接找可下载的 PDF 版本
3. **找 "guidance" 而非 "full text"**：很多时候官方发布的 guidance/advisory 比法律原文更实用，因为它直接告诉你"该怎么做"
4. **找 "comparison" 或 "chart"**：对于 CCPA vs GDPR 这种对比型任务，找现成的对比表远比自己读两份全文高效

---

## 下载后放哪里

所有下载的文档放到 `context/` 目录下，建议命名格式：

```
context/
├── ISO-IEC-27701-2025.pdf          # 已有
├── ISO20022.pdf                     # 已有（for Dummies 版）
├── FATF-Recommendations-2012.pdf
├── FATF-Virtual-Assets-Guidance-2021.pdf
├── GDPR-EU-2016-679.pdf            # 或者直接用 gdpr-info.eu 网页版
├── OFAC-Compliance-Framework.pdf
├── ...
```

---

*创建日期：2026-03-17*
