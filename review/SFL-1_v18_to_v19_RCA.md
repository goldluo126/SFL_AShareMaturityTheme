# SFL-1 v18 → v19 RCA（非规范）

> Canonical 以 `SFL-1_v19.md` 为准。  
> 审查建议命名 “v18.1 Parent-Tax Reconciliation Closure”；按用户要求产出完整 **v19.0 Parent-Tax Reconciliation Closure**。  
> 范围纪律：不新增指标 / 角色 / 状态 / Gate / 退出腿。

## 核对总表

| 项 / 编号 | 核对 | v18 证据 | 根因 (RCA) | v19 处置 |
|---|---|---|---|---|
| partial-disposal tax slices | **PASS（维持）** | §15.5.2 disposed/remaining | — | 不改 |
| TradingDays convention | **PASS（维持）** | §6.10 / §15.5.2：`a < d ≤ b` | — | 不改 |
| LockGroup formation timestamp | **PASS（维持）** | `LockGroupFormationPreOpen` | — | 不改 |
| owner collision policy | **PASS（维持）** | `CORP_ACTION_LOCK_COLLISION`→`LEDGER_INVALID` | — | 不改 |
| A-01 / B-01 parent tax alloc | **CONFIRMED** | §15.5.3：「按 `source_trade_lot_ids` 股数比例分摊」；§15.5.2 税额已按 slice 算 | tax slice 有 `parent_order_id`，但 parent 净权益仍允许 trade-lot 均摊总净额 | 强制实现 B：`CashReceivable_(e,p)` / `TaxPayable_(e,p,t)` / `SettledDividendCash_(e,p)`；§0.3 Unsettled 用 parent 下标；禁 trade-lot 均摊税额 |
| A-02 / B-02 planned exit PIT | **CONFIRMED** | 仅出现 `planned_exit_signal_date_of_arm`，无 dynamic 未触发定义 | 未触发时可 fallback / LegC / 未来实际退出日分叉；后者 look-ahead | `PlannedExitDate_arm(t)`：fixed20 恒定；dynamic 未触发→fixed20 fallback；触发后→已归档日；禁 t 后回填 |
| A-03 / B-03 bilateral true-up | **CONFIRMED** | payment 后仅 `DIV_TAX_CLAWBACK`（额外追缴） | 实际税 < 预计税时无 reversal；与“实际卖出优先”冲突 | `SettledEstimatedTax`→`FinalActualTax`→`TaxTrueUp`；正 clawback / 负 `DIV_TAX_REVERSAL`；`TotalNetProceeds` 用 `- TaxTrueUp_p` |

## 设计取舍

1. **Parent 税额路径：** 只强制现金股息税额走 tax-slice→`parent_order_id`；非税股份/权利金按冻结 `source_tax_lot_alloc` 股数归集，禁止用 trade-lot 覆盖税额。
2. **Dynamic planned date：** 未触发唯一 fallback = 预注册 fixed20 退出信号日（与入口验证一致），不引入新退出腿或最大持有期状态机。
3. **True-up 符号：** `TaxTrueUp = FinalActual − SettledEstimated`；`TotalNetProceeds` 统一减 `TaxTrueUp_p`（负值即加回过度预计税）。

## 不在本轮范围

P10、ExecutionAllocatedBudget、Gate 8 Population、Gate 9 cutoff / unresolved guard、公司行动 pre-open cutoff、Settled stock proceeds 科目边界、题材发现/角色/状态机及其他 Gate。
