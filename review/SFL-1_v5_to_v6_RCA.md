# SFL-1 v5 → v6 五项闭合问题 RCA 与裁决

> 性质：非规范性 Review/RCA。正式规则以 `SFL-1_v6.md` 为唯一 Canonical 来源。

## 总结

|编号|v5 问题|根因|v6 裁决|
|---|---|---|---|
|1|双实现要求与单实现降级意图冲突|可靠性目标（可重放）被错误绑定到实现组织形式（双重开发）|单一生产实现 + 隔离环境独立重发审计；全篇删除双 Agent 强制要求|
|2|LAC 声称用于退出，却没有 LAC 正式变量|冻结因果归因与动态运行传感器共用“迁移/衰退”语言，缺少变量 namespace|冻结迁移族与动态健康族彻底分轨；新增四个 LAC 指标、`LACDecay` 与 COLLAPSE 独立分支|
|3|H-ROLE 没有独立 Gate|假设按叙事顺序拆分，Gate 却按交易流水线合并，导致角色定义有效性被迁移结果隐含代理|新增独立 Study 2 / Gate 3；迁移及后续 Gate 顺延；核心家族扩为9个 Gate|
|4|Track B 的 embedding PIT 条件不可落地|把“最严格防泄漏”误写成“所有历史日期必须使用同一现代模型”，没有区分无训练词法轨与模型 regime|正式历史主口径改为 PIT Hashing-TFIDF；embedding 仅在训练截止日之后的新 measurement regime 使用；Track B 样本不足不阻断 Track S|
|5|null 计算复杂度失控且缓存边界未定义|统计定义按数学对象描述，没有同时设计等价的 sufficient-statistics 执行图；P10 形成二阶 null 爆炸|冻结 exact cache DAG、O(1) LOO、排序 order-statistics、5-fold cross-fit P10、naive 等价验收和运行 SLO|

---

## RC1：双 Agent 降级没有形成单一规范

### 具体冲突

v5 的 Run-in、漂移、Kill、§26 和 Freeze checklist 仍把“双独立实现一致”作为硬条件。只在局部备注声明“单实现 + 独立重发”无法改变这些规范性消费者。

### 根因

原设计真正要控制的是三类风险：

1. 输入快照被静默修改；
2. 同一代码存在非确定性；
3. 生产输出无法由冻结物重建。

双实现还能发现“两个团队对 SPEC 理解不同”，但成本显著更高。v5 将上述两类保证混成一个机制，导致一旦降低组织成本，全文可靠性条款失去统一替代物。

### v6 处理

- 只保留一个 Canonical 生产实现；
- 独立审计 runner 使用独立凭据、洁净容器、只读原始快照，不读取生产中间缓存；
- runner 可使用同一冻结代码（这不是双实现），但必须从原始输入重新计算并比较逐层哈希；
- 每日审计 D−1，20 日 Run-in 审计每日；正式期 T+1 12:00 前完成；
- 离散输出完全一致，浮点按容差表；不一致进入 `REPLAY_MISMATCH`；
- 连续或未闭合不一致进入 QUARANTINE/Kill。

### 明确损失

单实现 + 重发无法发现“SPEC 与代码以同一种方式写错”的共同语义错误。因此 v6 用 Canonical 静态闭包、黄金样例和 naive reference test 补偿，但不虚假宣称其等价于双实现。

---

## RC2：LAC 职责混乱

### 具体冲突

v5 一方面写 LAC 用于角色迁移和退出；另一方面规定 LDS/CDS/RMI/CapacityBreadth 只在冻结 Entry Cohort 上计算，且状态机不得被 LAC 改写。结果是“LAC 用于退出”没有任何可执行变量。

### 根因

两种不同 estimand 未被命名：

- **冻结 estimand：** D 日进入的成员是否发生角色迁移，用于 H-MIG、Gate 与可复现实验；
- **动态 surveillance：** 持仓期当前活跃成员是否衰退，用于风险退出。

前者必须防 post-treatment membership 改写；后者必须允许成员变化。用同一“迁移指标”语言描述两者，导致对象、集合和消费者错配。

### v6 处理

冻结族保留：

    LDS^E, CDS^E, PriceShare^E, RMI^E, CapacityBreadth^E

动态 LAC 族新增：

    LACBreadth
    LACTurnBreadth
    LACRelReturn5
    LACAttrition10

