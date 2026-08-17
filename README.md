# Consumer Credit Risk & Fraud Analysis

消费信贷违约风险与申请欺诈风险分析项目，基于公开信用卡违约数据与模拟申请欺诈数据，使用 **Python、Pandas、SQL 和 SQLite** 完成风险特征分析、特征工程、规则组合与风险分层。

本项目重点展示的不是机器学习建模，而是如何从业务字段出发，识别具有解释力的风险信号，并通过数据分析验证其风险区分能力。

---

## 1. Project Overview

消费信贷风险可以从不同业务阶段进行观察。本项目主要覆盖两个方向：

### 1.1 贷后违约风险
基于客户历史还款状态、授信额度、账单金额和还款金额等信息，分析哪些行为和账户特征与下一期违约风险存在明显关联。

主要分析内容包括：

- 最近一期是否逾期
- 逾期严重程度
- 近6个月逾期频率
- 近6个月最大逾期程度
- 连续逾期行为
- 授信额度
- 年龄
- 信用额度使用率
- 还款覆盖情况
- 多风险规则组合与风险分层

### 1.2 申请欺诈风险
基于模拟消费信贷申请数据，分析申请阶段的异常行为和身份风险信号，并构建可解释的欺诈风险规则。

主要分析内容包括：

- 近7天申请频率
- 身份信息一致性
- 异常申请地区
- 新设备申请
- 信用历史
- 多规则命中分析
- 低 / 中 / 高欺诈风险分层

---

## 2. Tech Stack

- **Python**
- **Pandas**
- **SQL**
- **SQLite**
- **Jupyter Notebook**

其中，Python / Pandas 主要用于数据清洗、特征构造、分组统计和风险规则组合；SQL 用于从账户字段中提取和验证核心风险指标。

---

## 3. Repository Structure

```text
consumer-credit-risk-fraud-analysis/
│
├── README.md
│
├── notebooks/
│   ├── 01_credit_default_analysis.ipynb
│   ├── 02_sql_risk_analysis.ipynb
│   └── 03_application_fraud_analysis.ipynb
│
├── data/
│   ├── README.md
│   └── fraud_df.csv
│
├── outputs/
│   └── fraud_result.csv
│
└── requirements.txt
```

---

## 4. Analysis Workflow

```text
数据检查
   ↓
单变量风险分析
   ↓
风险特征工程
   ↓
风险规则构造
   ↓
规则命中分析
   ↓
风险分层
   ↓
业务解释与局限性评估
```

项目中的风险指标均尽量保持业务可解释性，而不是直接使用黑盒模型进行预测。

---

## 5. Notebook 01 — Credit Default Risk Analysis

文件：

`notebooks/01_credit_default_analysis.ipynb`

该 Notebook 使用 UCI **Default of Credit Card Clients** 数据集，共包含 **30,000 条客户记录**。

目标变量：

- `default_flag = 0`：下一期未违约
- `default_flag = 1`：下一期违约

整体下一期违约率约为 **22.12%**。

### 5.1 Historical Delinquency Features

构造并分析以下核心逾期特征：

| Feature | Description |
|---|---|
| `overdue_month_count_6m` | 近6个月出现正向逾期的月份数 |
| `max_overdue_months_6m` | 近6个月最大正向逾期程度 |
| `continuous_overdue_flag_6m` | 近6个月是否存在连续逾期 |

主要结果：

- 9月逾期客户下一期违约率约 **50.29%**，未逾期客户约 **13.83%**
- 近6个月无正向逾期客户违约率约 **11.71%**，6个月均出现逾期的客户约 **70.32%**
- 连续逾期客户违约率约 **53.53%**，非连续逾期客户约 **15.49%**

历史逾期行为是本项目中风险区分能力最明显的一类特征。

### 5.2 Customer & Account Features

进一步分析：

- 授信额度
- 年龄
- 信用额度使用率
- 还款覆盖情况

构造：

| Feature | Description |
|---|---|
| `sep_credit_utilization` | 9月账单金额 / 授信额度 |
| `sep_payment_to_bill_ratio` | 9月还款金额 / 9月账单金额 |

部分结果：

- 低授信额度组违约率约 **31.79%**，高授信额度组约 **13.26%**
- 超额使用组违约率约 **30.05%**，低额度使用组约 **18.30%**
- 零还款组违约率约 **39.77%**，超额还款组约 **13.81%**

这些变量存在较明显的风险差异，但部分关系并非严格单调，因此更适合作为组合风险判断的一部分。

### 5.3 Rule-based Risk Segmentation

在单特征分析基础上构造5项探索性风险信号：

1. 近6个月逾期月份数 ≥ 3
2. 近6个月最大逾期程度 ≥ 2
3. 近6个月存在连续逾期
4. 9月信用额度使用率 ≥ 80%
5. 9月账单金额 > 0 且还款金额 = 0

根据风险规则命中数量进行分层：

| Risk Level | Rule Count | Default Rate |
|---|---:|---:|
| Low | 0 | 11.95% |
| Medium | 1–2 | 21.56% |
| High | 3–5 | 55.88% |

随着风险信号叠加，客户下一期违约率明显上升。

---

## 6. Notebook 02 — SQL Risk Indicator Analysis

文件：

`notebooks/02_sql_risk_analysis.ipynb`

该 Notebook 使用与 Python 分析相同的数据，并将 DataFrame 写入 SQLite 表：

`credit_risk`

主要使用：

