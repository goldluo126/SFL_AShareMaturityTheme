# SFL-1 v21 → v22 RCA（非规范）

> Canonical 以 `SFL-1_v22.md` 为准。  
> 审查建议命名 “v21.1 Monetary Quantization Final Closure”；按用户要求产出完整 **v22.0 Monetary Quantization Final Closure**。  
> 范围纪律：不新增指标 / Gate / 退出腿 / 税务模型；不改 proportional 经济公式结构。

## 核对总表

| 项 / 编号 | 核对 | v21 证据 | 根因 (RCA) | v22 处置 |
|---|---|---|---|---|
| proportional split / 守恒 / 伪代码顺序 | **PASS（结构维持）** | `T×q/Q`、`T−disposed`、先拆后 true-up | — | 保留；改为分整数算子 |
| A-01 / B-01 货币量化 | **CONFIRMED** | 「冻结定点或精确有理数」「按冻结舍入规则」；未定义单位/模式/tie | Canonical 禁止 params 补规范 → half-up / half-even / exact-rational 均可守恒却产生不同 disposed baseline 与 Cash 路径 | 单位=人民币分；`QuantizeTaxToFen=floor(x×100)`；`disposed_fen=floor(T_fen×q/Q)`；禁浮点与未定义舍入 |

## 设计取舍

1. **单位：** 非负整数分，与 `Cash_fen -= TaxTrueUp_fen` 同构。  
2. **disposed：** `floor` 与“尾差留 remaining”一致；`q=Q` 时 `disposed=T_fen`。  
3. **中间量：** 任意精度整数/有理数；禁止 IEEE754 参与量化或拆分。

## 不在本轮范围

disposal-before-payment 税制、EventKey 幂等、parent tax allocation、tax-lot FIFO、partial-disposal 架构本身、PlannedExitDate、Gate 8/9 其他内容。
