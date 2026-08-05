# SFL-1 v19 → v20 RCA（非规范）

> Canonical 以 `SFL-1_v20.md` 为准。  
> 审查建议命名 “v19.1 Tax-Posting Final Closure”；按用户要求产出完整 **v20.0 Tax-Posting Final Closure**。  
> 范围纪律：不新增指标 / 角色 / 状态 / Gate / 退出腿 / 税务模型。

## 核对总表

| 项 / 编号 | 核对 | v19 证据 | 根因 (RCA) | v20 处置 |
|---|---|---|---|---|
| parent-specific gross/tax allocation | **PASS（维持）** | §15.5.3 实现 B；§0.3 parent 下标 | — | 不改 |
| PIT PlannedExitDate_dynamic | **PASS（维持）** | §15.5.2 fixed20 fallback；禁 look-ahead | — | 不改 |
| A-01 / B-01 disposal&lt;payment 税制版本 | **CONFIRMED** | payment 记 `SettledEstimatedTax=Tax_(e,s,payment)`（payment 税制）；`FinalActualTax` 用 disposal 税制；又强制 `TaxTrueUp=0` | 只冻结 holding days，未冻结 tax_rule_version → SettledDividendCash 可 80 或 90 | payment 分支：disposal&lt;payment → `SettledTaxAtPayment=FinalActualTax`（disposal 税制）；`TaxTrueUp=0`；禁 payment 税制覆盖 |
| A-02 / B-02 TaxTrueUp 现金过账/幂等 | **CONFIRMED** | TaxTrueUp 入 TotalNetProceeds；伪代码仅 `update_cash_positions_after_sells` | 可只改研究 outcome 不改 Cash；重放可双过账 | 卖出后强制顺序 + `Cash-=TaxTrueUp` + `TaxTrueUpEventKey` 幂等；禁入 SettledCorporateActionCash |

## 设计取舍

1. **disposal-before-payment：** 最终税负在 disposal 时点按当时可知税制锁定；payment 不再重开税制版本窗口。
2. **现金过账：** TaxTrueUp 是真实组合现金事件，不是纯研究调整；符号统一 `Cash -= TaxTrueUp`。
3. **幂等：** 最小 EventKey = `(entitlement_id, disposed_slice_id, disposal_fill_id)`；`trueup_status∈{NOT_REQUIRED,PENDING,POSTED}`。

## 不在本轮范围

source_tax_lot_alloc、partial-disposal slices、TradingDays、PREOPEN_ACTION_CUTOFF、LockGroup、owner collision、Gate 9 cutoff、其他信号/状态机/执行/统计模块。
