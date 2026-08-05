# SFL-1 v12 → v13 RCA（非规范）

> Canonical 以 `SFL-1_v13.md` 为准。  
> 依据用户 **SFL-1 v12.0 SPEC 深度 Review**。范围纪律：**Identity–Ledger Final Closure**；不新增指标/角色/状态/Gate/退出腿。

---

## 0. 核对总表

| 编号 | 核对 | v12 证据 | v13 处置 |
|---|---|---|---|
| A-01 确认后前三行 | **CONFIRMED** | §12.4 仍写“CONFIRMED_EXPANSION 的前三行成立” | ConfirmedEventFlag 吸收事实 |
| A-02 对照资本口径 | **CONFIRMED** | “未成交现金0”+“成交金额加权”可双解 | 条件成交：两侧按实际成交净成本归一 |
| A-03 record-date 权益 | **CONFIRMED** | 有 record_date 字段但无 entitlement 账 | CorporateActionEntitlementLot |
| A-04 分红成本基础 | **CONFIRMED** | “除息日调整成本”无公式 | AvgCost 不因现金分红改变 |
| A-05 旧市值 AvgCost | **CONFIRMED** | 明文“旧市值+新净成本” | OldShares×OldAvgCost |
| A-06 P10 日随机 control | **CONFIRMED** | seed 含 D；continuation 比 Jaccard | spawn 后 membership 冻结运行 |
| A-07 P10 fold 随 D | **CONFIRMED** | Hash(theme,D,c) mod 5 | fold 于 spawn 冻结 |
| A-08 P10 ID 缺 control | **CONFIRMED** | ID 无 control_index；spawn 在 (theme,fold) | ID 含 control；per-control spawn |
| A-09 年度覆盖下限 | **CONFIRMED** | “对应下限”无公式 | 公式化 |
| A-10 零碎股永不关闭 | **CONFIRMED** | 仅公司行动清除 | FINAL_ODD_LOT_LIQUIDATION |
| A-11 同股多订单 | **CONFIRMED** | 只有持仓去重 | StockOrderLock |
| A-12 FillRate 分母 | **CONFIRMED** | “全部理论买单”可含重试日 | ParentFillRate 唯一主口径 |
| A-13 CommissionReserve 侵蚀 | **CONFIRMED** | Remaining 初值减 reserve 且跨日继承 | Lifecycle vs Intraday 拆分 |
| A-14 TailAdjust Impact | **CONFIRMED** | 未规定是否重算 | 仅最后一分钟重算一次 |
| A-15 TentativeValue | **CONFIRMED** | “无冲击价估计”无公式 | 正式公式 |
| A-16 伪代码时钟 | **CONFIRMED** | 同循环内生成并执行 | 真实日内顺序 |
| A-17 缺卖出执行 | **CONFIRMED** | 无 execute_streaming_sell | 补齐 |
| A-18 确认日 DAILY stats | **CONFIRMED** | just_frozen 后仍用 m[pseudo_stats] | 重载 EPISODE_NULL stats |
| A-19 重复退出订单 | **CONFIRMED** | emit_pending + emit_exit | 风险分支只改 desired；统一物化 |
| A-20 Gate9 现金 | **CONFIRMED** | max concurrent 不足覆盖亏损后 tape | parent-order 配对退出增量 |
| A-21 目录 v10 | **CONFIRMED** | `/sfl1_v10/` | `/sfl1_v13/` |
| A-22 α k 冲突 | **CONFIRMED** | 正文 v10 k=1；附录 v12 | v13 k=1（HOLDOUT 从未消耗） |
| B-01…B-10 | 随对应 A | — | 同上 + 8B ParentTarget 收益 + donor 敏感性 |

Impact 单次计入、bootstrap 整块、index_date、ThemeRoom、Gate1 cohort、Gate8A/8B 分离方向：维持并加固。
