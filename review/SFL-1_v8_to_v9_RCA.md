# SFL-1 v8 → v9 核对、RCA 与修复裁决

> 性质：非规范性 Review/RCA。正式规则以 `SFL-1_v9.md` 为唯一 Canonical 来源。
> 输入：外部对 v8.0 的 14 层深度 Review（A-01~A-10、B-01~B-08）。

## 一、逐条核对结论

| 编号 | 指控 | v8 证据 | 结论 |
|---|---|---|---|
| A-01 | null 排除用直接 Jaccard，簇用连通分量 | §8.3.1 `⋃_{v:J≥0.60}` vs §5.3 连通分量 | **属实** |
| A-02 | 伪代码只更新代表题材状态 | §25 `for theme in representatives` 含状态更新 | **属实** |
| A-03 | themes 未并入全部 open episodes | §25 `themes = load_base_universe_asof(D-1)` | **属实** |
| A-04 | Pctl_cross 含无合法 RMI/GCC 的 DISCOVERED | §13.2 “DISCOVERED 及以后” | **属实** |
| A-05 | Gate 1 未来伪题材未冻结 | DAILY_NULL 当日不冻结；Gate1 无 DISCOVERY_RESEARCH_NULL | **属实** |
| A-06 | H-ROLE“同资格”未定义 | §17.2 只写“同资格股票” | **属实** |
| A-07 | Gate5 predictor 日与附加通过条件 | §17.4 无 predictor_date；通过条件 2–4 未入 court | **属实** |
| A-08 | Gate6 只要求 \|K^trade\|≥4 | §18 Gate6；未要求 Eligible≥2 | **属实** |
| A-09 | Gate2/7 control registry/自排除/Temporary | 共用登记；Jaccard 含自身歧义；Temporary Cohort | **属实** |
| A-10 | Gate9 tape/隔离资本不唯一 | “生产信号规则”+“初始现金≥” | **属实** |
| B-01 | MES5 每分钟重复 min(.,Q) | §17.2 公式无 remaining_Q | **属实** |
| B-02 | Gate7 处理冻结 vs 控制重冻结 | Temporary Cohort at S | **属实** |
| B-03 | Gate5 look-ahead 风险 | 同 A-07 | **属实** |
| B-04 | Gate5 附加条件不在 court | 同 A-07 | **属实** |
| B-05 | Gate9 tape 依赖退出政策 | 同 A-10 | **属实** |
| B-06 | Kill 后两条无水平校准 | §21.7 声称联合≤5% | **属实** |
| B-07 | MatchedRandom 可含不合格股 | 从 K^trade\前2 抽，未要求入场资格 | **属实** |
| B-08 | 稳健性算法未完全定义 | §20.2.4 只有名称 | **属实** |

附带属实：Gate S 附录 B“同一入口/退出序列”与 overlay 改变入口矛盾；动态簇 ID 可能低估依赖；`pass_theme_gate` 失败跳过风险分支。

v8 已 CLOSED 项（簇算法、ConfirmQuality、§3.5、load-or-build、NON_EVAL/LAC、vol/bar、gap、09:35 资金、FillRate、Track S 等）**未复发**。

## 二、根因（4 个最终接口）

1. **同一题材簇未在排除/去重/统计中强制同对象**（直接邻居 vs 连通分量）。
2. **状态机消费面窄于持仓面**（代表题材更新 vs 全部 open episode）。
3. **研究对照未与生产冻结制度同构**（Temporary Cohort、未冻结 discovery null、同资格池）。
4. **Gate estimand 输入未完全外生**（tape 依赖退出、资本“≥”、Kill 原始阈值冒充显著性）。

## 三、v9 裁决摘要

| Pack | 裁决 | 位置 |
|---|---|---|
| 1 | 排除集 = ConnectedComponent(u,D) 全题材成员并集；黄金样例 | §8.3.1 |
| 2 | themes∪open；全部 open 更新状态/退出；仅代表可确认/入场；gate 失败仍风险分支 | §5.3、§12、§25 |
| 3 | Pctl_cross = 同日具合法冻结 RMI^E∧GCCSlope^θ 的开放 episode | §13.2 |
| 4 | DISCOVERY_RESEARCH_NULL_D 冻结成员+L，专供 Gate1 D+1:D+5 | §8.3.3、§17.1 |
| 5 | H-ROLE null pool = 伪题材 L^E∪K^E；MES5 流式 remaining_Q | §17.2 |
| 6 | predictor_date=τ_confirm；顶组/单调=guardrail；GCC^θ=诊断 | §17.4、§18 |
| 7 | Gate6：Eligible≥2∧NonSelectedEligible≥2；MatchedRandom 仅合格未入选；Gate7 独立 registry、排除 e≠self、用原 B^E/L^E/K^E/EPISODE_NULL、双方共同支持 | §18 |
| 8 | Gate9 tape=固定20日入口政策生产 tape；InitialCapital=ΣTargetValue | §18 |
| 9 | Kill 后两条=deterministic risk stop，不宣称 false-death；稳健性算法闭合；Gate S=端到端 overlay；partition 内 persistent cluster | §20.2、§21.7、附录B |

## 四、审计残留

独立复审确认 A/B 项均 CLOSED；附录 B Gate3/4/8 Dependence 列已与 §20.2.1 persistent cluster 对齐。

## 五、结论

18/18 指控成立。v9 为 **Final Interface Closure**，不新增交易指标/Gate。Freeze 仍取决于附录 C 物化证据。
