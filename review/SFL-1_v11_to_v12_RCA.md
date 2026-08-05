# SFL-1 v11 → v12 RCA（非规范）

> Canonical 以 `SFL-1_v12.md` 为准。  
> 依据用户 **SFL-1 v11.0 SPEC 深度 Review**，逐项核对后处置。

---

## 0. 核对总表

| Review 编号 | 核对结论 | v11 证据 | v12 处置 |
|---|---|---|---|
| A-01 Gate8 Population | **PARTIAL** | 已有 Population_fill + `DIAGNOSTIC_POLICY_RETURN`，但未写成 8A/8B 正式对象，附录 B 仍一行混写 | Closure1：Gate8A 主庭 + Gate8B 强制诊断 |
| A-02 P10 spawn identity | **PARTIAL** | 已有 ControlFingerprint/Jaccard/cooldown；开放中重叠&lt;0.80 时是否并存未唯一 | Closure2：supersede 强制关闭后再 spawn；identity key 冻结 |
| A-03 Impact 双计 | **CONFIRMED** | §14.1 step6“成交价=VWAP 再计冲击”与 \(P^{ex}\) 用语可解为两次冲击 | Closure3：成交执行价含冲击一次；报告分解不得再扣 |
| A-04 trade/tax lot | **CONFIRMED** | 同时有 AvgCost 与 FIFO tax_lots，未声明卖出成本口径；零碎股估值未完全 | Closure4：trade_lot / tax_lot / AvgCost 职责唯一 |
| A-05 Gate9 资本 | **基本 CLOSED** | 已有 DeployedCapitalBase=max 并发；MES 量纲可再钉死 | Closure5：冻结该唯一分母与 2% 含义 |
| A-06 版本路径 | **Hygiene** | 全文仍为 v11 字符串/函数名 | Closure7：统一 SFL-1_v12.0 |
| B-01 条件/政策分离 | 同 A-01 | — | Closure1 |
| B-02 P10 分母 | 同 A-02 | — | Closure2 |
| B-03 bootstrap 单位 | **基本 CLOSED** | §20.2.4 已整块抽样；需升为唯一禁止条款 | Closure6：强化唯一性 |
| B-04 Gate9 MES 量纲 | 同 A-05 | — | Closure5 |
| B-05 公司行动入净收益 | **PARTIAL** | payment_date/tax lots/零碎已有；缺卖出成本与 NAV 勾稽 | Closure4 |
| Gate1 cohort 同构 | **基本 CLOSED** | §8.3.3-1c + Gate1 正文已有 | 伪代码/附录显式唯一回引 |
| 非阻断 8.1–8.5 | 维持 | — | 不扩展 |

**总判：** v11 已是 HIGH-QUALITY FREEZE CANDIDATE；本版仅做 **Freeze Closure Release**，不新增指标/状态/对照/Gate。

---

## 1. Closure 细节

### Closure 1 — Gate 8A / 8B
- **8A Conditional Execution Court（正式主）：** Population_fill；Outcome=NetAlpha20；进 IUT 主终局。
- **8B Execution Policy Diagnostic（强制诊断，非第二主 Gate）：** 全部理论信号；未成交 Outcome=0；强制首页报告 FillRate 与 PolicyNetAlpha20；**不得**替代 8A，**不得**与 8A 混同一 CI。
- \(\theta_{ADV}\) 仍在隔离账户、全理论信号上，与 8A 组成 IUT 的第二共同主 estimand（逆向选择）。

### Closure 2 — P10 Identity Protocol
Identity key = `(spec_version, real_theme_id, fold, control_index, spawn_index)`。  
Continuation：开放 episode 且 Jaccard(members_D, base_membership)≥0.80 → 推进。  
Supersede：开放但 Jaccard&lt;0.80 → 立即 `CLOSED_SUPERSEDED`，再按 cooldown 规则决定是否 spawn。  
同 fingerprint 关闭后 cooldown=5 交易日。  
禁止跨 real_theme 合并；禁止日更键产生新 episode。

### Closure 3 — Impact Single-Count
选定方案 A：`P_exec = VWAP × (1 ± ImpactBps/10^4)`；fill_qty 与净成本均用该价；§14.4“容量冲击”仅为成本分解披露，**禁止**在 \(R^{net}\) 中再次扣除。

### Closure 4 — Lot Accounting
- `trade_lots`：成交批次（qty, P_exec, fees, settle_date）→ T+1 / SellableQty。
- `tax_lots`：FIFO，仅分红税与公司行动税务；税率取 `tax_rule_version` 在 payment_date 的 PIT 表；缺省支付日预扣，除非税表要求卖出追缴。
- 组合卖出成本 / \(R^{net}\)：**移动平均 AvgCost**（由 trade_lots 维护）。
- 零碎股：账本允许；卖出向下取整 100 股；残留按最新收盘计入 NAV；仅在事实表现金兑付或再行动合并时清除。

### Closure 5 — Gate 9 Capital
唯一：`DeployedCapitalBase = max_t ConcurrentGrossNotional_t`；两影子组合初始现金=该值；\(\Delta CumExcess\) 为两 NAV 路径累计超额之差，MES_pass=0.02 即 2 个百分点；禁止 Σ 全部订单金额、任意固定外部资本等替代。

### Closure 6 — Bootstrap Unit
§20.2.4 稳健性 bootstrap **唯一**以 ID1 persistent overlap component 整块抽样；逐 episode 抽样非法。

### Closure 7 — Version Hygiene
`spec_version="SFL-1_v12.0"`；`load_or_build_nulls_v12`；产物/路径/伪代码一律 v12。
