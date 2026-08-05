# SFL-1 v17 → v18 RCA（非规范）

> Canonical 以 `SFL-1_v18.md` 为准。  
> 审查建议命名 “v17.1”；按用户要求产出完整 **v18.0 Tax-Slice–Identity Collision Closure**。  
> 范围纪律：不新增指标 / 角色 / 状态 / Gate / 退出腿。

## 核对总表

| 项 / 编号 | 核对 | v17 证据 | 根因 (RCA) | v18 处置 |
|---|---|---|---|---|
| PREOPEN_ACTION_CUTOFF | **PASS（维持）** | §4.3 已唯一 | — | 不改 |
| Settled stock proceeds | **PASS（维持）** | SellNet vs SettledCash | — | 不改 |
| Symmetric unresolved guard | **PASS（维持）** | Z_p + CI� SettledCash | — | 不改 |
| Symmetric unresolved guard | **PASS（维持）** | Z_p + CI⊂[±tol] | — | 不改 |
| A-01 tax-lot mapping | **CONFIRMED** | 仅 source_trade_lot_ids | trade≠tax 生命周期 | `source_tax_lot_alloc` 冻结 |
| A-02 partial disposal | **CONFIRMED** | 单一 actual_disposal_date_l | 部分卖出无表示 | disposed/remaining slices |
| A-03/B-01/B-02 planned覆盖 | **CONFIRMED** | max(holding, fixed20) 在卖出后仍可取20 | 假想日覆盖实际处置 | disposed→只用 disposal；else 才 max |
| A-04 TradingDays | **CONFIRMED** | 调用无端点定义 | 临界档位分叉 | §6.10 / §15.5.2：`a<d≤b` |
| A-05 LockGroup 时点 | **CONFIRMED** | “迁移完成前”无形成日 | D0 vs D20 分叉 | `LockGroupFormationPreOpen` |
| A-06/B-03 owner collision | **CONFIRMED** | 单 owner，无冲突规则 | 双 parent 归属不唯一 | 方案 B：`CORP_ACTION_LOCK_COLLISION`→`LEDGER_INVALID` |

## 设计取舍

Owner collision 采用**方案 B（保守）**：不引入 multi-owner 子账；碰撞即 `LEDGER_INVALID`，禁止实现者选择保留哪个 owner。