正式动态衰退事件：

    LACDecay =
      连续2日(LACBreadth<0.20 AND LACTurnBreadth<0.30 AND LACRelReturn5<0)
      OR
      (LACAttrition10>0.40 AND LACRelReturn5<0)

`LACDecay` 只进入 COLLAPSE 的独立分支与 Leg X；不得进入入场、H-MIG、RMI 或 Gate 3/4。所有变量进入 owner/集合映射、参数表、生产流程和哈希审计。

---

## RC3：H-ROLE 被 H-MIG 隐含代理

### 具体冲突

“角色可以分离”与“需求发生迁移”是两个命题。迁移存在不证明角色定义有效：同一动量排名拆成两个名字，也可能产生看似迁移的相对路径。

### 根因

Gate 按策略流水线设计，而非按正式假设一一映射。H-ROLE 没有独立 Population、Estimand、Null、MES 和 Failure state。

### v6 处理

新增独立 Study 2 / Gate 3：

- 使用 D+1:D+5 的**未来功能结果**，避免用定义角色的 D 日构造变量自证；
- Attention contrast：未来注意力功能 L − K；
- Capacity contrast：未来容量吸收功能 K − L；
- 两者均对匹配 null 标准化，MES 均为0.05；
- 两者均 PASS 才 `PASS_ROLE_SEPARATION`；任一有反向证据才 `FAIL_ROLE_SEPARATION`；其余 INCONCLUSIVE；
- P5 角色随机化为强制安慰剂；
- H-MIG 独立移至 Gate 4，后续 Gate 顺延为 5–9。

---

## RC4：Track B 防泄漏规则不可执行

### 根因

训练截止日晚于历史起点确实可能泄漏，但“只允许一个 embedding 覆盖全部历史”不是唯一正确解。它错误地把模型 regime 与策略样本 regime 强行绑定。

### v6 处理

Track B 分两种互斥模式：

1. `B-LEX`（历史正式主模式）：字符 3–5 gram hashing（固定 2^18 维、固定 seed）+ 截至 D−1 expanding IDF；不训练词表、不接触未来文档；
2. `B-EMB`（可选 measurement regime）：某模型只可用于 `training_cutoff < regime_start` 的区间；换模型即新 regime，证据不无条件池化。

现代 embedding 不得回填其训练截止日前历史。无法满足 B-EMB 时自动使用 B-LEX，不使 Track B 整体失效。Track B 样本不足输出 `TRACK_B_INSUFFICIENT`，不阻断 Track S 主轨；只有 Track B 自己通过全部相关 Gate 后才能并入生产。

---

## RC5：null 是数学定义，不是可执行计算图

### 根因

v5 逐对象描述 200 个伪题材及 LOO，工程实现若机械嵌套循环，复杂度接近：

    题材日 × 200伪题材 × 成员 × 候选LOO × 完整链

P10 若再为每个伪题材生成第二层 200 controls，出现二阶爆炸。

### v6 处理

冻结计算 DAG：

1. `StockDayFeatureStore`：全市场股票日特征一次计算、滚动增量；
2. `DonorIndex`：行业×板块分层与马氏近邻索引每日一次；
3. `NullMembership`：每真实题材日一次生成，IES/IDS/CCS/GCC 共用；
4. `PseudoThemeStats`：每伪题材一次聚合充分统计量；
5. LOO 对可加计数/和使用减法精确更新；GCC 用排序 order statistics 精确删一；
6. 所有 null 分位用排序数组和 midrank 二分查找；
7. P10 使用固定 5-fold cross-fitting，不生成二阶 null；
8. 缓存键包含全部输入/代码/参数/成员哈希；任一变化整层失效，不允许近似陈旧缓存。

准确性闸门：

- 优化 DAG 对 naive reference：离散结果/集合/判词完全一致，浮点满足 §26 容差；
- P10 cross-fit 对独立二阶 null 基准：信号率差≤1个百分点、分位 Spearman≥0.99；
- 任何不满足均不得使用优化路径。

运行 SLO：

- 交易日全链 p95≤60分钟、p99≤90分钟；
- 21:00 前未完成的题材不降采样、不用旧缓存，记 `COMPUTE_TIMEOUT`；
- 超时题材占比>10%触发 QUARANTINE。
