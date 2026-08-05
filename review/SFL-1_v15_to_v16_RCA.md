# SFL-1 v15 → v16 RCA（非规范）

> Canonical 以 `SFL-1_v16.md` 为准。  
> 审查建议命名 “v15.1”；按用户要求产出完整 **v16.0 Final Support–Censoring Closure**。  
> 范围纪律：不新增指标 / 角色 / 状态 / Gate / 退出腿。

## 核对总表

| 编号 / 项 | 核对 | v15 证据 | 根因 (RCA) | v16 处置 |
|---|---|---|---|---|
| A-01/B-01 P10 episode-day 域 | **CONFIRMED** | PseudoEpisodeDays=Σ_e 生命周期可评估日，未 ∩ R_y | 分子未绑定真实可评估 theme-day | `P10_EVALUABLE_EPI_DAY`；仅对 `(u,D)∈R_y` 求和 |
| A-02/B-02 expected_tax | **CONFIRMED** | TaxPayable=expected_tax；holding_days 未冻结；§15.3.1 写 payment PIT | 输入与重估时点分叉 | `ExpectedHoldingDays_ex=max(当前,固定20)`；每日/税制重估；payment 结算公式 |
| A-03/A-08 share entitlement lock | **CONFIRMED** | lock 仅 shares/open buy/sell | 未来股份权益释放锁 | `OutstandingShareGeneratingEntitlement`；release 枚举 |
| A-04 terminal net | **CONFIRMED** | RECOGNIZED 按 §15.5.2，未写 Cash−Tax | gross vs net 歧义 | `UnsettledEntitlementTerminalValue=(CashRecv−TaxPay)+Stock+Rights` |
| A-05 伪代码顺序 | **CONFIRMED** | commission 在 reallocate 前 | Canonical 与正文冲突 | reject→reallocate→ExecutionAllocated→佣金→执行 |
| A-06/B-03 零金额 Intent | **CONFIRMED** | 强制物化；PolicyReturn=PnL/Target 可 0/0 | 零分母未定义 | 方案 A：Target=0⇒PolicyReturn=0、θADV=0；`RealizedNetPnL=TNP−BuyCost` |
| A-07/B-04/B-05 Gate9 cutoff/selection | **CONFIRMED** | “正式 outcome 截止时”无定义；ΔU 仅诊断 | censoring 与 complete-case 外推 | `G9_OUTCOME_CUTOFF=min(fill+90,partition_end)`；Population=`PairResolvableByCutoff`；ΔU/绝对率超限→INCONCLUSIVE |
| provisional/final AllIn | **NOT AN ISSUE** | 已唯一 | — | 维持 PASS |

## 设计取舍（已冻结）

- 零金额 Intent：**方案 A**（保留并记 0），完整反映无空间放弃交易。  
- Gate 9：**收窄 estimand** 至可清算配对 + unresolved 不平衡强制 INCONCLUSIVE，不新增 Gate。  
- expected_tax holding_days：`max(当前持有, 计划固定20退出持有)`，偏保守且可复现。
