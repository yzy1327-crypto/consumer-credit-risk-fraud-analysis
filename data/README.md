# Data

本文件夹用于存放项目中使用的数据文件。

## 1. Credit Default Data

违约风险分析使用公开的 UCI **Default of Credit Card Clients** 数据集。

该数据不在仓库中重复保存，Notebook 会通过 `ucimlrepo` 直接读取公开数据。

对应 Notebook：

`notebooks/01_credit_default_analysis.ipynb`

`notebooks/02_sql_risk_analysis.ipynb`

---

## 2. Application Fraud Data

文件：

`fraud_df.csv`

该数据为本项目使用的**模拟消费信贷申请欺诈数据**，用于展示申请反欺诈场景下的数据分析、风险特征工程、规则组合和风险分层方法。

### 字段说明

| 字段 | 含义 |
|---|---|
| `application_id` | 申请编号 |
| `apply_time` | 申请时间 |
| `id_number` | 身份证号标识 |
| `phone_number` | 手机号标识 |
| `device_id` | 设备编号 |
| `ip_address` | IP地址 |
| `apply_region` | 申请地区 |
| `apply_channel` | 申请渠道 |
| `loan_amount` | 申请贷款金额 |
| `apply_freq_7d` | 近7天申请次数 |
| `id_consistency` | 身份信息是否一致，1=一致，0=不一致 |
| `is_new_device` | 是否使用新设备申请，1=是，0=否 |
| `credit_history` | 是否存在信用历史 |
| `fraud_flag` | 欺诈标签，1=欺诈申请，0=正常申请 |

### 数据说明

- 数据量：2,000条申请记录
- 目标变量：`fraud_flag`
- 本数据为模拟数据，不代表真实金融机构的实际欺诈率
- 部分风险特征与欺诈标签之间的关系较强，因此相关结果仅用于展示分析和策略构建方法
- 不能将项目中的阈值直接用于真实业务审批、拦截或授信决策

对应 Notebook：

`notebooks/03_application_fraud_analysis.ipynb`

---

## 3. Output Data

最终欺诈风险结果表位于：

`outputs/fraud_result.csv`

其中保留：

- `application_id`
- 5项欺诈风险信号
- `fraud_rule_count`
- `fraud_risk_level`
- `fraud_flag`

用于展示每笔申请的规则命中情况和最终风险分层结果。
