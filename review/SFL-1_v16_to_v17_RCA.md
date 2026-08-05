# SFL-1 v16 → v17 RCA（非规范）

> Canonical 以 `SFL-1_v17.md` 为准。  
> 审查建议命名 “v16.1”；按用户要求产出完整 **v17.0 Tax-Lot–Selection Final Closure**。  
> 范围纪律：不新增指标 / 角色 / 状态 / Gate / 退出腿。

## 核对总表

| 项 / 编号 | 核对 | v16 证据 | 根因 (RCA) | v17 处置 |
|---|---|---|---|---|
| P10 episode-day | **PASS（维持）** | 已绑定 R_y | — | 不改 |
| A-01 多 tax-lot | **CONFIRMED** | “FIFO/加权最早取得日”单 holding | 聚合先于税率 | 逐 lot `GrossDividend×tax_rate` |
| A-02 卖出后 clock | **CONFIRMED** | `holding_days_as_of_today` 无冻结 | disposal 后仍增长 | `TaxHoldingEndDate=min(t,disposal)` |
| A-03/B-01 pre-open PIT | **CONFIRMED** | ex/payment 用当日 INPUT 20:00 | 开盘前读收盘后信息 | `PREOPEN_ACTION_CUTOFF=PrevTD 20:00` |
| A-04/B-04 SETTLED stock 双计 | **CONFIRMED** | “卖出净所得+SETTLED 处置所得”边界模糊 | 科目未分离 | `SellNetProceeds` vs `SettledCorporateActionCash` |
| ExecutionAllocatedBudget 顺序 | **PASS（维持）** | 已同构 | — | 不改 |
| zero-target Intent | **PASS（维持）** | PolicyReturn=0 | — | 不改 |
| Gate 9 cutoff | **PASS（维持）** | fill+90 / partition_end | — | 不改 |
| A-05/B-02/B-03 unresolved | **CONFIRMED** | ΔU=fixed−dynamic 单向；CI 未定义 | 选择偏倚与装置分叉 | `Z_p` 配对；CGM `CI⊂[±tol]` 双向 |
| A-06 successor lock | **CONFIRMED** | CODE_CHANGE 迁移持仓未写锁 | successor 可新开 parent | LockGroup + 原子迁移 owner/lock |

## 已维持 PASS

P10 episode-day support；ExecutionAllocatedBudget 伪代码顺序；zero-target Intent；Gate 9 cutoff。
