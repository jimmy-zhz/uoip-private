# Q&A 口径库 — Day of Data 现场备稿

> 🔴 **迁自 ToucanShelf `UOIP/presentation/QA`（2026-09-20）。权威版本从此在这里。**
> 按 2026-09-20 重定的边界（见 [`README.md`](README.md)），它是**技术口径底稿**
> 而不是演讲交付物：里面是禁语、保留条件、冻结的 audit run id，全都必须跟仓库的
> 数字同步。`SpeakerNote` / `HowToDeliver` / outline 留在 ToucanShelf——那些是
> 「怎么讲」，这篇是「能讲什么、不能讲什么」。
>
> 🔴 **这份稿子在 2026-09-19 当天没派上用场，原因记在
> [`20260919-four-questions.md`](20260919-four-questions.md)**：
> 它按「没有技术背景的大学生」写了 30 问，到场提问的是同行。
> **听众画像设错，备了三十发子弹一发没用上。** 下次改口径前先改这一行假设。
>
> 统计层面的追问不在本篇覆盖范围，见 [`m1-defence.md`](m1-defence.md)。

---

本文用于现场 Q&A 备讲。问题按没有技术背景的大学生可能使用的表达来写，不假设提问者熟悉数据工程、统计建模或市政运营。

每个问题包含三部分：中文回答方向、可引用数据，以及可以直接说出口的简短英文回答。现场不必把所有数字都讲出来；先回答主旨，被继续追问时再补数字。

## Useful English phrases

- Could you please repeat the question more slowly?
- If I understand correctly, you are asking whether...
- The short answer is yes, but with an important limitation.
- The short answer is no. The data measures something different.
- I may need to check the exact number, but the main point is...
- The current data cannot tell us that yet.
- That is a good question, but it is beyond what we tested in this project.
- Temi may be better placed to speak about the future direction.
- Cesario may be better placed to speak about the Winnipeg context.

## Question 1 What problem is this project trying to solve

**Student question**

> What problem are you trying to solve?

**回答方向**

从生活问题开始：同一场雪后，为什么不同地区看起来等待时间不同。项目没有声称已经找到原因，而是先把降雪、居民上报和公开排班放到同一时间与空间中，判断这些资料分别能回答什么。

**可引用数据**

- 项目最终比较的主要单位是 snowfall event 和 plow zone。
- 排班分析覆盖 19 次 city-wide operations 和 22 个有排班记录的 zones。

**Short English answer**

We started with a simple Winnipeg winter question: why might one area seem to wait longer after snowfall? We connect weather, resident reports, and the public clearing schedule so that we can separate what each source tells us. The project does not claim that we have already proved the cause.

## Question 2 Why did you choose snow clearing

**Student question**

> Why did you choose snow clearing as the topic?

**回答方向**

这是 Winnipeg 学生和居民都能理解的日常问题，同时又能很好地展示一个数字可能代表计划、观察或公众反应，而不是同一件事。这个案例让抽象的数据质量问题变得具体。

**可引用数据**

- 无需主动报数字。

**Short English answer**

Snow clearing is something people in Winnipeg understand from daily life. It also gives us a clear example of why a number can look simple while representing a plan, a report, or a weather measurement. That makes the data problem easier to explain.

## Question 3 Did you measure how long each road waited

**Student question**

> Did you measure how long each road waited to be cleared?

**回答方向**

明确回答没有。公开排班中的时间是计划班次的开始和结束，不是每条道路实际完成的时间。项目因此把问题从 actual completion time 改成 planned order。

**可引用数据**

- 每个计划班次都是 12 hours。
- 同一班次内的区域具有相同的计划开始和结束时间。

**Short English answer**

No. The public dataset gives us planned shift times, not the actual completion time for every road. We can compare where an area appeared in the plan, but we cannot say exactly when its roads were finished.

## Question 4 Why is a schedule not the same as a result

**Student question**

> Why can't you use the end of a shift as the time the work was finished?

**回答方向**

用 timetable 类比。课程表的结束时间不代表每个学生在那一刻完成作业；同理，计划班次结束不代表区域内每条道路都在那个时间完成。

**可引用数据**

