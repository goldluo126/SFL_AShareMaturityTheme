# SFL-1 v9 → v10 核对、RCA 与修复裁决

> 性质：非规范性 Review/RCA。正式规则以 `SFL-1_v10.md` 为唯一 Canonical 来源。
> 输入：外部对 v9.0 的 14 层深度 Review（A-01~A-15、B-01~B-08）。

## 一、逐条核对结论

| 编号 | 指控 | v9 证据 | 结论 |
|---|---|---|---|
| A-01 | 确认日冻结后历史斜率无唯一口径 | 有冻结 B/L/K/null；无 FrozenMetricBackcast；斜率用 D−3:D−1 | **属实** |
| A-02 | Gate7 处理用 S、控制用 S−1 信号 | §18 Gate7 “S−1 及以前可算值”用于 EntryScore/CONF/RMI/GCC | **属实** |
| A-03 | P10 无稳定伪 episode | P10 对象为 (c,D)；recipient 含 D；无跨日 episode ID | **属实** |
| A-04 | 合并后无 union null | §5.4 合并后 u1 接管状态；无 union null 规则 | **属实** |
| A-05 | 动态并簇超 20% 无处置 | §15 硬上限 20%；簇每日重算；无超限算法 | **属实** |
| A-06 | Gate6/7 毛净/卖出不唯一 | 写“§14.1 假想成交”；无 R_arm_net 公式 | **属实** |
| A-07 | block bootstrap 抽样单元歧义 | “以 episode 为块、按 ID1”可读为抽 episode | **属实** |
| A-08 | TargetValue→股数未定义 | TargetValue 人民币；执行用股数；无转换 | **属实** |
| A-09 | 跨日补单金额 vs 股数 | 剩余目标金额与数量混用 | **属实** |
| A-10 | T+1 可卖库存缺失 | 理论承认 T+1；账本无 SellableShares | **属实** |
| A-11 | 公司行动账本缺失 | 有复权因子；无分红/送转账本路径 | **属实** |
| A-12 | 对照篮子补单生命周期不清 | §0.3.2 只写同一 τ_entry + §14.1 | **属实** |
| A-13 | REJECT_GAP 与 later-filled 重叠 | θ_ADV 按 rejected vs filled 两组 | **属实** |
| A-14 | Gate9 tape 行粒度/母订单 | 字段无 signal_id/parent；Σ target_value 可重复 | **属实** |
| A-15 | 经济 guardrail 公式与终局冲突 | §20.5 无公式；§22 缺 Gate9 PASS+动态 guardrail FAIL 分支 | **属实** |
| B-01~B-08 | 对应上表统计后果 | 同上 | **属实** |

v8/v9 已 CLOSED 项（连通分量排除、open episode 日更、DISC_NULL、MES5 流式、Gate5 predictor、Eligible≥2、独立 registry、Kill 分离等）**未复发**。

## 二、根因

1. **量尺切换未定义回算**：确认冻结改变测量制度，但差分历史未强制同尺回算。
2. **账务合并 ≠ 测量合并**：合并规则把状态机交给 u1，却无对应 null。
3. **研究对照时点与匹配时点混淆**：把匹配截止误写成控制信号截止。
4. **执行层仍是“金额意图、股数实现”而未闭合**：缺母订单人民币余额、T+1、公司行动。
5. **政策对照对象粒度未到母信号/母订单**：P10 (c,D)、Gate8 组、Gate9 tape 行均可多重计数。

## 三、v10 裁决（Execution–Measurement Final Closure）

| Pack | 裁决 | 位置 |
|---|---|---|
| 1 | FrozenMetricBackcast：τ_confirm 日用冻结 cohort/null 回算所需历史；禁拼接临时量尺；不改写确认前状态史 | §3.5、§11、§12 |
| 2 | 账务可合并；measurement 不合并；各 origin 保留 null/状态；退出按 origin 持仓 | §3.3、§5.4 |
| 3 | P10_NULL_EPISODE 稳定生命周期；年化按唯一 episode 计数 | §19 P10 |
| 4 | Gate7：匹配≤S−1；四臂信号=S；执行=S+1；从未容量启动至 S | §18 Gate7 |
| 5 | 人民币母订单 + RemainingTargetValue；T+1 SellableShares | §14、§15 |
| 6 | PIT corporate-action ledger + DG | §4、§15.5 |
| 7 | Gate6/7 正式 R_arm_net 完整净执行；毛口径诊断 | §18 |
| 8 | Gate8 信号级配对 θ_ADV | §18 Gate8 |
| 9 | Gate9 tape 母订单字段；InitialCapital 每 parent 一次 | §18 Gate9 |
| 10 | §20.5 公式 + 终局：Gate9 PASS 且动态 guardrail FAIL → PASS_ENTRY_ONLY | §20.5、§22 |
| 附 | 并簇超限=禁止新买不强制减仓；H-CLOCK 主估 level 交互 + slope guardrail；bootstrap 按 ID1 整块抽 | §15、§2、§17.4、§20.2 |

## 四、审计残留

独立复审确认主清单 CLOSED；已修 P10 年度计数残留“按日求和”措辞、`UnsettledShares`→`UnsettledBuyShares`、Gate6 账本标题。

## 五、结论

全部所列 A/B 指控成立。v10 只做执行–测量闭包，不新增题材指标/Gate。Freeze 仍取决于附录 C 物化。
