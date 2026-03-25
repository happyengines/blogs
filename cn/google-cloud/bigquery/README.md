# BigQuery入门 - 了解Google Cloud的大规模数据分析平台

> 本文围绕Google Cloud的BigQuery，重点介绍"它能做什么"以及"为什么被广泛采用"。适合正在考虑引入BigQuery或希望了解数据分析平台全貌的读者。

## 结论：什么是BigQuery

BigQuery是**Google Cloud提供的全托管数据仓库（DWH）** 服务。

一句话概括：

- **无需服务器管理**，可在**数秒至数分钟内分析TB至PB级数据**的云端DWH
- **只需编写SQL**即可进行数据分析，无需基础设施知识也能上手
- **按量计费**，仅对实际使用量收费

```sql
-- BigQuery基本查询示例
-- 使用公开数据集统计GitHub提交数
SELECT
  author.name,
  COUNT(*) AS commit_count
FROM
  `bigquery-public-data.github_repos.commits`
WHERE
  committer.date > '2024-01-01'
GROUP BY
  author.name
ORDER BY
  commit_count DESC
LIMIT 10;
```

这样的查询即使面对**数TB的数据也能在数秒内返回结果**，这正是BigQuery的强大之处。

---

## 1. BigQuery的定位

### 在Google Cloud中的角色

Google Cloud拥有众多服务，而BigQuery负责**数据的存储、加工和分析**。

```text
数据流向（示意图）

[数据源] → [采集/导入] → [存储/加工] → [可视化/分析]
            Cloud Storage    BigQuery      Looker Studio
            Pub/Sub          (这里!)       Connected Sheets
            Dataflow                       Looker
```

![BigQuery在Google Cloud中的定位](./images/bigquery-in-gcp.png)
<!-- 截图: Google Cloud控制台 > BigQuery概览页面 -->

---

## 2. BigQuery的主要特点

### 2-1. 全托管且无服务器

传统DWH需要构建服务器、调优和扩展。BigQuery则**将这些全部交由Google管理**。

| 项目 | 传统DWH | BigQuery |
|------|---------|----------|
| 服务器构建 | 需要 | 不需要 |
| 扩展 | 手动配置 | 自动 |
| 调优 | DBA执行 | 基本不需要 |
| 维护 | 定期需要 | 不需要 |

### 2-2. 压倒性的处理速度

BigQuery基于Google的内部技术（Dremel、Colossus），通过**列式存储**和**大规模并行处理**，即使是TB级数据也能高速处理。

```sql
-- 示例：对约6.5TB的数据进行查询，也能在数分钟内完成
SELECT
  FORMAT_DATE('%Y-%m', created_at) AS month,
  COUNT(*) AS total_events
FROM
  `bigquery-public-data.github_repos.events`
GROUP BY
  month
ORDER BY
  month DESC;
```

![BigQuery查询执行结果 - 处理时间显示](./images/bigquery-query-result.png)
<!-- 截图: BigQuery控制台查询执行后的结果画面，显示处理字节数和执行时间 -->

### 2-3. 按量计费优化成本

BigQuery的定价体系简洁明了：

| 费用类型 | 内容 | 参考价格 |
|----------|------|----------|
| **存储** | 按保存数据量计费 | 活跃数据: $0.02/GB/月 |
| **查询（按需）** | 按处理数据量计费 | $5/TB |
| **免费额度** | 每月免费使用量 | 存储10GB + 查询1TB/月 |

**每月1TB以内的查询免费**，因此个人学习或小规模分析几乎可以零成本使用。

### 2-4. 直接使用标准SQL

BigQuery支持**标准SQL（Standard SQL）**，现有的SQL知识可以直接运用。

```sql
-- 窗口函数也可使用
SELECT
  department,
  employee_name,
  salary,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM
  `project.dataset.employees`;

-- WITH子句（CTE）也可使用
WITH monthly_sales AS (
  SELECT
    FORMAT_DATE('%Y-%m', sale_date) AS month,
    SUM(amount) AS total
  FROM
    `project.dataset.sales`
  GROUP BY
    month
)
SELECT * FROM monthly_sales WHERE total > 1000000;
```

### 2-5. 与Google服务轻松集成

BigQuery可以与以下Google服务轻松连接：

- **Google 表格**: 可直接查询表格中的数据
- **Google Analytics（GA4）**: 自动导出访问日志
- **Looker Studio**: 将查询结果用仪表板可视化
- **Cloud Storage**: 直接读取CSV/JSON/Parquet等文件

---

## 3. 开始使用BigQuery

### 3-1. 创建Google Cloud账号