- `shift_end_utc - shift_start_utc` 在排班记录中恒为 12 hours。
- 完整排班面板为 19 operations × 22 zones = 418 rows。

**Short English answer**

It is like a class timetable. A class ending at noon does not tell us when every student finished an assignment. In the same way, the planned end of a clearing shift does not prove when every road was completed.

## Question 5 Did some areas usually appear earlier than others

**Student question**

> Did you find that some areas were usually scheduled earlier?

**回答方向**

回答是，但强调这是平均计划顺序，不是道路实际完成速度。可以用 Zone S 和 Zone C 的差别说明，同时补一句大多数区域在不同作业中都会移动。

**可引用数据**

- Zone S mean shift = 1.26。
- Zone C mean shift = 3.47。
- 差值为 2.21 shift positions，按每班 12 小时约为 26.5 planned hours。
- 21 of 22 scheduled zones 至少有一次进入 shift 1。

**Short English answer**

Yes. For example, Zone S averaged about shift 1.3, while Zone C averaged about shift 3.5. That is a difference in planned order, not proof that every road in Zone C was completed a day later.

## Question 6 Was the order always the same

**Student question**

> Were the same areas always first or last?

**回答方向**

回答不是。平均值显示长期模式，但每次 operation 都可能变化。Zone C 平均较晚，却也曾经排到 shift 1；Zone K 是唯一从未进入 shift 1 的 scheduled zone，但它也不是平均最晚的区域。

**可引用数据**

- 21 of 22 zones 曾进入 shift 1。
- Zone K 的观测范围是 shifts 2–3。
- Zone V 和 M 在后期平均后移 1.31 和 1.02 个班次。

**Short English answer**

No. The averages show a pattern, but the order changed between operations and over time. Even Zone C, which was latest on average, appeared in the first shift at least once.

## Question 7 What is the difference between a ward and a plow zone

**Student question**

> What is the difference between a ward and a plow zone?

**回答方向**

Ward 用于选举市议员，plow zone 用于组织除雪作业。它们覆盖同一座城市，但服务于不同工作，因此边界不需要重合。

**可引用数据**

- Winnipeg 有 15 wards。
- 城市边界图中有 25 plow zones，其中 22 个出现在当前排班资料中。

**Short English answer**

A ward is used for city representation, while a plow zone is used to organize snow-clearing work. They divide the same city for different purposes, so their boundaries do not line up.

## Question 8 Why does using the wrong map matter

**Student question**

> What would go wrong if you used wards instead of plow zones?

**回答方向**

可能把属于其他作业区的居民上报算进来，同时漏掉真正属于目标作业区的上报。计算过程仍然可能成功并生成漂亮图表，但回答的是错误空间单位的问题。

**可引用数据**

- 只有 2 of 25 plow zones 完整落在单一 ward 中。
- 10 of 25 zones 没有一个 ward 占其冬季工单的多数。
- Zone V 跨 10 wards，最大 ward 份额为 26.0%。

**Short English answer**

We could count reports from the wrong work area and miss reports that belong to the area we are studying. The computer might still produce a clean total, but the total would answer the wrong geographic question.

## Question 9 How did you decide what counts as one snowfall event

**Student question**

> How did you decide where one snow event starts and ends?

**回答方向**

解释天气资料按天记录，并不会自动标出 storm boundary。项目采用可重复执行的规则：单日达到 3 cm，或连续 10 天累计达到 10 cm；相邻雪日允许一个空档日连接。

**可引用数据**

- Rule name: `v1-3cm-or-10d10cm`。
- Single day ≥ 3 cm OR trailing 10 days ≥ 10 cm。
- `gap_days = 1`。
- 该规则得到 99 events，覆盖 18 winters。

**Short English answer**

The weather data gives us one row per day, so it does not draw a boundary around a storm. We used a repeatable rule based on either three centimetres in one day or ten centimetres across ten days. The rule is useful, but it is still a definition created for the analysis.

## Question 10 Why did you change the snowfall rule

**Student question**

> Why did you change your original snow-event rule?

**回答方向**

原规则只看单日降雪，会漏掉连续多天的小雪累积。促成修改的证据是：那些没有降雪事件却出动犁雪的日子，21 日累计降雪保留了对照组的 76%，而单日峰值只保留 26%——雪照样下了同量级，只是从没有一天超过 3 cm。据此加入 rolling accumulation 条件。

