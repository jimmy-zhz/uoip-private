# M1 统计答辩手册

> 起因：2026-09-19 Day of Data Winnipeg 现场，一位未留名的资深听众（疑似
> Improving）指着架构图小字 `M1 demand forecast · Poisson GLM · live` 提问。
> 问题先落到同台的 professor（对项目不了解，答不了），再转到我，我答不上来。
>
> 他真正在测的不是「Poisson 是什么」，而是**这个选型是不是我自己做的**。
> 一旦他判定「AI 搞定一切」，求职与会后建联的评价都会塌。
>
> 本文件的目标不是背标准答案，是**能即兴重组已有的论证**。

---

## 0. 怎么用这份文件

三件事，优先级从高到低：

1. **§2 的四条主动暴露** —— 背熟，能在被问之前主动说出其中两条。
   这一节的价值高于全文其余部分之和。
2. **§7 的待验证项** —— 拿到线上环境就跑，回来把答法定死。
3. §3–§6 是深挖时的弹药库，不需要逐字背，需要知道「论证在哪」。

**核心判断：这个项目的统计选型全都有真实依据，而且全都写下来了。**
`docs/dev/design/20260820-scoring-chain-and-m1.md` §8 是一张明确的
「被否决的选项」表。`models/request_forecast/` 两个文件的 docstring 密度
接近论文。问题纯粹是这些东西不在我脑子里。

---

## 1. 一分钟版本（只来得及说一段时）

> M1 predicts how many winter-related resident reports each operational zone
> generates in one snowfall event. It's a Poisson GLM with `log(address_count)`
> as an offset — so we're modelling a per-address rate while keeping the target
> a count, which means MAE reads in real tickets. We hold out the most recent
> snow season, never a random split, and we report the model and a baseline as
> a pair. The word `live` on that diagram is overselling it: this is a
> back-test on archived weather, not a forecast-fed production model.

**`live` 这个措辞要改。** 图上写 `live` 是相对 M2 的 `planned` 而言，
指「这条链路已实现并在跑」，不是「已在雪前实时服务」。建议改
`back-test · running`。在被追问之前主动纠正它，比被拆穿好得多。

---

## 2. 四条主动暴露 🔴 最重要的一节

**懂的人和不懂的人，区别不在于能不能给出正确答案，在于能不能说出自己方案的代价。**

AI 生成的论证通常是单向辩护的——每个选择都完美。真人做的选型带着
「我知道这里不干净」。在被问之前主动说出下面任意两条，
「AI 搞定一切」这个判断就成立不了。

### 2.1 留出季样本量撑不起模型选择

留出季只有 **7 个事件 × 22 分区 = 154 格**，而且目标高度零膨胀
（25 分位 = 0）。

> The held-out season is seven events. That's enough to say the model didn't
> blow up out of sample; it is not enough to support a model-selection claim.
> If someone asks me to defend "Poisson beats negative binomial" on this data,
> I can't — and I wouldn't try.

### 2.2 baseline 是模型自己的一个特征（嵌套比较）

`model.py:98` 的 `seasonal_naive()` 直接 `return panel["expanding_mean"]`，
而 `expanding_mean` 就在 `config/models/m1.yaml` 的 lag 特征列表里。

> Worth being upfront about: the baseline is nested inside the model — the
> expanding mean is literally one of the GLM's features. So "beats baseline"
> isn't two independent methods competing; it's asking whether the extra terms
> buy anything on top of the naive one. In-sample the model essentially can't
> lose, which is exactly why the time-ordered holdout is the only comparison
> we report.

**这一条是全场最能证明「我懂」的。** 如果他自己看出来而我没意识到，
那是最坏的情况。

### 2.3 severity_score 与另外两个特征共线

见 §5.1。待验证，但构造上几乎确定。

> `severity_score` is a linear blend of two features already in the matrix, so
> the design matrix is near-rank-deficient. It doesn't hurt the predictions —
> which is what the scoring table consumes — but it does mean the individual
> coefficients aren't separately interpretable. And "interpretable coefficients"
> was one of our stated reasons for picking a GLM over a boosted tree, so
> that's a tension we own rather than one we've resolved.

### 2.4 权重是先验拍的，实测影响序与名义顺序相反

BO-6 的名义权重 0.40 / 0.30 / 0.30，实测影响是
**顺位 0.300 > 请求量 0.270 > 天气 0.167 分数单位**——顺序反了。

> The three weights are a prior, not a fitted result. We went back and measured
> what they actually do to the ranking, and the influence order comes out
> reversed from the nominal one. That changed how we're allowed to talk about
> it: we don't say "request volume is the dominant factor", because 0.40 is a
> nominal weight, not an influence.

这条把我从「定了三个数字」变成「定了三个数字，回头量了真实影响，
发现跟直觉相反，于是改了对外口径」。是**方法论成熟度**，不是统计知识。