1. 访问 [Google Cloud Console](https://console.cloud.google.com/)
2. 使用Google账号登录
3. 激活免费试用（$300额度 + 90天）

![Google Cloud Console 账号创建页面](./images/gcp-signup.png)
<!-- 截图: Google Cloud免费试用注册页面 -->

### 3-2. 打开BigQuery控制台

1. 从Google Cloud Console左侧菜单选择「BigQuery」
2. 或直接访问URL: `console.cloud.google.com/bigquery`

![BigQuery控制台初始页面](./images/bigquery-console.png)
<!-- 截图: BigQuery控制台初始画面，显示导航面板、查询编辑器、详情面板三个区域 -->

BigQuery控制台主要由3个区域组成：

| 区域 | 功能 |
|------|------|
| **导航面板**（左侧） | 项目、数据集、表的列表显示 |
| **查询编辑器**（中央上方） | 编写和执行SQL的区域 |
| **结果面板**（中央下方） | 显示查询执行结果或表的详细信息 |

### 3-3. 使用公开数据集试试看

BigQuery提供了可以立即试用的**公开数据集**，创建账号后即可使用。

```sql
-- 公开数据集示例：美国婴儿姓名
-- 无需添加数据集，可直接执行
SELECT
  name,
  gender,
  SUM(number) AS total
FROM
  `bigquery-public-data.usa_names.usa_1910_current`
WHERE
  year >= 2000
GROUP BY
  name, gender
ORDER BY
  total DESC
LIMIT 10;
```

![公开数据集查询执行示例](./images/bigquery-public-dataset.png)
<!-- 截图: 上述查询的执行结果画面 -->

### 3-4. bq命令行工具

除Web控制台外，还可以通过终端操作。

```bash
# 安装Google Cloud SDK后，即可使用以下命令

# 执行查询
bq query --use_legacy_sql=false \
  'SELECT name, SUM(number) as total
   FROM `bigquery-public-data.usa_names.usa_1910_current`
   GROUP BY name
   ORDER BY total DESC
   LIMIT 5'

# 列出数据集
bq ls

# 查看表结构
bq show bigquery-public-data:usa_names.usa_1910_current
```

---

## 4. 与其他DWH服务的比较

在考虑引入BigQuery时，通常会与以下服务进行比较。

| 方面 | BigQuery | Amazon Redshift | Azure Synapse |
|------|----------|-----------------|---------------|
| 计费方式 | 查询处理量 + 存储 | 节点数（按时计费） | 处理量或专用池 |
| 扩展 | 自动 | 手动调整 | 自动/手动 |
| 初始成本 | 有免费额度 | 2个月免费试用 | 有免费额度 |
| 优势 | Google集成、易用性 | AWS集成、稳定性 | Microsoft集成 |

**选择要点**: 如果已经大量使用Google服务（GA4、表格等），由于与BigQuery的集成非常顺畅，因此推荐选择BigQuery。

---

## 5. 容易踩坑的地方和注意事项

### 成本方面的注意

- **避免使用 `SELECT *`**: 扫描所有列会增加处理数据量，从而增加成本。请只指定需要的列。

```sql
-- 不推荐: 全列扫描（成本高）
SELECT * FROM `project.dataset.large_table`;

-- 推荐: 只指定需要的列（成本低）
SELECT id, name, created_at FROM `project.dataset.large_table`;
```

- **活用分区表**: 按日期进行分区可以限定扫描范围，降低成本。

```sql
-- 通过分区过滤限定扫描范围
SELECT *
FROM `project.dataset.partitioned_table`
WHERE date_column BETWEEN '2024-01-01' AND '2024-01-31';
```

### 执行查询前确认数据量

BigQuery控制台会在查询执行前在右上角显示**预估处理数据量**。请养成执行前必定确认的习惯。

![查询执行前的处理数据量显示](./images/bigquery-estimated-bytes.png)
<!-- 截图: 查询编辑器右上角「运行此查询将处理 ○○」的显示 -->

---

## 总结

| 要点 | 内容 |
|------|------|
| BigQuery是什么 | Google Cloud的全托管DWH |
| 优势 | 高速处理、无服务器、按量计费 |
| 入门方式 | Google Cloud免费试用 → 用公开数据集立即实践 |
| 成本优化 | 避免 `SELECT *`，活用分区 |

BigQuery是一个**"先试试再说"门槛非常低**的服务。建议先在免费额度范围内，使用公开数据集来尝试数据分析。

### 下一步

后续将陆续添加以下实践性文章：

- BigQuery成本优化技巧
- 分区与聚类的使用区分
- 使用BigQuery ML开始机器学习
- 使用Dataform构建数据管道

---

*最后更新: 2026-03-25*