🔴 **不要说这条规则把那些无降雪犁雪的案例找回来了。** 8 个 accumulation-only 事件与那些案例的对应关系从来没有量过，而且改完规则后仍有 2 次犁雪没有对齐上任何降雪事件。可以讲的是修改的理由和保留的开放案例，不是修改的战果。

**可引用数据**

- 累积条件识别出 8 accumulation-only events。
- 排班对齐结果为 17 matched 和 2 unmatched。
- 8 个累积事件解释了几次未对齐犁雪，**没有测量过**。

**Short English answer**

The first rule only looked at single days, so it could miss several small snowfalls that added up over many days. We changed it because the records showed that snow really had fallen, just never more than three centimetres on any one day. We have not measured how many of the unmatched cases the new rule explains, and two are still open.

## Question 11 What do the three numbers on the winter street mean

**Student question**

> What do 14.6 centimetres, 43 reports, and shift 4 mean?

**回答方向**

三个数字描述同一个 snowfall event 和同一个 zone，但回答三个不同问题。14.6 cm 是事件降雪量，43 是居民冬季相关上报数量，shift 4 是公开计划中的排班位置。它们不能合起来证明实际路况或等待原因。

**可引用数据**

- Case: `SNOW-20241208 × Zone V`。
- Total snowfall = 14.6 cm。
- Winter resident reports = 43。
- Planned position = shift 4。

**Short English answer**

They describe the same event and the same work area, but they answer different questions. The snowfall was 14.6 centimetres, residents made 43 winter-related reports, and the area was planned into shift 4. Together they give context, but they do not prove the road condition or the cause of a delay.

## Question 12 Do more reports mean worse roads

**Student question**

> If an area has more resident reports, does that mean its roads were worse?

**回答方向**

回答不一定。报告数同时受人口、报告习惯、服务认知和记录方式影响。项目缺少 independent road-condition ground truth，因此只能说模型或分数描述 reported demand，不能把它叫作真实路况。

**可引用数据**

- 无需主动报数字。
- 模型目标是 winter-related resident reports，不是 road condition。

**Short English answer**

Not necessarily. More reports may reflect more problems, but they may also reflect population or reporting behaviour. We treat the number as reported demand, not as a direct measurement of road condition.

## Question 13 What is the Load Score

**Student question**

> What is the Load Score in simple terms?

**回答方向**

把它解释为回看历史事件的综合视图，将居民上报、计划顺序和天气严重程度放在一起，帮助比较 event-zone cases。它不是实时路况分数，也不是官方城市决策工具。

**可引用数据**

- Nominal weights: demand 0.40, planned rank 0.30, weather 0.30。
- Full panel = 59 events × 22 zones = 1,298 cells。

**Short English answer**

The Load Score is a way to review past cases using three pieces of information: resident reports, planned order, and weather severity. It helps us organize the evidence, but it is not a live road-condition score or an official City decision tool.

## Question 14 Why are some scores called partial

**Student question**

> Why do some events have only a partial score?

**回答方向**

因为只有部分 snowfall events 能匹配到排班资料。没有 planned order 时只能使用居民上报和天气，不能把缺失排班误当成低值，也不能把 partial 与 complete score 直接放在同一尺度比较。

**可引用数据**

- 17 of 59 events 有三项完整输入。
- 42 of 59 events 没有匹配的 schedule data。
- 374 complete cells；924 partial cells。
- Complete ceiling = 100；partial ceiling = 70。

**Short English answer**

Only 17 of the 59 events have all three inputs. The other 42 do not have matching schedule data, so they use a different partial scale. Missing schedule information must not be interpreted as a low score.

## Question 15 What does the AI model predict

**Student question**

> What exactly does the AI model predict?

**回答方向**

准确限定目标：根据过去的 snowfall event information 和 zone history，估计每个 zone 可能出现的冬季相关居民上报数量。它不预测道路完成时间、不测量真实路况，也不替城市选择路线。

**可引用数据**

- Prediction target: estimated winter-related resident reports per event and zone。
- Model type: Poisson regression。如无追问不必主动讲模型名称。
- Full panel = 1,298 event-zone cells。