---

## 3. 第一层：Poisson 本身（他已经问了一半）

### 3.1 为什么是计数模型

目标是「一次降雪事件中一个分区产生多少条冬季上报」——非负整数，
30% 是 0，最大 381。普通线性回归会预测负数，而且假设同方差。
Poisson 用 log link 保证非负，每个自变量是**乘法效应**：
系数取 `exp` 就是「这个因素让预期上报量乘几倍」。

> It's a count model. We're predicting how many winter-related resident reports
> a zone is likely to generate in one snowfall event, so we model the count
> directly rather than fitting a straight line to it.

### 3.2 为什么用 offset 而不是把目标除以地址数

`config/models/m1.yaml` 的原话：`log(address_count)` 作 GLM offset
**等价于建模「每地址请求率」，但保持目标仍是计数，所以 MAE 与实际工单量同量纲**。

除法会让 MAE 变成「每地址多少」——0.003 是好是坏没人判断得了。

`features.py:255` 是实现，`features.py:70` 的 `validate_panel` 拦
`unit_size <= 0`（取 log 会变 `-inf`，会在 statsmodels 深处炸而不是在数据层报错）。

**这条还连着 BO-6 的一个实测结论**：请求量因子的地址数分母是承重的——
去掉它 `r(顺位, 请求量)` 会从 +0.017 虚涨到 +0.139。

### 3.3 为什么不是 Negative Binomial 🔴 他下一个大概率是这个

**这条的答案比看上去强，因为它是预先注册的。**

`design §4.1` 原文：模型族 = Poisson GLM 作基线主模型；
**仅当留出季 MAE 不优于 seasonal-naive 时**，才试负二项 / 梯度提升，
且换模型必须换 `model_version` 并保留旧版预测（ADR 0010 D5）。

> We pre-committed to the escalation path before fitting: Poisson first because
> it's the simplest count model that respects the offset, with negative binomial
> and a GBM as the documented fallback if it lost to the baseline on the
> held-out season. It didn't lose, so we didn't escalate. The overdispersion is
> real and we know it — it's a reason our intervals are too narrow, not a reason
> the point estimates are wrong.

**要点：泊松假设均值等于方差。** 这批数据是零膨胀 + 过离散，
所以点预测还能用，**但不确定性被低估、置信区间偏窄**。
说清「它伤的是区间不是点估计」，比含糊承认「是有问题」强得多。

### 3.4 为什么不是 GBM

`design §8` 原文三条：目标是计数、面板只有 2,178 格、
**系数要能在幻灯片上解释**。

⚠️ 注意这第三条与 §2.3 的共线问题**互相打架**，要自己先说，见 §5.1。

---

## 4. 第二层：评估设计

### 4.1 嵌套 baseline

见 §2.2。**必须主动说。**

附带一条：`evaluate()`（`model.py:176`）在 baseline 未定义的行上
**同时**丢掉模型和 baseline 的那一行，不是只丢 baseline 的。
理由写在 docstring 里：在 baseline 看不见的行上给模型打分，
是拿两个行集比较然后叫它「比较」。

### 4.2 时序切分与泄漏

`assert_no_history_leak()`（`model.py:218`）不只检查两个集合不相交，
而是断言**每个训练事件的开始时间都严格早于最早的留出事件**。

> A random split passes a "these two frames share no rows" check and fails this
> one, and that difference is the whole point — the lag features would let the
> test season's own history predict itself.

滞后特征（`features.py:133`）用 `shift(1)`，严格只看同分区**更早**的事件。
扩展均值是在**已经 shift 过的**序列上做 expanding，不是反过来。

另一条容易被当成泄漏但不是的：`season_index` 用的是**全面板**的雪季序号。
docstring 已经预先辩护过：它只用事件自己的日期，预测时刻已知，
承载的是多年工单量增长——训练 2008–2025、打分 2026 的模型否则无法表达这件事。

### 4.3 为什么报两个指标

MAE 读得懂（多少张工单）；Poisson deviance 是泊松自己的似然尺度，
对零格和大值的处理才正确。只报 MAE 等于用高斯的眼光评价泊松。

`poisson_deviance()`（`model.py:155`）里 `y·log(y/μ)` 在 `y=0` 时取 0——
那是极限值不是特例，否则零格返回 NaN 会毒化整个均值。
**零格常见：1,298 格里 908 格非零，约 30% 正好是这个情况。**

### 4.4 "seasonal-naive" 是错名 🔴 主动纠正

日志里写 "seasonal-naive"，实现是**因果扩展均值**（causal expanding mean）。
已记在 L3 launch §3.3。主动说出来 = 自己发现的口径错误，只会加分。

### 4.5 实测数字

