# 2026-09-19 现场四问 — 技术侧复盘

> **本篇是这次现场技术侧的权威版本**：每题「当场怎么答的」对「现在该怎么答」，
> 每条结论都挂 `file:line` 或可复跑的数字。技术数字以本篇为准。
>
> ToucanShelf 的 `UOIP/events/20260919-Day of Data 现场纪实` 记人、记场面、
> 记会后线索。按 2026-09-20 重定的边界（见 [`README.md`](README.md#三个承载地的边界2026-09-20-重定)），
> 那边只保留与工程无直接关系的部分——**不是本篇的另一个版本，是另一块内容**。
>
> 复盘日期：2026-09-20。纪实里标「待核实 / 待确认」的三处，本篇结掉。

---

## 总览

| # | 问题 | 当场 | 现在 |
|---|---|---|---|
| 1 | 数据量不大为什么用 Spark | 说了 1800 万 | 🔴 **口径错位**，不是错数 |
| 2 | 为什么自己认定 weather event | 「数据源和目标不同」 | 答案没错但**扔掉了最硬的证据** |
| 3 | 没看到 AI 为什么叫 AI-driven | 「有规划，部分尚未落地」 | 🔴 **答反了——说少了，不是说多了** |
| 4 | `Poisson GLM · live` | 答不上来 | 见 [`m1-defence.md`](m1-defence.md) |

🔴 **四题里有三题，正确答案当时就已经存在于仓库或设计文档里。** 不是能力不够，
是**准备的形式不对**——QA.md 按「没有技术背景的大学生」写了 30 问，到场的是
同行。听众画像设错，备了三十发子弹一发没用上。

---

## 问题一 · 数据量看起来不大，为什么要用 Spark

**当场**：数据量不小，光市民投诉就有 1800 万条。

**纪实标的疑点**：QA.md Question 19 写的是「约 12.5 million service-request
rows」，跟 1800 万对不上。

### 🔴 结论：两个数都是真的，是**口径错位**，不是记错

| 数字 | 是什么 | 出处 |
|---|---|---|
| **18,375,656** | **上游整表**（Socrata 侧全量，2026-08-09 实测） | 契约 `full_table_min` |
| **≈12.48 M** | **我们实际采集并处理的** Silver 行数 | 12,474,313（2026-08-18 首次全量）· 12,477,414（2026-08-19 复测，与建 Gold 时逐行相同） |

差的 590 万不是缺口，是**采集范围有意不是全历史**：2016-08-01 起采全天，
之前只采冬季。CLAUDE.md 对此有一条红字警告——
🔴「对账的分母不是 18.4 M」，拿 18.4 M 对 Silver 会看到 590 万行的假缺口。

所以当场说 1800 万**不算撒谎，但答错了题**：他问的是「你们处理多少」，
你给的是「上游有多少」。

### 该这么答

> About 12.5 million service-request rows in the cleansed layer, across 4,878
> day partitions. The upstream table is larger — around 18.4 million — but we
> deliberately don't ingest all of it: full-year from 2016, winter-only before
> that, because the analysis is winter operations.

**Spark 的真实理由不是行数，是分区数。** 这一点更能答住同行：

> Honestly, 12 million rows isn't a Spark-sized number on its own. What drove
> it was partition count and the shape of the work — 4,878 day partitions, a
> point-in-polygon join against 25 zone geometries, eight of which needed
> `make_valid`. And the query layer has a measured wall: seven real columns
> across all 4,878 partitions times out, so everything is written to prune on
> the partition column.

证据：`.claude/rules/gold-sql.md` 开头那张实测表（单日 1,385 行秒回；
全表 4,878 分区 🔴 `Read timed out`）。
⚠️ 注意 2026-09-01 的更正：**那堵墙的高度是部署决定的**——存储节点与计算节点
跨数据中心，每次对象读都过 WAN。同样 4,878 个分区如果同机房未必会失败。
别把它说成代码的固有属性。

---

## 问题二 · 市政有些数据按 weather event 组织，你们为什么自己认定

**当场**：数据源和目标不同。市政那套服务市政运营，我们服务居民。

**这个答案没错，但它是个立场陈述，不是证据。** 而证据是现成的，而且很硬。

### 该补的那一层

事件切分不是随手定的阈值，是**测出来的**，而且中途被自己的数据推翻过一次：

1. 初版判据：单日降雪 ≥ **3 cm**。选定后 N = 100（排班期 59），
   ward × 事件非零率 77.9%。
2. 🔴 **然后发现 19 次犁雪里有 4 次，在任何单日阈值下都不落在降雪事件内。**
3. 查因：三条猜想（吹雪风积 / 冻雨 / 单点气象代表性）**全部证伪**，
   三个比值都 ≤ 0.31 且方向与假设相反。真因是**阈下累积**——21 日累计降雪
   保留了对照组的 76%，单日峰值只保留 26%，差 2.92 倍。雪照样下了同量级，
   只是从没有单日超过 3 cm。
4. 于是判据改成「单日阈值 **或** 滚动累积」的并集，`dim_snowfall_event`
   落 `accum_flag`。生产实测 **8 个 `accum_flag = true`**，N = 99 / 排班期 59。

> We didn't take the City's event definition because we're answering a
> different question, but we also didn't just pick a threshold. We started at
> 3 cm a day, then found four plow operations that fall outside *any*
> single-day threshold — the cause turned out to be sub-threshold accumulation,
> so the definition became a union of a daily threshold and a rolling one.
> Eight events in production exist only because of the second rule.

⚠️ **别说「那 4 次被这条判据救回来了」**——实测仍有 2 次
（2021-01-07 / 2026-02-26）`is_aligned = false`，8 个 accum 事件与那 4 次的
关系**没有量过**。

---

## 问题三 · 没看到 AI，叫 AI-driven 是因为项目是用 AI 做出来的吗

**当场**：有模型训练与预测 load score 的规划，**有一部分尚未落地，时间不够**。

### 🔴 这一题答反了。不是吹过头，是**说少了**。

M1 不是规划，是**跑通生产的**：

| | 实测 |
|---|---|
| M1 Poisson GLM + F5 全链路 | ✅ 跑通生产 |
| 预测面板 | **2,178 格，零缺失** |
| 留出季 2025-2026 | MAE **7.345** vs 基线 23.628 |
| F6 `fact_winter_event_zone_load` | **1,298 行** |
| F7 `fact_recommendation` | **748 行** |
| Gold 层 | **17 张表全部有生产数据**，零行的表 0，全空的列 0 |

真正没落地的只有一件：**接天气预报端到端**（那条通路一行数据都没跑过），
以及 M2（H2 范围）。

**而且当场该说的话，设计文档 §9 早就定稿了**，一字没改可以直接念：

> 「AI-driven」指排序与评分由模型预测驱动；推荐层本身是可审计的确定性逻辑，
> **这是刻意的设计而非能力缺失。**

英文版：

> The ranking and scoring are driven by a model's predictions — a Poisson
> count model that's been trained and is producing rows in the gold layer.
> What's deliberately *not* AI is the recommendation layer on top: that's
> auditable deterministic logic, by choice. A city crew scheduler needs to be
> able to ask why a zone came out first and get an answer, not a black box.

⚠️ **但两条禁语仍然成立，不能因为这题答少了就往回补过头**：
①「模型优于基线」不是可辩护的公开结论（留出季 7 个事件、零膨胀、
baseline 与模型嵌套）；② 不得说「消费实时预报」。

🔴 **这是当天最该带走的一题**——标题里的 AI-driven 与现场能演示的内容之间
被当众指出落差，而落差**其实不存在**，是自己没讲出来。

---

## 问题四 · 架构图上 `Poisson GLM · live` 那行小字

**当场**：正在和别人交流没听清，此前也没关注过这行字，完全答不上来。

全部答辩材料在 **[`m1-defence.md`](m1-defence.md)**。这里只记三件纪实里标了待办的：

1. ✅ **offset 有**。纪实写「按分区做计数是否放了 offset，待回代码确认」——
   有，`log(unit_size)`，`unit_size` = 分区地址数。
   `models/request_forecast/model.py:7` 的 docstring 第一句，
   实现在 `features.py:255`，`features.py:70` 还拦了 `unit_size <= 0`。
   **可以正面答「做了 offset，不是变相的面积排序」。**

2. 🔴 **新发现的真问题：`severity_score` 与另外两个特征构造性共线。**
   `severity_score = 0.5·norm(total_snowfall_cm) + 0.5·norm(GREATEST(0, -min_temperature_c))`
   （`sql/dml/dim_snowfall_event.sql:32,55-65`），而 m1.yaml 的特征里
   **同时有**这三个。min-max 是仿射变换，`GREATEST(0,·)` 只在气温 > 0 时非线性——
   Nov–Apr 的 Winnipeg 基本不会。**待验证的 SQL 在 `m1-defence.md` §7.1。**

3. ⚠️ **`live` 这个措辞建议改 `back-test · running`。** 它本意是相对 M2 的
   `planned` 而言，但容易读成「已在生产实时预测」。

---

## 这一天暴露的流程问题

### 1 听众画像设错，是根因

QA.md 按「没有技术背景的大学生」写了 30 问。到场的是数据与技术从业者，
问的全是技术选型、口径定义、命名是否名实相符。**前三问没有一个落在预设里。**

下次：**先确认听众构成，再写 Q&A。**

### 2 讲稿不讲的东西，图上不要印

问题四是图与讲稿打架的直接后果。讲稿 Slide 22 写了「不显示模型名」，
QA.md Question 15 写了「如无追问不必主动讲」，而架构图的小字把模型名印上去了。
**图先把话说出口，讲稿却没配弹药。**

### 3 🔴 机械审计的结果：不是一行字的问题，是 69 行

2026-09-20 对最终 deck 跑了一遍：架构图 **73 个标签，69 个在整场从没被念过**。
`Poisson GLM · live` 只是其中被问到的那一条。

**根因是它不是文字，是像素**——出处 `docs/images/platform-architecture.svg`，
以 PNG 贴进第 22 页。用 python-pptx 提取全部 43 页文本，**这行字一个字都不出现**。
所以任何基于文本的讲稿覆盖检查，都**结构性地**跳过它。

**流程修正**：图里的字要从**源文件**（`.svg` / `.drawio.xml` / 生成脚本）抓出来
对讲稿，不能从 pptx 抓。讲稿没覆盖的标签，要么删掉，要么补备答，没有第三种状态。

按「资深听众会先问哪个」排，前几条：`Sole source of truth · no backup`（🔴 比模型
危险，运维出身的人会当场问）· `M2 · Logistic / GBM`（跟 Poisson 同形状）·
`cross-DC` · `Hive Parquet → Iceberg (H2)` · `client mode` ·
`Docker · 4 core / 24 GB ARM`（⚠️ 自己先对一遍，CLAUDE.md 记的是可用 7 GB）。