**Short English answer**

It estimates the number of winter-related resident reports for each work area during a snowfall event. It does not predict when a road will be finished, measure road condition, or choose routes for the City.

## Question 16 Is the model accurate

**Student question**

> How accurate is the model?

**回答方向**

先说有历史测试结果，但证据仍小，不宣传模型已经成功。解释 MAE 是平均误差且越低越好。

🔴 **一旦把 baseline 的 23.628 说出口，三条保留必须一起说**，否则就变成了「模型优于基线」这个不可辩护的结论：① 留出季只有 7 个事件；② 目标高度零膨胀，30.0% 为 zero、25 分位为 0.1，这种分布下 MAE 的差距容易被夸大；③ 对照实现本身还很弱，日志里叫它 seasonal-naive 是错名，实际是因果扩展均值。**更稳妥的做法是不主动报 baseline 数字**，只讲模型自身的误差量级和样本太小。

**可引用数据**

- Held-out test = 7 events × 22 zones = 154 cells。
- Model v1 MAE = 7.345。
- Model without month input MAE = 7.919。
- Baseline MAE = 23.628（被追问才给，且必须带上面三条保留）。
- Full target distribution 中 30.0% 为 zero，25 分位 = 0.1，maximum = 381。

**Short English answer**

In the historical test, the model's mean absolute error was about 7.3 reports. The test contained only seven snowfall events, and most areas report zero or almost zero, so that number is easy to over-read. Our simple comparison is also still weak, so we do not claim the model beats a baseline. We describe it as a working prototype rather than a validated planning tool.

## Question 17 Why are seven test events not enough

**Student question**

> Why can't you trust the model after testing seven events?

**回答方向**

七个事件不能代表 Winnipeg 所有冬季情况，尤其极端风暴、微量积雪和不同年份可能差别很大。虽然每个事件有 22 个 zones，但这些 zones 共享同一场天气，不等于 154 场独立测试。

**可引用数据**

- 7 held-out events × 22 zones = 154 cells。
- 154 cells 不等于 154 个独立 snowfall events。

**Short English answer**

Seven events cannot represent every kind of Winnipeg winter. Although those events produce 154 area-level rows, the rows within one event share the same storm. We need more independent events before making a decision-level claim.

## Question 18 Can the City use this system now

**Student question**

> Could the City use this system to plan snow clearing now?

**回答方向**

回答目前不应该作为城市规划工具。现在完成的是 historical back-test；缺少实际道路完成时间、独立路况测量、更大的验证样本和雪前 forecast end-to-end process。可以说它适合作为研究原型和证据整理工具。

**可引用数据**

- 当前验证仅 7 held-out events。
- 当前模型使用 historical measured weather。
- 使用 weather forecast 的雪前完整流程仍是 next step。

**Short English answer**

Not as a planning tool yet. It works as a historical prototype, but we still need more test events, actual operational outcomes, and a complete process using weather forecasts before snow arrives.

## Question 19 What is the platform for if the model is not ready

**Student question**

> If the AI is not ready for decisions, what is the platform useful for?

**回答方向**

平台的价值不只在模型。它保存原始资料、统一格式、连接时间与空间、运行检查并保留结果来源；新数据可以重复走同一过程，结果被质疑时可以回到原始记录和规则。

**可引用数据**

- 约 12.5 million service-request rows。
- 17 published Gold tables。
- 平台为 self-hosted two-node deployment。
- 数据质量框架有 39 rules，展开为 daily 83 checks，另有 4 weekly roll-ups。

🔴 **12.5 M 与现场说出口的「1800 万」不是矛盾，是口径错位**——前者是本平台采集并处理的
Silver 行数，后者是上游整表。两个数都真，混用就会被抓。展开见
[`20260919-four-questions.md`](20260919-four-questions.md) 问题一。

**Short English answer**

The platform is useful even before the model is ready. It keeps the original records, applies the same cleaning and matching process, runs checks, and lets us trace a result back to its evidence. That makes the work repeatable and checkable.

## Question 20 What was the hardest part of the project

**Student question**

> What was the hardest part of the project?

**回答方向**

