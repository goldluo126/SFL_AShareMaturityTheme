# SFL-1 v7 → v8 核对、RCA 与修复裁决

> 性质：非规范性 Review/RCA。正式规则以 `SFL-1_v8.md` 为唯一 Canonical 来源。
> 输入：外部对 v7.0 的 14 层深度 Review（A-01~A-14、B-01~B-08 及 Closure Pack 1–8）。

## 一、逐条核对结论

对 Review 全部 A/B 阻断与 v7 原文逐条核对。**结论：下列项全部属实；v6 已 CLOSED 项未复发。**

| 编号 | 指控 | v7 原文证据 | 核对结论 |
|---|---|---|---|
| A-01 | 高重叠题材簇算法不唯一 | §5.3 仅 `J≥0.60 → 同簇`，Jaccard 非传递，无连通分量/完全连接选择 | **属实** |
| A-02 | “确认质量最高”无公式/tie-break | §5.3 只写文字；伪代码 `deduplicate_theme_clusters` 在 IES/IDS 之前 | **属实** |
| A-03 | `LeaderAmountShare` 未定义 | §8.5 IES 使用该符号，全文无公式 | **属实** |
| A-04 | 确认后 IES/IDS 冻结 vs 动态不唯一 | §8.1 仍写 \(B_{u,D-1}\L_{u,D}\)；伪代码每日 `identify_attention_roles` | **属实** |
| A-05 | EPISODE_NULL 冻结 vs 每日 build | §8.3.3 冻结；§24/§25 `build_matched_nulls_v7(theme,D,...)` | **属实** |
| A-06 | NON_EVALUABLE vs LACDecay 优先级 | §12.0 “维持前一日”；§12.8 LACDecay→COLLAPSE；§9.2.3 又允许 LAC 退出 | **属实** |
| A-07 | Gate 7 事件日与 Gate 2 匹配日错位 | Gate 7 回引 §17.1（τ_confirm 日匹配器）用于主信号日 | **属实** |
| A-08 | Gate 7 未确认控制无合法 RMI/GCC/EntryScore | 控制“临时构造”、无 Temporary/确认角色冻结规则 | **属实** |
| A-09 | 对照篮子 TargetValue / 臂资本不唯一 | §0.3.2“与真实信号 TargetValue 相同”；Gate 6 四臂无总资本/分配规则 | **属实** |
| A-10 | two-way cluster / bootstrap seed 未完全冻结 | §20.2 只有装置名；无 CGM 公式、HC 修正、自由度、seed payload | **属实** |
| A-11 | 分钟成交量单位与 bar 语义未冻结 | §4.1 只写 vol/amount；§14.1 用 vol×5% 取整 100 股 | **属实** |
| A-12 | D 日预留与 09:35 资金再分配冲突 | §15 晚间预留 vs §24 09:35 统一排序；REJECT_GAP 后是否释放未定 | **属实** |
| A-13 | Gate 9 相同入场 tape 与有限资金矛盾 | Gate 9 要求相同入场序列，又暗示完整组合账本 | **属实** |
| A-14 | 成交率与收益集中度未定义 | §20.4/§20.5/§21.7 使用“成交率”“贡献”，无唯一公式 | **属实** |
| B-01 | Gate 7 不能识别过滤交互 | 同 A-07/A-08 | **属实** |
| B-02 | Gate 2 Whipsaw 两侧测量制度不对称 | 处理用冻结 B^E/L^E/EPISODE_NULL；控制可继续 DAILY_NULL | **属实** |
| B-03 | Gate 3 95% vs α_v=0.025 | §17.2 写单侧95%；§18.0/附录 B 写 1−α_v | **属实** |
| B-04 | 主聚类无有限样本/控制复用修正 | §17.1 控制跨日可复用；§20.2 无控制簇维 | **属实** |
| B-05 | Gate 4 附加条件未入/出正式法庭 | §17.3 三项 IUT 后又写两项附加通过条件 | **属实** |
| B-06 | Gate 6 域被写宽 | MatchedRandom 需 \|K^trade\|≥4；附录 B 写 ≥2 | **属实** |
| B-07 | Gate 9 不能同时保持相同入场与真实现金约束 | 同 A-13 | **属实** |
| B-08 | 绩效 Kill 联合误杀预算不成立 | §21.7 后两条称“不消耗序贯预算”但仍是随机收益阈值 | **属实** |

**附带确认（非独立编号但 Review 点名）：** Leg H 无操作对象；StockDayFeatureStore 缓存 theme-relative Rel；α-wealth 应写 ≤5% 而非 <5%；Track S/B 主法庭归属未闭合；P5 正式地位不清；REJECT_GAP 阈值日内切换边界未冻。

**未采纳为阻断（已 CLOSED / 非错误）：** Review 第二节已 CLOSED 的 v6 项不再重开；冻结 K、分钟参与≠真实排队、P10 5%/20 先验等非阻断限制仅登记。

