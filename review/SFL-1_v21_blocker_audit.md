# SFL-1 v21 Blocker Audit（非规范）

审计对象：`SFL-1_v21.md`（Tax-Basis Split Closure）  
对照：v20.0 三项定向深度 Review（A-01 / B-01；两项 PASS）。

## 结论文本

| 项 | 结果 |
|---|---|
| 审查所列 A 类阻断（A-01） | **0（文本闭合）** |
| 审查所列 B 类阻断（B-01） | **0（文本闭合）** |
| 本轮两项 PASS | 维持（tax-version precedence / idempotency） |
| Freeze | 仍须附录 C / Run-in / NG / 推断验收全部通过 |

## 逐项闭合证据

| ID | 状态 | v21 锚点 |
|---|---|---|
| A-01/B-01 basis split | CLOSED | §15.5.2 `SettledEstimatedTax basis split`：`T×q/Q` + `T−disposed` + 守恒 + 尾差 remaining |
| tax-version / idempotency / cash posting | PASS 维持 | 未回归；伪代码改为 `split_tax_slices_and_settled_tax_basis_by_sell_fills` |

## 静态扫描摘要

- `SettledEstimatedTax_disposed = T × q / Q`：存在  
- 守恒等式与“禁全部 T 给 disposed/remaining/0”：存在  
- 旧 helper `split_tax_slices_by_sell_fills`：已替换  
- `/sfl1_v21/`；α-wealth v21 k=1（v1–v20 未触碰 HOLDOUT）

## 剩余工程门槛

附录 C、Run-in（1000股/T=100 → 卖300 基线30/70；Cash-=30）、NG01–NG07、推断验收、分区清单物化。文本闭合 ≠ 已 Freeze。