不要回答技术部署最难。更贴合演讲的是：最难的是确认字段、地图和事件实际上代表什么。数据能成功连接或代码能运行，不代表业务问题已经被正确表达。

**可引用数据**

- 可举三个发现：12-hour planned window、15 wards vs 25 zones、99 rule-defined snowfall events。

**Short English answer**

The hardest part was understanding what the records really represented. A field that looked like completion time was only a planned window, and two valid maps described different kinds of areas. The code can run while the question is still wrong.

## Question 21 What data would improve the project most

**Student question**

> What additional data would help you most?

**回答方向**

优先回答 actual completion records 和 independent road-condition measurements。这两类资料能帮助区分计划与执行，并检验居民上报是否真的对应路况。之后才是更丰富的实时或 forecast inputs。

**可引用数据**

- 当前 schedule 只有 planned shift order。
- 当前 resident reports 不是 independent road-condition ground truth。

**Short English answer**

The most useful additions would be actual road-completion records and an independent measurement of road condition. Those would help us compare the plan with what happened on the street. Better forecast inputs would also help us test the system before an event.

## Question 22 What would you do next

**Student question**

> What is the next step for the project?

**回答方向**

给出三步即可：扩大跨冬季验证；增加更强且定义清楚的 baseline；把 weather forecast 接入雪前 end-to-end process。如果能获得 actual completion 或 road-condition data，再重新定义更接近运营结果的目标。

**可引用数据**

- 当前 held-out set 只有 7 events。
- 当前使用 measured historical weather，而不是 operational forecast input。

**Short English answer**

The next step is to test across more winters, compare the model with stronger baselines, and run the full process using weather forecasts before snow arrives. If actual completion or road-condition data becomes available, we could also test a more operationally meaningful target.

## Question 23 Could this approach be used in another city

**Student question**

> Could another city use the same approach?

**回答方向**

方法可以迁移，但规则和数据含义不能直接复制。另一个城市需要重新确认自己的排班字段、空间单位、天气条件和服务流程；平台思路可复用，具体结论不可照搬。

**可引用数据**

- 无需报项目数字。

**Short English answer**

The general approach could be reused, but the exact rules and conclusions could not simply be copied. Another city would need to check the meaning of its own schedules, maps, weather data, and service process.

## Question 24 How can students use the lesson from this project

**Student question**

> What can students take away from this project?

**回答方向**

把结论转成三个普通学习场景也能使用的问题：这个字段是计划还是结果；两张图的边界是否描述同一对象；无法匹配的记录为什么无法匹配。强调先理解记录，再计算答案。

**可引用数据**

- 无需报数字。

**Short English answer**

Before calculating, ask three questions. Does this field describe a plan or a result? Do these maps describe the same kind of area? And why do some records not match? Understanding the record is part of solving the problem.

## Question 25 Is the project public

**Student question**

> Can we see the project or the data ourselves?

**回答方向**

说明项目代码和复现路径在公开仓库中，主要输入来自 City of Winnipeg Open Data、Open-Meteo historical archive 和公开政策文件。提醒冻结结果对应特定构建，外部资料以后可能修订。

**可引用数据**

- Repository: `github.com/huzhi-zhao/urban-ops-intelligence-platform`。
- 当前演讲图表绑定 audit run `dq-20260908T083001-70e601`，该轮为 0 errors 和 0 warnings。

**Short English answer**

Yes. The project code is available in the public repository shown on the final slide. The main inputs come from City of Winnipeg Open Data, the Open-Meteo historical archive, and public City policy documents.

## Question 26 Did you use personal information

**Student question**

> Does the project use people's personal information?

**回答方向**

谨慎回答项目分析对象，不替公开数据平台作超出材料的隐私保证。我们使用公开的服务请求记录，并把分析聚合到 snowfall event 和 plow zone；研究关注数量、类别、时间和可用坐标，不识别或分析具体个人。

**可引用数据**

- 分析单位为 event-zone aggregation。
- 79% requests 没有 coordinates；空间统计只在有坐标的目标子集中计算。

**Short English answer**

We used public service-request records and analyzed them in groups by event and work area. The project focuses on counts, categories, time, and available location information. We do not try to identify or study individual residents.