## 二、根因分析（收敛为 4 个接口根因）

### RC-1：章节内闭包 ≠ 跨章节对象同构

v7 对每个局部对象（null、Gate 2 matcher、状态机、执行）各自写完整，但对“同一正式对象在不同章节是否同一测量制度”缺少映射表。典型：确认后 IDS 集合、EPISODE_NULL load-or-build、Gate 7 事件时钟。
**处置：** v8 增加唯一时间映射表与 load-or-build 协议；禁止伪代码与正文语义分叉。

### RC-2：对照生成器被“名字复用”而非“事件复用”

Gate 7 复用 Gate 2 生成器名字，但事件时点不同；Gate 2 控制侧未冻结测量 cohort。根因是把“真实题材对照”当成可跨研究共用的黑箱。
**处置：** 每个研究装置独立定义风险集、冻结日、角色/null；禁止跨事件时点回引。

### RC-3：统计法庭有判词边界、无算法身份

PASS/FAIL/α_v 已有，但 two-way cluster 公式、seed payload、guardrail vs IUT、控制复用依赖未冻结，导致判词可因实现细节翻转。
**处置：** 新增正式推断算法规范；统一全部 Gate PASS 为单侧 1−α_v；明确 guardrail 冲突→INCONCLUSIVE。

### RC-4：组合账本与科学 estimand 混写

Gate 9、对照篮子 TargetValue、09:35 资金释放把“机制假想账本 / 生产有限资金账本 / 外生 tape”写在同一语义层。
**处置：** 显式分流三套账本；Gate 9 选择纯退出增量（外生冻结入场 tape + 隔离资本）；生产政策另报端到端。

## 三、v8 修复裁决摘要（Closure Pack 对齐）

| Pack / 编号 | v8 裁决 | 位置 |
|---|---|---|
| Pack1 / A-01,A-02 | 题材簇=Jaccard 图连通分量；ConfirmQuality=min(IES,IDS^θ)；tie-break 固定；先算质量再去重选代表 | §5.3 |
| Pack2 / A-03,A-04 | LeaderAmountShare 公式；确认前/日/后 IES·IDS·null·角色唯一映射；确认后只用冻结 B^E/L^E | §3.5、§8.1、§8.5 |
| Pack3 / A-05,A-07,A-08,B-01,B-02 | load_or_build_nulls；Gate2 Control Measurement Cohort；Gate4 伪题材 GCC 用其余 199 null leave-one-out；Gate7 独立主信号日生成器，控制=已确认尚未容量启动 | §8.3、§17.1、§17.3、§18 Gate7、§25 |
| Pack4 / A-10,B-03,B-04,B-05 | CGM two-way；HC1；t 参考；seed payload；禁止 Gate2 控制跨配对复用；Gate3→1−α_v；P5/Gate4附加=guardrail | §17.2–17.3、§18.0、§20.2 |
| Pack5 / A-09,B-06 | 篮子总 TargetValue=真实信号；等权拆到成员；Gate6 域 \|K^trade\|≥4；四臂等总资本 | §0.3.2、§18 Gate6、附录B |
| Pack6 / A-11,A-12 | vol=股；分钟 bar=[t:00,t:59]；gap 日初阈值版本；09:35 先拒后释后再分配；卖单升级 | §4.1、§14、§15.2 |
| Pack7 / A-13,B-07 | Gate9=外生冻结入场 tape + 隔离充足资本；不声称完整有限资金生产政策 | §18 Gate9 |
| Pack8 / A-14,B-08 | FillRate=成交金额/目标金额；贡献仅总净收益>0 时用正贡献份额；Kill 三条件联合 Bonferroni；Track S 主法庭；删 Leg H；α-wealth≤5%；β 为版本级 | §16、§18.0、§20.4、§21.7、§3.2 |
| 附带 | Rel 不进 StockDayFeatureStore；MorningExecutableValueShare5 展开；伪代码同步 | §8.3.8、§17.2、§25 |

## 四、刻意不做的事

- 不新增交易思想、指标、Gate 或安慰剂；
- 不重写经济命题与主架构；
- 不引入双实现强制；
- 不把 Track B 并入主终局。

## 五、审计残留（已在同轮闭合）

独立复审另发现并已修：§4.1 表被单位散文打断；P5 seed 三处不一致（统一为 `Hash(spec_version,"P5",episode_id,rep)`）；§12.9 冷却同簇改引 §5.3 连通分量；工程目录 `/sfl1_v7/` → `/sfl1_v8/`。

## 六、结论

22/22 A/B 指控成立。v8 是 **Canonical Consistency Closure**：只修复交叉冲突与统计法庭缺口。Freeze/HOLDOUT/正式影子仍取决于附录 C 物化与闸门通过，不因本文件自动 GO。
