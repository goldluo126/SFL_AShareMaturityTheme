# SFL-1 v13 → v14 RCA（非规范）

> Canonical 以 `SFL-1_v14.md` 为准。  
> 版本命名：用户要求产出完整 v14；审查建议的 “v13.1” 内容并入 **v14.0 Court–Ledger Consistency Closure**。  
> 范围纪律：不新增指标 / 角色 / 状态 / Gate / 退出腿 / 对照类型。

## 核对总表

| 编号 | 核对 | v13 证据 | 根因 (RCA) | v14 处置 |
|---|---|---|---|---|
| A-01/B-01 NetAlpha20 总回报 | **CONFIRMED** | §0.3 写卖出/买入；Gate 8B 含公司行动，8A 未强制同一函数 | 总回报分子未单一化 | §0.3 `TotalNetProceeds`；8A/8B/9 共用 |
| A-02/B-02 entitlement NAV | **CONFIRMED** | NAV=`Cash+ΣPosition`；又写 receivable 计入 NAV | 法律 entitlement 与会计确认时点混用 | record / ex / payment 三时点；NAV 含 receivable − TaxPayable；FreeCash 禁 receivable |
| B-03 P10 coverage | **CONFIRMED** | `N_epi ≥ N_real×40` + `CoverDays≥0.9×TradingDays` | 用 spawn 数惩罚长寿 episode；分母用交易日而非可评估题材日 | `PseudoEpisodeDays/RealEvaluableThemeDays≥40`；覆盖真实可评估题材日 ≥90% |
| A-03 control 恰好一次 | **CONFIRMED** | “每个 control 恰好一次” vs respawn | 对象从 control_index 误绑到 episode | 每 `P10_NULL_EPISODE_ID` 审计一次 |
| A-04 NON_EVALUABLE 退出 | **CONFIRMED** | 冻结正常转移；时间退出未列 | 伪代码只消费 LAC 的歧义 | 明确固定20日 / Leg C / 已触发腿继续 |
| A-05 StockPositionLock | **CONFIRMED** | 仅封锁开放买单 | lock 与“单一持仓归属”不同构 | 持仓 / 开买单 / 开卖单期间禁第二 parent |
| A-06/B-04 Gate9 旧参数 | **CONFIRMED** | 正文 ExitInc；§27 ΔSharpe+21日；seed `G9_BOOT` | 参数表/seed 未随正文迁移 | MES=0.01/0.01；`G9_COMP_BOOT`；删旧法庭 |
| A-07/B-05 θExitComp | **CONFIRMED** | “聚合后平均”无公式 | component 等权 vs parent 等权歧义 | `(1/G)Σ_g (1/n_g)Σ_p ExitInc`；`w_p=1/(G n_g)` |
| A-08 获配预算 | **CONFIRMED** | `Intraday=Lifecycle−Reserve` | 意图预算误当执行上限 | `ExecutionAllocatedBudget_t` |
| A-09/B-06 all-in qty | **CONFIRMED** | `max_qty` 用 `Pex` only | 变动费用后置导致超支 | `AllInUnitCost`；日终 `CommissionTopUp` 只补差额 |
| A-10 Gate8 Population | **CONFIRMED** | “全部理论信号” / 无 ParentTarget | 对象层未分层 | SignalCandidate / TheoreticalOrderIntent / ParentOrder |
| B-07 odd-lot 虚拟现金 | **CONFIRMED** | 5 日 `ODD_LOT_CASH_EXIT` | 虚拟成交冒充可实现 | 删除虚拟变现；仅真实结算；`SYNTHETIC_TERMINAL_MARK`→NON_EVALUABLE_EXECUTION |
| 伪代码 buy_orders | **CONFIRMED** | `execute(...buy_orders)` | 过滤后变量未传入执行器 | 只执行 `eligible_buy_orders` + asserts |

## 已确认 CLOSED（v13 保留，不回归）

ConfirmedEventFlag；fill-cost 同构 NetAlpha20 分母；P10 membership/fold 冻结；ID 含 control_index；FSR evaluable-only；Lifecycle vs CommissionReserve 主体拆分；AvgCost=OldShares×OldAvgCost；ParentFillRate；Gate8B PnL/ParentTarget 方向；伪代码真实日内时钟；确认日 EPISODE_NULL 重载；`/sfl1_v14/`；α-wealth（v14 声明：v1–v13 未触碰 HOLDOUT，本版 k=1）。

## 非阻断（维持设计取舍）

冻结 K 不纳入后期新容量股；Track B 不进主终局；分钟参与≠真实订单簿；Gate 5 毛收益分轨；Rights NEVER_EXERCISE；题材簇超限只禁新买；单一生产实现+独立重发。
