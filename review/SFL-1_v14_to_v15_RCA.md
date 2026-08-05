# SFL-1 v14 → v15 RCA（非规范）

> Canonical 以 `SFL-1_v15.md` 为准。  
> 审查建议命名 “v14.1”；按用户要求产出完整 **v15.0 Execution–Entitlement Final Closure**。  
> 范围纪律：不新增指标 / 角色 / 状态 / Gate / 退出腿。

## 核对总表

| 复审项 / 编号 | 核对 | v14 证据 | 根因 (RCA) | v15 处置 |
|---|---|---|---|---|
| P10 theme-day (A-01/B-07) | **CONFIRMED** | “题材日数”未定义 key；§27 单位 `%交易日`；缺 denom=0 | 计数对象未形式化 | `RealThemeDayKey=(theme,D)`；`% real-theme-days`；0→INSUFFICIENT |
| Stock receivable 时钟 (A-02/B-01) | **CONFIRMED** | §15.5.1 RECOGNIZED receivable；§15.5.3 “ex 开盘前到账股数” | 表文冲突两时钟 | ex=RECOGNIZED receivable；payment/listing 才增股份 |
| TaxPayable 双计 (A-03/B-01) | **CONFIRMED** | CashReceivable 税后折现 + NAV 含 TaxPayable | 净额与应付并存 | CashReceivable=gross；TaxPayable=expected_tax；ex 用当日 PIT 税制 |
| StockReceivable MTM | **CONFIRMED** | 仅 P_ref，无每日重估 | 估值路径分叉 | 每日 mark-to-market |
| TotalNet LEGAL_ONLY (A-04/B-02) | **CONFIRMED** | “尚未结算 entitlement 终值”未过滤状态 | 含权价+终值双计风险 | 仅 RECOGNIZED；LEGAL_ONLY 不计；SETTLED 只走实际处置 |
| ExecutionAllocatedBudget (A-05/B-03) | **CONFIRMED** | min 含 Reserved 与 FreeCash | FreeCash 已扣预留 → 预算归零 | 删除 CurrentFreeCash；现金约束=Reserved |
| AllIn 循环 (A-06/B-04) | **CONFIRMED** | AllIn 用 Pex 算 qty0，Pex 又依赖 TentativeValue | 未定义 provisional 初值 | Provisional(VWAP)→Impact→Final；只一轮 |
| Gate8 Intent (A-07/B-05) | **CONFIRMED** | “若保留 Intent” | Population 交给实现者 | 凡算出 TargetValue 必须物化 Intent |
| θADV × lock (A-08) | **CONFIRMED** | 忽略 rooms，未写忽略 StockPositionLock | 隔离域不唯一 | 纯 REJECT_GAP；忽略 rooms+lock |
| Gate9 unresolved (A-09/B-06) | **CONFIRMED** | 仅 8A SYNTHETIC；无 pair 规则 | 单侧清算偏倚 | `G9_NON_EVALUABLE_EXECUTION` 整对剔除 + 两臂未清算率 |
| lock owner (A-10) | **CONFIRMED** | 无 owner 字段 | owner 可能自拒 | `lock_owner_*` + owner exempt |
| odd-lot 卖出分支 | **CONFIRMED** | §15 允许整笔；§14.3 未 bypass 100 取整 | 共享 round-lot 永不卖出 | §14.3 final_liquidation 分支 |
| NON_EVALUABLE 时间退出 | **NOT AN ISSUE** | 已闭合 | — | 维持 |
| Gate9 Canonical 主冲突 | **NOT AN ISSUE** | ExitInc 已同步 | 依赖上游 outcome | 补 unresolved pair |

## 已关闭（维持）

NON_EVALUABLE 时间退出；Gate9 θExitMean/Comp MES 与 `G9_COMP_BOOT`；odd-lot 禁虚拟现金主体；P10 episode-day 方向；StockPositionLock 持仓封锁方向。