- `SELECT`
- `FROM`
- `WHERE`
- `COUNT`
- `SUM`
- `AVG`
- `CASE WHEN`
- `GROUP BY`
- `AND / OR`
- SQLite 多参数 `MAX()`

SQL 部分重点验证：

1. 整体客户数、违约人数及违约率
2. 9月逾期与未逾期客群风险差异
3. 逾期严重程度
4. 近6个月逾期月份数
5. 近6个月最大逾期程度
6. 连续逾期风险

该模块主要展示如何直接从底层账户字段中使用 SQL 构造并验证可解释的风险指标。

---

## 7. Notebook 03 — Application Fraud Risk Analysis

文件：

`notebooks/03_application_fraud_analysis.ipynb`

该模块使用 **2,000 条模拟消费信贷申请数据**：

- 正常申请：1,803
- 欺诈申请：197
- 整体欺诈率：9.85%

### 7.1 Fraud Risk Features

经过单特征分析，构造5项风险信号：

| Feature | Risk Condition |
|---|---|
| `high_freq_flag` | 近7天申请次数 ≥ 4 |
| `id_inconsistent_flag` | 身份信息不一致 |
| `abnormal_region_flag` | 境外或未知地区 |
| `is_new_device` | 使用新设备申请 |
| `no_credit_history_flag` | 无信用历史 |

部分结果：

- 高频申请组在当前模拟数据中的欺诈率明显高于非高频申请组
- 身份信息不一致客群欺诈率明显升高
- 新设备申请欺诈率约 **17.44%**，非新设备约 **4.12%**
- 无信用记录申请欺诈率约 **17.13%**，有信用记录约 **6.03%**

### 7.2 Fraud Rule Count

将5项风险信号相加构造：

`fraud_rule_count`

不同规则命中数对应的欺诈率：

| Rule Count | Fraud Rate |
|---:|---:|
| 0 | 0% |
| 1 | 2.07% |
| 2 | 18.01% |
| 3+ | 100%* |

\* 3条及以上规则命中组达到100%是当前模拟数据的特征，不能直接解释为真实业务中的确定性欺诈规则。

### 7.3 Fraud Risk Segmentation

根据规则命中数量进行探索性风险分层：

| Risk Level | Rule Count | Applications | Fraud Rate |
|---|---:|---:|---:|
| Low | 0–1 | 1,607 | 1.12% |
| Medium | 2 | 261 | 18.01% |
| High | 3+ | 132 | 100%* |

最终输出风险结果表：

`outputs/fraud_result.csv`

结果表包含：

- `application_id`
- 5项欺诈风险信号
- `fraud_rule_count`
- `fraud_risk_level`
- `fraud_flag`

---

## 8. Key Findings

### Default Risk

- 历史逾期行为是最明显的违约风险信号之一
- 逾期频率、严重程度和连续性均能够有效区分不同风险客群
- 授信额度、额度使用率和还款覆盖情况能够提供额外风险信息
- 多个风险信号组合后，风险层级之间呈现更明显的违约率梯度

### Fraud Risk

- 高频申请、身份异常和异常地区在模拟数据中具有较强的欺诈风险区分能力
- 新设备和缺少信用历史属于概率型风险信号，更适合与其他规则结合判断
- 随着风险规则命中数量增加，欺诈率明显上升
- 可解释的规则组合可以用于形成风险分层和人工审核优先级

---

## 9. Data

### Credit Default Data

违约风险分析使用公开的 **UCI Default of Credit Card Clients** 数据集。

本项目通过 Python 获取公开数据，因此仓库中不重复保存原始违约数据文件。

### Application Fraud Data

`data/fraud_df.csv`

为本项目使用的**模拟申请欺诈数据**，用于展示申请反欺诈分析、特征工程和规则策略构建方法。

模拟数据中的部分风险特征与欺诈标签关系较强，因此相关结果仅用于方法展示，不应直接用于真实业务决策。

---

## 10. Limitations

本项目存在以下限制：

- 违约分析使用公开历史数据，字段和业务口径有限
- 欺诈分析使用模拟数据，其风险规律比真实业务更清晰
- 当前规则阈值主要根据数据探索结果设置，尚未进行样本外验证
- 未进行训练集 / 测试集划分，也未构建机器学习模型
- 未评估真实业务中的通过率、召回率、误伤率、人工审核成本和风险损失
- 当前分析用于识别统计关联和风险区分能力，不用于证明因果关系
- 项目中的风险分层为探索性分析，不能直接作为真实金融机构的审批、授信或拦截策略

---

## 11. How to Run

安装依赖：

```bash
pip install -r requirements.txt
```

建议按照以下顺序运行：

```text
01_credit_default_analysis.ipynb
        ↓
02_sql_risk_analysis.ipynb
        ↓
03_application_fraud_analysis.ipynb
```

申请欺诈 Notebook 需要将：

`fraud_df.csv`

放置在 Notebook 可以读取的位置。

---

## 12. Project Positioning

本项目定位为：

> **消费信贷风险数据分析与可解释风险策略项目**

重点展示：

- Python / Pandas 数据分析
- SQL 风险指标提取
- 数据质量检查
- 风险特征工程
- 分组风险分析
- 可解释规则构建
- 风险分层
- 风控业务分析与结果解释

项目不以复杂模型为核心，而是强调从业务问题出发，将原始数据转化为可解释风险指标，并通过数据验证风险规则是否具有区分能力。