留出季 2025-2026：**MAE 7.345 vs 基线 23.628**，deviance 8.150 vs 36.757。

🔴 **但「模型优于基线」不是可辩护的公开结论**（BO-8 §0.2.2）。
三条保留必须跟结论一起讲：留出季只有 7 个事件、目标高度零膨胀、
baseline 与模型嵌套。

---

## 5. 第三层：特征矩阵

### 5.1 🔴 severity_score 的构造性共线（待验证，见 §7）

`sql/dml/dim_snowfall_event.sql:55-65`：

```
severity_score = 0.5 · norm(total_snowfall_cm)
               + 0.5 · norm( cold_c )

cold_c = GREATEST(0.0, -min_temperature_c)     -- 同文件 :32
```

而 `config/models/m1.yaml` 的事件特征里**同时有**
`total_snowfall_cm`、`min_temperature_c`、`severity_score`。

min-max 归一化是**仿射变换**；`GREATEST(0, ·)` 只在气温 > 0 时引入非线性——
而面板是 Winnipeg 的 Nov–Apr。**所以在面板绝大多数行上，
`severity_score` 是另外两个特征的精确线性组合。**

后果要说准：

| | |
|---|---|
| ✅ 预测 | **不受影响**。共线性不伤预测。别慌 |
| 🔴 系数 | **不可单独解释**——而这正是选 GLM 而不是 GBM 的理由之一（`model.py:7`）|
| 🔴 标准误 | 膨胀，任何 p 值都不能信 |

处置建议（如果要动）：把 `severity_score` 从 M1 特征集里拿掉，
只留它作 BO-6 的 `weather_severity_factor`——那里它是单独使用的。
**但 schema 冻结，动之前走变更流程。**

### 5.2 accum_flag 近似可分 🟢 主动提这条会加分

`model.py:121` 把 `maxiter` 从默认提到 200，注释写的是
`accum_flag` 在短面板上**近似可分（near-separable）**、收敛慢而不是不收敛；
并且显式检查 `results.converged`，不收敛就 `raise` 拒绝发系数。

> There's a near-separable indicator in there — `accum_flag`, which marks the
> events that only exist because of the rolling-accumulation rule. It converges
> slowly rather than not at all, so we raised maxiter and we check `converged`
> explicitly, because a silent non-convergence would be reported as a fitted
> model.

主动讲这条 = 证明「有人在盯数值行为」，不是调库调出来的。

### 5.3 什么被**故意**排除在特征之外

`features.py:49` 的 `FORBIDDEN_FEATURES`：`shift_number` / `rank_factor` /
`rank` / `unit_rank` / `plow_zone_rank`。有单测断言它们不在设计矩阵里，
`feature_names()` 在读 YAML 时就拒绝。

理由：顺位是 BO-6 的**独立第二项**（权重 0.30），
喂进 M1 等于把它偷偷混进第一项（权重 0.40），
三项独立性的实测结论 `r(顺位, 请求量) = +0.017` 随即失效。

> We deliberately keep schedule order out of the model. It's the second,
> independent term in the load score, and feeding it to the demand forecast
> would smuggle a 0.30-weight factor into the 0.40-weight one — which would
> void the independence result the whole three-factor form rests on.

**这条很强**：它展示的是「知道两个组件之间有什么不能流动」，
是系统级的统计判断，不是调参。

### 5.4 首个事件的处理

每个分区的第一个事件没有滞后特征。**丢弃而不是插补**
（`features.py:168`）：填进去的「上一次工单数」模型没法与真值区分。
代价是 2,178 格里丢 22 格，全在 2008 年，远早于排班期——
所以预测面板一格不少，而这一点是 `assert_prediction_panel_has_history()`
**检查**出来的，不是假设的。

---

## 6. 第四层：BO-6 那个公式

```
scored          : load_score = 100 × (0.40·rf + 0.30·rk + 0.30·w)   上限 100
partial_no_rank : load_score = 100 × (0.40·rf +           0.30·w)   上限  70
```

| 他会问 | 答 |
|---|---|
| 权重哪来的 | 先验，不是拟合。见 §2.4——主动说影响序反了 |
| 三项独立吗 | 两两 |r| 最大 +0.460（请求量 × 天气）；`r(顺位, 请求量) = +0.017`、`r(顺位, 天气) = −0.006`。删掉顺位项在 15/15 个事件上都改变分区排序 |
| 归一化范围 | min-max 跑**全面板 1,298 格**，不是逐事件。逐事件会让每个事件里恰好有一个 1.0 和一个 0.0，跨事件比较随即失去意义——而 `load_level` 正是跨事件比的 |
| 缺顺位怎么办 | NULL，**不填 0**。「没有排班记录」推不出「排在最后」（ADR 0008 §2.3）。分数走 `demand_weather_only` profile，上限 70，**不重归一化** |
| 等级阈值 | 固定四分位 × 各自上限 C，不是数据分位数。数据分位数会让新增一个事件把历史事件的等级顶下去，幂等性就没了 |