## Question 27 How do you know the results can be checked

**Student question**

> How can someone check where a number came from?

**回答方向**

用 keep the receipts 的说法。原始资料被保存，清洗、匹配、查询和图表有固定路径；结果带 build information，公开仓库中保留 source SQL 和冻结输入。检查通过只代表已定义规则通过，不代表所有未知都解决。

**可引用数据**

- 17 published Gold tables。
- 演讲使用 23 frozen figures，绑定同一次 certification run。
- Certification run: `dq-20260908T083001-70e601`，0 errors，0 warnings。

**Short English answer**

The process keeps the original records and the steps used to clean, match, and calculate each result. The figures are tied to one certified build, and the source queries are available in the project repository. This lets us trace a number back to its evidence.

## Question 28 Does more snow cause more reports

**Student question**

> Does heavier snow cause more resident reports?

**回答方向**

不要接受 cause 这个词。可以说降雪量与报告数可能有关，也是模型输入的一部分，但现有观察资料不能单独建立因果关系。人口、报告习惯、道路类型、时间和其他天气条件都可能影响上报。

**可引用数据**

- 当前模型预测 reports，不做 causal identification。
- Weather value 在同一 event 的各 zones 间相同，不能解释同场雪中所有 zone-level differences。

**Short English answer**

Heavier snow may be related to more reports, but our project does not prove that it causes them. Reporting behaviour, population, road type, and other conditions may also matter. We treat snowfall as one piece of evidence rather than a complete explanation.

## Question 29 Why not remove every unmatched record

**Student question**

> Why not change the rule until every record matches?

**回答方向**

如果为了得到整齐结果而不断改规则，会隐藏真实的数据边界并产生 overfitting。规则应当在有外部或记录证据时修改；没有足够证据的案例应继续标记为 unknown。这里讲「我们没有一直改到全对齐」，不要讲「改完救回了几个」——那个数没有量过，见 Question 10。

**可引用数据**

- 最终仍有 2 unmatched schedule cases。
- 累积规则是在有记录证据时才改的，改完仍保留 2 个开放案例。

**Short English answer**

If we keep changing the rule until everything matches, we may only be hiding the uncertainty. We change a rule when the records give us a reason. When the evidence is still missing, it is more honest to keep the case open.

## Question 30 What is the main message of the presentation

**Student question**

> What is the one main idea you want us to remember?

**回答方向**

回到全场主旨：在计算之前先确认数据代表什么。一个准确计算也可能回答错误问题；承认当前资料不能回答的部分，是可靠分析的一部分。

**可引用数据**

- 无需报数字。

**Short English answer**

Understand the record before asking it for an answer. A calculation can be accurate and still answer the wrong question. Saying what the data cannot tell us is part of doing good analysis.

## Quick routing during Q and A

- **Project meaning, presentation message, and future direction**: Temi can lead.
- **Winnipeg winter context and the opening story**: Cesario can lead.
- **Data, numbers, limitations, platform, and implementation**: James can lead.
- Temi can repeat or simplify the question before handing it to another speaker.
- If the question requires evidence the project does not have, use: **The current data cannot tell us that yet.**
- If an exact number is uncertain, use: **I may need to check the exact number, but the main point is...**

---

## 🔴 这份稿子没覆盖的，2026-09-19 实测

现场四个问题里，**这 30 问一个都没对上**。差距不在数量，在层次：

| 本篇假设的提问者 | 实际提问者 |
|---|---|
| 没有技术背景的大学生 | 同行 / 疑似咨询公司资深从业者 + 合讲的 professor |
| 想知道「这在解决什么问题」 | 想知道「这些选型是谁做的、依据是什么」 |
| 追问到数字就停 | 追问到**方法论与代价** |

下一版必须补的两类，本篇结构上都没有位置：

1. **架构图上每一个标签都可能被问。** 实测 73 个标签，讲稿覆盖 4 个。
   图里的字是像素不是文本，任何基于讲稿文本的覆盖检查都结构性地跳过它们。
2. **统计选型的代价。** 「为什么是 Poisson GLM」这类问题，答对术语不够，
   要能说出自己方案的代价。见 [`m1-defence.md`](m1-defence.md)。
