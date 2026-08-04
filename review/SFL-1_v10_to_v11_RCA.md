# SFL-1 v10 → v11 RCA（非规范）

> 本文件**不是** Canonical SPEC。Canonical 以 `SFL-1_v11.md` 为准。  
> 依据用户提交的 **SFL-1 v10.0 SPEC 深度 Review**，对每条问题先核对 v10 是否真实存在，再给出 RCA 与 v11 修复落点。

---

## 0. 核对总表

| Review 编号 | 核对结论 | v10 证据 | v11 处置 |
|---|---|---|---|
| A-01 确认换尺循环 | **CONFIRMED** | §3.5.1 FrozenMetricBackcast 覆盖“确认斜率所用 IDS 历史”；§5.1 τconfirm 用 IDS 斜率 | Pack1：确认永久用归档临时量尺；backcast 不回写确认谓词 |
| A-02 P10 spawn 未定义 | **CONFIRMED** | §6.2“新发现时 spawn”+ spawn_index，无算法 | Pack2：ControlFingerprint / cooldown / spawn 算法 |
| A-03 Gate1 真实/null 不对称 | **CONFIRMED** | Gate1 null 冻结；真实走 §3.5 动态映射 | Pack2：DISCOVERY_RESEARCH_COHORT_D |
| A-04 ThemeRoom 合并冲突 | **CONFIRMED** | §5.4 并入 u1 ThemeRoom；§15.0 origin 入场簇 | Pack5 旁支：origin risk 永久不变 |
| A-05 对照提前成交共享退出 | **CONFIRMED** | §8.1 允许对照先成交；退出对齐真实 τentry | Pack5：双方统一从真实 τentry 执行 |
| A-06 ID2 对 Gate1–5 | **CONFIRMED** | §20.2.1“信号日所属自然月”；Gate1–5 无信号日 | Pack6：每 Gate index_date |
| A-07 开盘涨停冲突 | **CONFIRMED** | §14.3 09:30=涨停拒单 vs 开板后参与 | Pack3：09:30–09:34 连续封死才拒 |
| A-08 冲击循环/未来日额 | **CONFIRMED** | ImpactBps=f(该日实际成交金额) 且决定 P^ex | Pack3：顺序冲击模型 |
| A-09 最低佣金 vs Remaining | **CONFIRMED** | 全日合并一次最低佣金 + 分钟扣 Remaining | Pack3：预留 + 日终结算 |
| A-10 T+1 伪代码顺序 | **CONFIRMED** | §25.3 roll 在当日买入后 | Pack4：开盘前 roll |
| A-11 公司行动时点/行权 | **CONFIRMED** | available_at 收盘后过账 vs 除权开盘前；无 payment_date；RIGHTS 无政策 | Pack4：开盘前生效 + payment_date + 默认放弃行权 |
| A-12 Gate8 Population | **CONFIRMED** | §0.3 未成交不进 NetAlpha20；附录B含未成交 | Pack7：主效果 conditional-on-fill |
| A-13 no-gap 组合干扰 | **CONFIRMED** | 信号级配对但未隔离生产现金/槽位 | Pack7：隔离研究账户 |
| A-14 Gate9 资本分母 | **CONFIRMED** | InitialCapital=Σ全部 OriginalTargetValue | Pack8：最大并发部署资本 |
| B-01…B-08 | 随对应 A 项关闭 | — | 同 Pack |
| 非阻断 7.1–7.5 | 维持 | — | 不升格 |

---

## 1. A-01 确认换尺循环

### 根因
冻结动作发生在确认**之后**，又允许 backcast 覆盖确认斜率所用 IDS，使“是否确认”可被后创建量尺改写。

### 修复
- `τconfirm` **永久**仅由确认前已归档 `DAILY_NULL` 临时量尺决定。
- `FrozenMetricBackcast` **禁止**回写确认 IDS 历史与确认谓词结果。
- 仅允许 backcast 确认后变量（GCC/RMI 及确认后状态机输入）。

---

## 2. A-02 P10 spawn

### 根因
“新发现”是自然语言，无稳定对象映射。

### 修复
定义 `ControlFingerprint = hash(sorted(control_index membership on D))`；仅当 fingerprint 相对前一交易日为新，或同 fingerprint 上一 `P10_NULL_EPISODE` 已 CLOSED/INVALID 且过 cooldown 后，才 `spawn_index += 1`。

---

## 3. A-03 Gate1 不对称

### 根因
null 路径 D 日冻结；真实路径仍走动态 §3.5。

### 修复
对每个候选日 D 的真实题材同步冻结 `DISCOVERY_RESEARCH_COHORT_D`（Base/L/观察集合/分母），Gate1 D+1:D+5 真实与 null 使用同一冻结制度。

---

## 4. A-04 ThemeRoom

### 根因
账务合并指针被误写成风险归属迁移。

### 修复
`MERGED_INTO` 仅执行/报告指针；origin ThemeRoom 归属永久不变。

---

## 5. A-05 对照时钟

### 根因
“可成交性”与“持有时钟”混用。

### 修复（同日可执行对照）
对照仅从真实 `τ_entry` 当日开始尝试成交；退出时钟与处理侧对齐。D+1 对照可成交但真实未成交时，只记可成交诊断，不提前建仓。

---

## 6. A-06 index_date

### 修复
附录 B 每 Gate 唯一 `index_date`；`ID2 = YYYY-MM(index_date)`。

---

## 7. A-07…A-11 执行与账本时钟

### 修复摘要
- 涨停拒单：09:30–09:34 连续无量封死。
- 冲击：截至当前分钟累计拟成交顺序模型；不回写过去分钟；不使用全日终总额决定当前分钟价。
- 最低佣金：日初按母订单预留，分钟用比例费用推进，日终一次结算最低佣金并调整 Remaining/现金。
- 每日顺序：开盘前公司行动生效 → Unsettled→Settled → SellableQty → 盘中执行 → 收盘后信号/次日订单。
- `payment_date`：现金分红仅 payment_date 开盘前入 AvailableCash；RIGHTS 默认永不行权；零碎股保留但不可卖出非整手，卖出按 100 股向下取整。

---

## 8. A-12…A-14 Gate8/9

### 修复
- Gate8 主 `NetAlpha20`：**conditional-on-fill**；未成交只进 θ_ADV 与诊断。
- no-gap：隔离研究账户，固定 ParentOrderTargetValue，不消费生产现金/ThemeRoom。
- Gate9：`DeployedCapitalBase = max_t ConcurrentGrossNotional_t`（tape 上已部署总名义峰值）；Sharpe/ΔCumExcess 使用该分母对应的现金账户初始化。

---

## 9. v11 范围纪律

严格限定为 **Final Clock–Execution Closure**：不新增题材指标、状态、对照或 Gate。
