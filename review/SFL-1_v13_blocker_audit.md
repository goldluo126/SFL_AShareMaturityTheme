# SFL-1 v13 阻断闭合审计（非规范）

对照 v12 Review A-01…A-22 / B-01…B-10。

| 编号 | v12 核对 | v13 落点 | 审计 |
|---|---|---|---|
| A-01 确认后前三行 | CONFIRMED | §12.3.1 ConfirmedEventFlag；§12.4 | **CLOSED** |
| A-02 对照资本 | CONFIRMED | §0.3.2 实际成交净成本同构 | **CLOSED** |
| A-03 entitlement | CONFIRMED | §15.5.1 | **CLOSED** |
| A-04 分红成本 | CONFIRMED | AvgCost 不因现金分红变 | **CLOSED** |
| A-05 旧市值 AvgCost | CONFIRMED | OldShares×OldAvgCost | **CLOSED** |
| A-06 P10 日随机 | CONFIRMED | spawn 后 membership 冻结运行 | **CLOSED** |
| A-07 P10 fold | CONFIRMED | spawn 时冻结 fold | **CLOSED** |
| A-08 P10 ID | CONFIRMED | ID 含 control_index | **CLOSED** |
| A-09 年度覆盖 | CONFIRMED | N_epi ≥ N_real×40 公式 | **CLOSED** |
| A-10 odd-lot | CONFIRMED | FINAL_ODD_LOT_LIQUIDATION | **CLOSED** |
| A-11 StockOrderLock | CONFIRMED | §15.0 | **CLOSED** |
| A-12 FillRate | CONFIRMED | ParentFillRate | **CLOSED** |
| A-13 CommissionReserve | CONFIRMED | Lifecycle vs Intraday | **CLOSED** |
| A-14 TailAdjust | CONFIRMED | 仅最后一分钟重算 | **CLOSED** |
| A-15 TentativeValue | CONFIRMED | §14.1 正式公式 | **CLOSED** |
| A-16 伪代码时钟 | CONFIRMED | §24/§25 真实顺序 | **CLOSED** |
| A-17 卖出执行 | CONFIRMED | execute_streaming_sell_1430 | **CLOSED** |
| A-18 确认日 null | CONFIRMED | just_frozen 重载 EPISODE_NULL | **CLOSED** |
| A-19 重复退出 | CONFIRMED | desired state + 幂等物化 | **CLOSED** |
| A-20 Gate9 现金 | CONFIRMED | parent-order ExitInc | **CLOSED** |
| A-21 目录 | CONFIRMED | /sfl1_v13/ | **CLOSED** |
| A-22 α k | CONFIRMED | v13 k=1；HOLDOUT 未消耗 | **CLOSED** |
| B-01…B-10 | 随 A | 8B ParentTarget；donor 敏感性；FSR evaluable | **CLOSED** |

Impact 单次、bootstrap 整块、index_date、ThemeRoom、Gate1 cohort、Gate8A/8B 框架：维持。

**规范文本层：** Review 所列 A/B 已闭合。  
**Freeze：** 仍取决于附录 C 工程闸门与扩展 Run-in 黄金路径物化。