🔴 **两个 profile 的 `load_level` 不可比——同名不同尺。**
`demand_weather_only` 的 CRITICAL 门槛是 **52.5 不是 75**。

🔴 **不得说 `rank_delta > 0` 是「模型优于基线」**——它是位移。
同事件内两个排名都是 1..22 的排列，位移和恒为 0；
故意训坏的 `nomonth` 版本同样有 188 格上移。

---

## 7. 待验证清单（拿到线上环境就跑）

### 7.1 🔴 共线程度 —— 优先级最高

```sql
SELECT count(*)                                  AS n_events,
       count_if(min_temperature_c > 0)           AS n_above_zero,
       corr(severity_score, total_snowfall_cm)   AS r_sev_snow,
       corr(severity_score, min_temperature_c)   AS r_sev_temp
FROM uoip_gold.dim_snowfall_event;
```

`n_above_zero` 接近 0 ⇒ 共线确定，§2.3 照原文说。
若显著非 0，非线性有真实作用，答法要收敛成「partially collinear」。

更直接的一步（需要 `.venv-ml`）：对 `[const, total_snowfall_cm,
min_temperature_c, severity_score]` 算条件数（condition number）或
逐列 VIF。条件数 > 30 是教科书阈值。

### 7.2 拒绝率

`silver/_rejects/service_request/` 的行数。记忆中是 0，**要复核**——
「0 rejected rows out of 12.47 M」是一句很强的话，但说错代价高。

### 7.3 架构图那 69 个从未被念过的标签

2026-09-19 审计：架构图 73 个标签，**69 个在整个 deck 里从没被念过一次**。
`Poisson GLM · live` 只是其中一条被问到的。

根因：那行字**在 pptx 里不是文字，是像素**——出处是
`docs/images/platform-architecture.svg`，以 PNG 贴进第 22 页。
任何基于文本的讲稿覆盖检查都结构性地跳过它。

**流程修正**：图上的字要从**源文件**（`.svg` / `.drawio.xml` / 生成脚本）
抓出来对讲稿，不能从 pptx 抓。讲稿没覆盖的标签，要么删掉，要么补备答，
不允许第三种状态。

---

## 8. 六个概念的最小定义

真正用到的就这些。**别看统计教材——读
`models/request_forecast/model.py` 和 `features.py` 的 docstring，
491 行，每条都绑着我们自己的数字，覆盖下面六条里的五条。**

1. **计数模型 / log link / offset** — 目标是非负整数；log link 保非负；
   offset 是系数固定为 1 的自变量，用来表达「按规模归一」
2. **零膨胀与过离散** — 泊松假设均值 = 方差；违反时**点估计还行，区间偏窄**
3. **嵌套模型（nested）** — 一个模型包含另一个作为特例；
   比较的含义变成「额外的项有没有加东西」
4. **时序切分与泄漏** — 随机切分会让测试期的历史预测自己
5. **多重共线性** — 伤解释、不伤预测；诊断用 VIF / 条件数
6. **Pearson r vs Spearman ρ** — 前者测线性、后者测单调。
   BO-2 的顺位稳定性用的是 ρ（前后半期 ρ = +0.591）

---

## 9. 出处索引

| 主张 | 出处 |
|---|---|
| Poisson GLM + log(unit_size) offset | `models/request_forecast/model.py:7`、`features.py:255` |
| offset 必须为正的守卫 | `features.py:70` |
| baseline = expanding_mean | `model.py:98`、`model.py:118` |
| maxiter / 收敛检查 | `model.py:121`、`model.py:131` |
| deviance 的零格处理 | `model.py:155` |
| 两侧同时丢行 | `model.py:176` |
| 泄漏断言 | `model.py:218` |
| 滞后特征的 shift(1) | `features.py:133` |
| 禁用特征 | `features.py:49`、`features.py:200` |
| 首事件丢弃而非插补 | `features.py:168`、`features.py:182` |
| 特征清单 / 切分 / 指标 | `config/models/m1.yaml` |
| severity_score 公式 | `sql/dml/dim_snowfall_event.sql:32`、`:55-65` |
| 模型族定案与升级路径 | `docs/dev/design/20260820-scoring-chain-and-m1.md` §4.1 |
| 回测口径、不得说消费预报 | 同上 §4.3 |
| 被否决的选项表 | 同上 §8 |
| 对外表述边界 | 同上 §9 |
| 留出季实测、三条保留 | `docs/dev/launch/20260820-scoring-chain-and-m1-launch.md` §3.3 |
| 四条禁语 | 同上 §8 |
