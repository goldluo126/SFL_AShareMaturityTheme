# SFL-1 v17 Blocker Audit（非规范）

审计对象：`SFL-1_v17.md`（Tax-Lot–Selection Final Closure）  
对照：v16.0 八项定向深度 Review（A-01…A-06 / B-01…B-04）。

## 结论文本

| 项 | 结果 |
|---|---|
| 审查所列 A 类阻断 | **0（文本闭合）** |
| 审查所列 B 类阻断 | **0（文本闭合）** |
| 本轮已 PASS 四项 | 维持 |
| Freeze | 仍须附录 C / Run-in / NG / 推断验收全部通过 |

## 逐项闭合证据

| ID | 状态 | v17 锚点 |
|---|---|---|
| A-01 per-lot tax | CLOSED | §15.5.2 `TaxPayable_(e,t)=Σ_l Gross×rate` |
| A-02 disposal freeze | CLOSED | `TaxHoldingEndDate_l` |
| A-03/B-01 PREOPEN | CLOSED | §4.3 `PREOPEN_ACTION_CUTOFF`；税制只用该 cutoff |
| A-04/B-04 proceeds | CLOSED | §0.3 `SellNetProceeds` / `SettledCorporateActionCash` |
| A-05/B-02/B-03 ΔU | CLOSED | §18 `Z_p`；CGM `CI⊂[±tol]`；绝对率≤0.20 |
| A-06 successor lock | CLOSED | §15.0 LockGroup；§15.5.3 CODE_CHANGE 原子迁移 |
| P10 / budget / zero Intent / G9 cutoff | PASS 维持 | 未回归 |

## 静态扫描摘要

- `加权最早取得日`：0  
- `ΔU = UnresolvedRate_fixed20 - UnresolvedRate_dynamic`：0  
- `PREOPEN_ACTION_CUTOFF` / `GrossDividend_(e,l)` / `SellNetProceeds_p` / `SecurityIdentityLockGroup` / `Z_p`：均存在  
- `/sfl1_v17/`；α-wealth v17 k=1（v1–v16 未触碰 HOLDOUT）

## 剩余工程门槛

附录 C、Run-in 黄金路径、NG01–NG07、推断验收、分区清单物化。文本闭合 ≠ 已 Freeze。
