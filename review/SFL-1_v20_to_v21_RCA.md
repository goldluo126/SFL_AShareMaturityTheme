# SFL-1 v20 → v21 RCA（非规范）

> Canonical 以 `SFL-1_v21.md` 为准。  
> 审查建议命名 “v20.1 Tax-Basis Split Closure”；按用户要求产出完整 **v21.0 Tax-Basis Split Closure**。  
> 范围纪律：不新增指标 / Gate / 退出腿 / 税务模型。

## 核对总表

| 项 / 编号 | 核对 | v20 证据 | 根因 (RCA) | v21 处置 |
|---|---|---|---|---|
| disposal-before-payment tax-version | **PASS（维持）** | SettledTaxAtPayment=FinalActualTax；禁 payment 税制覆盖 | — | 不改 |
| TaxTrueUp idempotency | **PASS（维持）** | EventKey + POSTED + post_tax_trueup_cash_once | — | 不改 |
| A-01 / B-01 SettledEstimatedTax child baseline | **CONFIRMED** | payment 写 SettledEstimatedTax 于父 slice；部分卖出只拆 shares/disposal_date；TaxTrueUp=Final−SettledEstimated 未定义 child 的 SettledEstimatedTax | shares 拆分已定义，payment 估计税负债未随 split 传递 → TaxTrueUp 金额可取 30/−40/60 等 | `SettledEstimatedTax_disposed=T×q/Q`；remaining=`T−disposed`；守恒；尾差留 remaining；禁全给 disposed/remaining/0 |

## 设计取舍

1. **比例拆分（实现 A）：** 与 GrossDividend 按股归集同构；保持 payment 总估计税在全生命周期守恒。
2. **尾差策略：** disposed 先按冻结舍入；剩余全部进 remaining；末次清算吃掉剩余 baseline。
3. **伪代码：** `split_tax_slices_by_sell_fills` → `split_tax_slices_and_settled_tax_basis_by_sell_fills`。

## 不在本轮范围

parent-specific tax allocation、PlannedExitDate、tax-lot allocation、partial-disposal 架构本身、TradingDays、Gate 8/9 其他规则、其他执行/信号/统计模块。
