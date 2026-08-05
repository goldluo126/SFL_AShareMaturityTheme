# SFL-1 v12.0 正式交易规格书

## 题材发现—确认分离 × 角色迁移 × 容量时钟 × 可实现交易

**英文代号：** Theme Discovery–Confirmation, Role Migration and Capacity Clock Framework
**简称：** SFL-TDCRMC
**版本：** v12.0 Freeze Closure Release
**状态：** `FREEZE-CANDIDATE`；规范文本已自包含（v12 Freeze Closure Release）。仅在 §4 数据闸门、§8.3 生成器验收（NG01–NG07）、§17 机制研究闸门、§20.2 正式推断算法验收、§20.4-bis 可行性核查、§21.1 分区清单物化与附录 C 全部通过后方可冻结
**适用市场：** 中国A股，日频信号、分钟级执行、仅做多
**信号时钟：** D日收盘后计算，D+1执行
**研究目标：** 放弃预测底部；确认题材共同需求已经发生，识别边际需求从注意力载体向容量载体传播的早期阶段，赚取容量定价尚未完成的中段收益
**非承诺：** 本文是可回测、可复现、可证伪的研究与交易规格，不代表该策略已经被实证证明有效
**修订范围纪律：** 本版本严格限定为 Freeze 最终闭合（estimand / identity / impact / lot / 资本 / bootstrap / 版本卫生）；不新增题材指标、角色、状态、对照、Gate 或退出腿。

## Canonical 与自包含声明

1. **唯一规范源：** 本文件是 SFL-1 v12.0 的唯一规范性来源。实现、回测、验收、影子运行和正式判词不得引用任何早期版本、审查报告、变更日志或口头解释补全规则。
2. **规则闭包：** 全部正式对象、集合、算子、时间点、状态转移、随机过程、执行路径、统计装置、异常处置和治理判词均在本文正文或本文附录中展开。内部 `§` 引用仅用于定位本文内容，不构成外部依赖。
3. **配置边界：** `params.yaml`、`null_generator.yaml`、`kill_criteria.yaml` 等机器文件是本文已定义规则的序列化冻结物，不得覆盖或补充本文未定义的规范。`fee_schedule.yaml` 是按 §14.4 schema 接入的 PIT 制度事实。若规则型机器文件与本文冲突，以本文为准并判工程失败。
4. **数据边界：** 市场数据、PIT 快照和样本日期属于输入事实，不属于外部规范。所有输入 schema、可用时钟、解析规则和失败处置由本文定义。
5. **禁止隐式继承：** 任何省略规则正文、转而要求实现者从其他版本或历史实现补全的表述均无规范效力；发现即判 Canonical 检查失败。
6. **修订纪律：** 后续修订必须附同类缺陷全域扫描；变更历史应保存在独立的非规范性 Review/CHANGELOG 文档中，不得混入 Canonical SPEC。

---

# 0. 先验边界与最终主命题

## 0.1 不能诚实声称的事情

本规格书不声称能够从公开日频数据中直接识别：

- "主力集团"是谁；
- 某笔买入来自机构、游资、量化还是散户；
- 某个题材在基本面上一定正确；
- 龙头断板一定代表退潮；
- 大市值、高成交额股票一定是中军；
- 共同上涨一定来自共同信息，而不是共同注意力或价格反馈。

龙虎榜、融资余额、订单金额分档等只能提供受限代理，不能将资金身份当作核心因果变量。龙虎榜类数据仅允许进入 §17.5 的机制诊断研究（EXPLORATORY），不得进入任何信号、排序、门槛或退出规则。

**对手方与竞争衰减声明：** 本策略假定的收益对手方是（i）在容量定价完成后追高买入的迟到注意力资金，与（ii）在题材确认前低估共同需求持续性的过早卖出者。该玩法（题材扩散 + 容量载体补涨）在 A 股短线与量化群体中属公开知识，**alpha 竞争衰减（同类策略拥挤导致中段被抢跑）是本框架显式登记的失败条件**：若 FORWARD 期主终点相对 VALIDATION/HOLDOUT 系统性衰减而机制变量（IDS/\(RMI^E\)/GCC 路径）不变，应优先解释为拥挤衰减而非机制消失，处置见 §21.7 Kill Criteria 与 §21.6 漂移条款。

## 0.2 A股题材行情的可检验主命题

> 在一个于D日前已经被定义的候选题材集合中，少数高显著性股票首先完成注意力点火；如果剔除这些股票和待买股票后，其他成员仍出现独立扩散，并且边际需求开始由低容量注意力载体向高容量核心载体迁移，而容量核心尚未完成主要价格扩张，则在按 §14.1 执行规则于信号有效期内可成交的条件下，D+1（或有效期内重试日）买入容量核心可能获得扣除真实成本后的条件超额收益。

形式化为：

\[
E\left[
R^{net}_{i,\tau_{entry}:\tau_{entry}+19}
\mid
Discovery_{u,D-1},
Confirmation^{-i}_{u,D},
CapacityClock^{-i}_{u,D},
CCS\_level_{i,D},
Filled_{i}
\right]
>
E\left[
R^{net}_{control,\tau_{entry}:\tau_{entry}+19}
\mid Filled_{control}
\right]
\]

其中容量载体强度唯一使用 \(CCS\_level_{i,D}\)；收益窗按 §0.3 与实际成交对齐，处理和对照两侧均以可成交为条件。

主命题不是"热门题材会上涨"，而是：

\[
\text{独立扩散}
+
\text{容量需求加速}
+
\text{价格尚未充分扩张}
\Rightarrow
\text{可交易的条件优势}
\]

**共同支持限定：** 本命题及全部正式判词只适用于能够按 §8.3 构造出不少于 100 个合格匹配伪题材的题材日。无法匹配的题材日（NON_EVALUABLE）不产生信号、不进入样本，判词语言不得外推到该域。

## 0.3 唯一允许决定策略生死的主终点

\[
NetAlpha20
\]

即：真实可执行买入后，自成交日起持有20个交易日的净超额收益。

### 0.3.1 收益窗与持有计数

- \(\tau_{entry}\)：**首次非零成交日**（D+1，或 §13.3 有效期内的重试成交日）；成交按 §14.1 逐分钟流式参与，入场成本 = 有效期内全部成交的数量加权净成本（含部分成交与追补）；
- 持有计数：\(\tau_{entry}\) 计为持有第 1 日；有效期内后续追补成交共享同一退出时钟；
- 入口验证阶段（§16.1）：第 20 个持有交易日收盘生成退出信号，第 21 个交易日 14:30—15:00 按 §14.3 流式参与卖出（顺延规则同节）；
- \(R^{net}_i\) = 卖出净所得 / 买入净成本 − 1；买入净成本与卖出净所得均按 §14.1/14.3/14.4（冲击单次计入）；卖出成本基础使用 §15.3.1 **AvgCost**；
- 未在有效期内成交的信号不进入 **Gate 8A** 主 NetAlpha20 样本；其占比与 **Gate 8B All-Signal Policy Return**（未成交记 0）按 §18 Gate 8 / §20.6 强制报告；Gate 8 逆向选择对照防止 REJECT_GAP 掩盖坏规则。

### 0.3.2 episode 层基准（信号归因基准）

同一信号、同一匹配集合内，200 个匹配伪题材容量篮子的假想净收益均值：

\[
NetAlpha20_{i,u}
=
R^{net}_{i,\tau_{entry}:\tau_{entry}+19}
-
\frac{1}{|C^{valid}|}
\sum_{c\in C^{valid}}
R^{net}_{basket(c),\tau_{entry}:\tau_{entry}+19}
\]

**对照篮子执行语义：**

1. 篮子成员 = 该伪题材（§8.3）在配对确认日冻结的 \(K^E\) 中，于信号日 D 满足 §9.1 当日交易资格的 \(K^{trade}\)；
2. **同日可执行对照（v12 唯一）：** 对照篮子**仅从真实信号的 \(\tau_{entry}\) 当日**开始尝试买入与流式参与；在真实 \(\tau_{entry}\) 之前的任何尝试日，对照只做 09:35 可成交性诊断登记（`CONTROL_FILLABLE_BEFORE_ENTRY`），**禁止**提前建仓或累计 RemainingTargetValue 扣减；
3. 自真实 \(\tau_{entry}\) 起：对照成员按 §14.1 人民币母订单完整算法执行；若真实信号在 \(\tau_{entry}\) 后仍有 D+2/D+3 追补尝试，对照在相同尝试日同构重试并继承 RemainingTargetValue；对照不得在真实未尝试日自行成交；
4. **篮子 TargetValue：** 整个对照篮子总目标金额 = 配对真实信号的单笔 `ParentOrderTargetValue`；成员等权拆分人民币；禁止每只对照各获一份；
5. 全部拒绝判定同构适用；
6. REJECT_GAP：与真实信号相同状态、对照股自身板块的同一阈值表；
7. 决策被拒成员从篮子剔除且额度不重分；部分成交按实际金额计权；未成交为现金收益 0；篮子收益 = 成交成员金额加权；
8. 卖出：与真实信号同一卖出日（共享退出时钟），§14.3 + T+1 + §14.4 同构；持有计数自真实 \(\tau_{entry}\) 起；
9. \(C^{valid}\) = 成交成员非零的篮子；\(|C^{valid}|<100\) → episode 层主终点 `NON_EVALUABLE`；
10. **禁止**“对照提前买入却共享处理侧退出时钟”的实现。

### 0.3.3 组合层基准（业绩归因基准）

组合日净值对**暴露匹配基准**的净超额：

\[
R^{BM}_t = w_{t-1}\times R^{\text{全A等权}}_t
\]

其中 \(w_{t-1}\) 为前一交易日收盘时组合风险资产市值 / NAV，现金部分两侧同记零收益。全A等权指数按 DG15 自建（§4.4）。**满仓全A等权、沪深300与中证1000对照仅作次要报告，不进入主判词。** 风险暴露不匹配的满仓基准禁止用于正式验收。

动态退出只能在入口、题材条件和容量载体均独立通过后进行验证，不能用退出系统掩盖无效入口。

---

# 1. A股的因果结构

## 1.1 价格是受约束的边际需求清算

短期价格变化近似由以下关系决定：

\[
\Delta P
\propto
\frac{
MarginalBuyDemand-MarginalSellSupply
}{
AvailableLiquidity
}
\]

A股中以下制度和参与者结构会放大该机制：

1. 做空和融券受到约束；
2. 股票实行T+1；
3. 涨跌停限制会延迟价格发现并造成排队；
4. 个人投资者、短线资金、量化与机构的容量和持有期限高度异质；
5. 榜单、涨停、新闻、政策标签和社交传播会集中有限注意力；
6. 相同价格的订单遵循价格优先、时间优先，强势涨停股票的可交易性与未来表现存在逆向选择。

因此，策略不能将"收盘涨停"直接等同于"次日可以按开盘价买到"。

## 1.2 题材的本质：临时形成的共同解释协议

题材不是一个永久不变的股票篮子。

题材是：

> 市场参与者在特定时间内，临时接受的一套共同解释；该解释降低寻找标的、理解信息和猜测他人行为的成本，使边际需求集中于一组可识别股票。

题材同时包含三层：

    语义层：市场在交易什么故事
    行为层：哪些股票此刻真的被当作该故事交易
    角色层：这些股票分别承担点火、空间、扩散、容量和趋势功能

供应商概念只能提供语义候选，不能直接代表当日有效题材。

## 1.3 谢林点不要求资金主体事先串联

共同需求可以在没有集中组织者的情况下形成：

    叙事或信息冲击
    → 降低认知与搜索成本
    → 资金集中于高辨识度股票
    → 价格和涨停提高可见度
    → 更多参与者预期他人继续关注
    → 共同注意力和价格反馈扩散

所以系统不检验"主力是否集团作战"，而检验：

- 是否出现可见的点火者；
- 点火之外是否有独立扩散；
- 扩散是否进入容量载体；
- 容量载体是否仍有后续可实现收益。

## 1.4 题材行情中的五类市场功能

|角色|市场功能|不等同于|
|---|---|---|
|点火者 Igniter|最早出现异常定价|最终龙头|
|空间载体 Space Leader|定义连板高度和情绪上限|最适合买入股票|
|注意力核心 Attention Leader|吸收最高可见度与成交关注|机构持仓核心|
|容量载体 Capacity Carrier|能够承载较大边际需求|简单的大市值股票|
|趋势载体 Trend Carrier|在分歧后持续保持承接和相对强度|低位补涨股|

同一股票可承担多个角色，角色也可在episode内迁移。角色是连续竞标的功能位置，不是出生时的任命；先涨停者是点火股，即注意力角色的第一候选人，而非终身龙头。

## 1.5 真正的alpha来源

本框架假定的alpha不是题材标签本身，而是：

\[
Alpha
=
\text{边际需求在流动性层级间的传播}
-
\text{价格完全反映该传播的速度}
\]

如果注意力点火后没有独立扩散，则是单股事件或情绪脉冲。

如果扩散后没有容量需求接管，则可能是短线小票内部循环。

如果容量需求已经高度拥挤，则没有剩余中段。

---

# 2. 因果图与独立假设

## 2.1 因果图

    外部信息、政策、产业叙事或价格事件
                   │
                   ▼
         共同注意力 / 共同解释
                   │
                   ▼
        点火者与注意力核心形成
                   │
          ┌────────┴────────┐
          ▼                 ▼
   独立成员扩散         单股独舞/假启动
          │                 │
          ▼                 └──> 失败
  题材共同需求被确认
          │
          ▼
不同容量约束的资金寻找可承载标的
          │
          ▼
容量载体异常需求加速
          │
          ▼
定价权由注意力核心向容量核心扩散/迁移
          │
     ┌────┴─────┐
     ▼          ▼
价格尚未充分反映   已全面拥挤
     │          │
     ▼          └──> 无可交易优势
 D+1买入容量载体

## 2.2 独立假设

### H-DISC：发现有效性

D日前定义的题材候选集合，比同日匹配的伪题材更容易在随后出现非龙头独立扩散。

### H-CONF：确认有效性

剔除注意力核心和候选股票后，独立扩散确认能够降低未来5日假突破率。

### H-ROLE：角色可分离性

注意力角色和容量角色在横截面特征与时间路径上可以稳定区分，不是同一涨幅排名的不同名称。

### H-MIG：角色迁移命题

题材确认后，容量载体的定向需求份额和正向价格贡献存在系统性上升，并非仅由龙头自身制造。

### H-CLOCK：容量时钟命题

在已确认题材内，确认日容量时钟水平（\(GCC^{-i}\)）与个股容量强度（\(CCS\_level\)）的交互能够预测未来20日机制毛超额；生产路径另要求时钟“仍在上升”（\(GCCSlope>0\)），该上升条件作为 Gate 5 稳健性 guardrail 验证，不单独构成第二共同主 estimand。

### H-CARRIER：容量载体选择特异性

在同一确认题材与同一信号日内，完整 EntryScore 选择的容量载体未来净收益高于容量匹配随机选择、全部容量股等权和题材内相对强度选择。

### H-INT：题材过滤交互命题

容量载体信号在已确认且发生容量扩散的题材中，比在匹配控制题材中更有效。

### H-EXEC：可实现性命题

加入真实开盘排队、涨跌停、分钟VWAP、费用、滑点、冲击、停牌及退出顺延后，净超额仍然为正。

### H-EXIT：退出增量命题

双层退出相对固定持有20日能够改善风险调整后净收益，而不是只减少右尾收益。

## 2.3 假设—研究—Gate 唯一映射

|正式假设|唯一主研究|唯一正式 Gate|失败判词|
|---|---|---:|---|
|H-DISC|§17.1 发现部分|1|FAIL_DISCOVERY_UNIT|
|H-CONF|§17.1 确认部分|2|FAIL_CONFIRMATION|
|H-ROLE|§17.2|3|FAIL_ROLE_SEPARATION|
|H-MIG|§17.3|4|FAIL_ROLE_MIGRATION|
|H-CLOCK|§17.4|5|FAIL_CAPACITY_CLOCK|
|H-CARRIER|§18 Gate 6 生产排序对照|6|FAIL_CARRIER_SELECTION|
|H-INT|§18 Gate 7 2×2|7|FAIL_FILTER_INTERACTION|
|H-EXEC|§18 Gate 8|8|FAIL_EXECUTION / FAIL_ADVERSE_SELECTION|
|H-EXIT|§17.6、§18 Gate 9|9|FAIL_EXIT_INCREMENT|

任何假设不得由其他 Gate 的结果隐含替代；下游 Gate 只在全部上游 Gate PASS 后开庭。

---

# 3. 题材发现与确认严格分离

## 3.1 核心时间纪律

对于D日确认、D+1执行的交易：

    截至D−1：决定"观察哪些股票构成候选题材"
    D日：决定"这些预先存在的候选是否发生独立扩散和容量启动"
    D+1：执行交易

禁止：

    D日用涨停股票创建题材
    → D日再用这些涨停证明题材启动
    → D+1交易

任何在D日新创建的题材，最早只能在D+1确认、D+2交易。

## 3.2 发现轨道

### Track S：语义/PIT发现，主轨道

允许的来源：

1. 项目自建并逐日归档的vendor概念快照；
2. 有精确公开时间戳的政策、产业新闻和公司公告；
3. 只使用当时已发布文本构建的公司—题材关联；
4. 至少两个独立来源的一致成员关系，或一个来源加公司公开文本证据。

禁止用今天的概念成员回填历史。

若历史接口没有可恢复的纳入和剔除日期，回测起点只能从项目首次完整归档日开始。

### Track B：行为发现，次轨道

用于发现vendor尚未命名、但已经在D−1以前形成行为关联的候选集合。

输入只能使用截至D−1的数据：

- 最近20日行业残差收益；
- 最近20日异常成交共同出现；
- D−1以前的公开文本语义相似度（按下述 B-LEX 或 B-EMB，PIT 纪律见 DG13）；
- 既有PIT标签重叠。

禁止使用D日涨停或D日收益建立D日候选集合。

#### 文本模式（互斥）

**B-LEX：历史正式主模式。** 对每个公司，将截至 D−1 20:00（INPUT_SNAPSHOT_CUTOFF，§4.3）可用文本归一化后做字符 3–5 gram hashing：

- 输入必须是 ETL 以冻结 `parser_version` 生成并保存哈希的 `text/plain` UTF-8 字节；出现 HTML 标签或解码失败即隔离。依次执行 Unicode 15.0 NFKC、ASCII 字母小写化、Unicode General Category 以 P/C/Z 开头的 code point 替换为单空格、连续空白折叠、首尾去空白；不做简繁转换、分词或停用词删除；
- 在每个连续 CJK/ASCII字母/数字 token 两端加入 `^`、`$`，只在 token 内生成长度3、4、5的 Unicode code-point n-gram；不跨 token；
- document 唯一键为 `(company_id, source_url, published_at, content_hash)`；同公司相同 content_hash 只保留 available_at 最早的一条；
- 固定维度 \(M=2^{18}\)；对 n-gram 的 UTF-8 bytes 使用 MurmurHash3 x86_32、seed=0。令无符号32位结果为 h，index=`h mod M`，sign=高位 bit31 为0时 +1、为1时 −1；
- 文档桶 k 的 signed count 为 \(z_{d,k}=\sum sign(ngram)\)；\(TF_{d,k}=sign(z_{d,k})\log(1+|z_{d,k}|)\)；
- 截至 D−1 的去重文档数为 \(N_D\)，\(df_{k,D}=\sum_d\mathbf1(z_{d,k}\ne0)\)，\(IDF_{k,D}=\log((1+N_D)/(1+df_{k,D}))+1\)；
- 文档向量 \(v_{d,k}=TF_{d,k}IDF_{k,D}\) 后做 L2 归一化；零向量视为无有效文本；
- 公司向量为过去120日文本 TF-IDF 向量按 \(e^{-age/60}\) 衰减加权后 L2 归一化；
- \(SemanticSim\) 为公司向量 cosine similarity；无有效文本时记不可配对，不填0；
- 每个 D 的处理顺序固定为：先纳入 `available_at <= D-1 20:00` 的新增去重文档并更新 \(N_D,df,IDF\)，再以同一 \(IDF_D\) 重算受影响的120日公司向量；
- IDF 的文档频次与公司向量按日增量更新，缓存规则见 §8.3.8–§8.3.9。

B-LEX 不依赖预训练语义模型，允许覆盖具有 PIT 文本快照的完整历史，是 Track B 正式历史检验的默认口径。

**B-EMB：可选 measurement regime。** 使用登记 embedding 时，必须满足：

- `training_cutoff < regime_start`；
- 模型、分词器、pooling、最大长度和权重哈希在整个 regime 内固定；
- 只编码 `available_at <= D-1 20:00` 的文本；
- 模型变更即关闭当前 regime，新建 regime；不同模型 regime 的证据不得无条件池化；
- 禁止把模型训练截止日前的历史文本回填为正式 B-EMB 信号。

同一 regime 只能选择 B-LEX 或一个 B-EMB 版本，选择在看任何结果前冻结。B-EMB 不可用时使用 B-LEX，不得因此删除 Track B 历史。

#### BehaviorSim 唯一定义

对股票 \(i,j\)，使用 D−20:D−1 的共同有效交易日（不少于15日）：

\[
ReturnCorr_{ij}
=
\frac{1+\operatorname{corr}(x_i,x_j)}{2}
\]

\[
TurnCorr_{ij}
=
\frac{1+\operatorname{corr}(TurnZ_i,TurnZ_j)}{2}
\]

\[
ActiveJaccard_{ij}
=
\frac{\sum_t\mathbf1(Active_{i,t}=1\land Active_{j,t}=1)}
{\max(1,\sum_t\mathbf1(Active_{i,t}=1\lor Active_{j,t}=1))}
\]

\[
BehaviorSim_{ij}
=
Median(ReturnCorr_{ij},TurnCorr_{ij},ActiveJaccard_{ij})
\]

任一相关序列方差为0时对应相关分量记0.5；共同有效日不足15日时该股票对不可连接。

#### 确定性聚类

采用complete-linkage层次聚类，不使用简单连通分量。

股票对相似度：

\[
Sim_{ij,D-1}
=
0.60\times SemanticSim_{ij,D-1}
+
0.40\times BehaviorSim_{ij,D-1}
\]

任一股票缺有效文本向量时，该股票对的 \(SemanticSim\) 不可用，禁止以0或行为分数回填；该 pair 不可连接。Track B 因文本覆盖不足形成的股票/episode 缺失率必须报告。

主阈值：

    最小簇规模 = 5
    簇内最小两两相似度 >= 0.60
    簇内平均语义相似度 >= 0.65

complete-linkage用于阻止一只多概念股票将两个无关题材桥接成大连通分量。

**并列 tie-break：** 层次聚类每一步合并距离出现并列时，先合并"簇内最小股票代码"字典序更小的簇对；两簇对的最小代码再并列时，比较次小代码，依此类推。该规则消除跨实现的树构造分歧。

Track B单独报告，不与Track S混合计算主判词。

**主法庭归属（v8 唯一）：**

- **主终局 Gate 1–9 = Track S 唯一样本**；附录 B Population 若未写 Track，默认 Track S；
- Track B = 独立辅助 court，只产生 `TRACK_B_PASS` / `TRACK_B_FAIL` / `TRACK_B_INSUFFICIENT`，**不影响当前主终局**；
- Track B 样本不足输出 `TRACK_B_INSUFFICIENT`，不阻断 Track S；
- 只有 Track B 在自身共同支持域内通过其独立 Gate 1–9，才允许从下一新 SPEC/regime 起与 Track S 合并；不得在当前评估样本中即时合并。

## 3.3 三个成员集合

每个题材episode必须同时保存三个集合。

### Base Universe \(B_{u,D-1}\)

D日前已经定义的候选成员集合，用于D日确认。构造规则：Track S 的 PIT 成员快照，或 Track B 在 D−1 截止数据上的聚类输出。

### Entry Cohort \(E_{u,D}\)

D日首次确认时的成员快照，用于：

- 入场归因；
- 防止事后改写信号；
- 重放历史决策。

### 冻结角色集合 \(B^E,L^E,K^E\)

在首次确认日 \(\tau_{confirm}\) 收盘后一次性定义：

\[
B^E_u=E_{u,\tau_{confirm}},\qquad
L^E_u=L_{u,\tau_{confirm}},\qquad
K^E_u=K_{u,\tau_{confirm}}
\]

三者在 episode 生命周期内永久冻结。若 \(|K^E|<3\)，episode 登记 `NO_ROLE_COHORT`：可保留发现/确认样本，但不得进入 H-ROLE、H-MIG、容量时钟或产生交易。

持仓/信号日的可交易候选：

\[
K^{trade}_{u,t}
=
\{i\in K^E_u:\ i\text{ 当日满足 §9.1 的交易状态、订单容量与 §9.3 约束}\}
\]

不得从 \(B^E\setminus K^E\) 补入新容量角色。正式迁移变量用 \(B^E,L^E,K^E\)；选股和候选级 LOO 用 \(K^{trade}\)，但 \(GCC^{\theta}\) 仍基于完整 \(K^E\) 测量题材状态。

episode 合并时（v10：**账务可合并，measurement 不合并**）：

- 科学研究样本在合并日前一日右删失，两个原 episode 不产生合并后的 H-ROLE/H-MIG 观测；
- **禁止**构造 union \(B^E/L^E/K^E\) 后用单一 null 计算 IES/IDS/GCC/RMI 等 level 变量（无合法 union null，且禁止确认后重抽）；
- 各 origin episode 继续使用自身冻结 \(B^E/L^E/K^E\)、EPISODE_NULL、状态机与风险腿；
- 账务持仓可并入承接 episode 的组合记账，但每笔持仓保留 `origin_episode_id`；退出信号由 origin episode 状态机产生，只作用于该 origin 的持仓（或该 origin 在承接账下的份额）；
- 合并后不生成新的主信号；承接 episode 不得因并入而重算角色。

### Live Active Core \(A_{u,t}\)

持仓期间每日更新的活跃核心。构造规则：

1. **种子：** \(A_{u,\tau_{confirm}} = E_{u,\tau_{confirm}}\)；
2. **每日纳入（对全市场股票 j）：** 同时满足
   - 以当日更新前成员 \(A_{u,t-1}\) 固定回看 t−20:t−1，j 与该成员集合每日等权行业残差收益序列的相关系数 \(\ge 0.50\)；
   - 过去10日内至少1日 \(Active_{j}=1\)（定义见 §6.4）；
   - 非ST、非停牌、非退市整理；
3. **每日移出：** 连续10个交易日不满足纳入条件(a)与(b)的股票；
4. **身份登记：** 每只成员记录 `joined_at` / `left_at` / `join_reason`，LAC 每日快照哈希归档。

确认日的种子规则覆盖每日纳入/移出规则；首次动态更新从 \(\tau_{confirm}+1\) 开始，使用 \(A_{u,\tau_{confirm}}\) 作为 \(A_{u,t-1}\)。

LAC 用于：

- 新成员扩散监控；
- 龙头更替识别；
- 持仓期动态健康与衰退识别。

LAC **不用于** H-ROLE、H-MIG、入场确认、\(LDS^E/CDS^E/RMI^E\) 或容量时钟。正式迁移 estimand 永远基于冻结 Entry Cohort；LAC 只是一套持仓后动态风险传感器。

### LAC 正式动态变量

对持仓 episode 的交易日 \(t\)：

\[
LACBreadth_{u,t}
=
\frac{\sum_{j\in A_{u,t}}\mathbf 1(Active_{j,t})}{\max(1,|A_{u,t}|)}
\]

\[
LACTurnBreadth_{u,t}
=
\frac{\sum_{j\in A_{u,t}}\mathbf 1(TurnZ_{j,t}>0)}{\max(1,|A_{u,t}|)}
\]

\[
LACRelReturn5_{u,t}
=
\sum_{s=t-4}^{t}
\frac{\sum_{j\in A_{u,t}}x_{j,s}}{\max(1,|A_{u,t}|)}
\]

`LACRelReturn5` 使用当日成员集合固定回看，避免早期缺少历史 LAC 快照；要求当日集合非空且5个收益日完整，否则记 `INSUFFICIENT_WINDOW`。

\[
LACAttrition10_{u,t}
=
\min\left(
1,\,
\frac{|\{j:left\_at_j\in[t-9,t]\}|}{\max(1,|A_{u,t-10}|)}
\right)
\]

不足10个 LAC 观测日时 \(LACAttrition10\) 记 `INSUFFICIENT_WINDOW`，该分支不触发。LACBreadth 与 LACTurnBreadth 需要当日集合非空；任一为空或缺失时记 `INSUFFICIENT_WINDOW`。连续衰退分支至少需要2个完整 LAC 观测日；任何输入窗口不足时 `LACDecay=0` 并登记，不得用缺失填0触发退出。

正式动态衰退事件：

\[
LACDecay_{u,t}
=
\mathbf 1\left(
\begin{array}{l}
\text{连续2日 }LACBreadth<0.20
\land LACTurnBreadth<0.30
\land LACRelReturn5<0\\
\quad\lor\\
LACAttrition10>0.40
\land LACRelReturn5<0
\end{array}
\right)
\]

`LACDecay=1` 只进入 §12.8 COLLAPSE 的独立分支，并由 Leg X 执行；不得改写任何冻结迁移变量或历史入场状态。

**变量—集合映射表：**

|变量家族|计算集合|禁止用途|
|---|---|---|
|IES、IDS、CONF、状态机判定、EntryScore|Base Universe / Entry Cohort / 冻结角色集合（题材级或 LOO，见 §3.4）|不得在 LAC 上重算确认|
|\(LDS^E,CDS^E,PriceShare^E,RMI^E,CapacityBreadth^E\)|Entry Cohort 内冻结的 L 组与 K 组|不得用 LAC 新成员改写迁移归因|
|LACBreadth、LACTurnBreadth、LACRelReturn5、LACAttrition10、LACDecay|Live Active Core|只允许进入持仓期动态监控与 COLLAPSE 独立分支；不得生成入场信号|
|新成员 Active 事件|Live Active Core|进入 LAC 动态变量与机制报告，不进入冻结迁移估计|

Entry Cohort永不修改；Live Active Core允许动态变化。二者不得混用。

## 3.4 变量→对象归属表

rank/level 尺度纪律（§9.2）之外，本规范实行归属纪律：每个正式变量声明其 owner，任何规则只能消费与其对象粒度一致的变量；跨归属消费必须在本表显式登记。

|变量|owner|计算口径|合法消费者|
|---|---|---|---|
|\(IES_{u,D}\)|题材|全 L 组，对 null 标准化|状态机、入场资格、Study 1|
|\(IDS^{\theta}_{u,D}\)（题材级）|题材|按 §3.5：确认前临时 \(B\setminus L\)；确认后冻结 \(B^E\setminus L^E\)|状态机、Whipsaw5、Leg B2/X、§12 全部门槛|
|\(IDS^{-i}_{u,D}\)（候选级）|候选股|按 §3.5 映射的 \(B\setminus(L\cup\{i\})\)|该候选入场资格、\(CONF^{-i}\)、P1|
|\(GCC^{\theta}_{u,D}\)（题材级）|题材|全体 K 的中位数|状态机（§12.4–12.8）|
|\(GCC^{-i}_{u,D}\)（候选级）|候选股|\(K^E\setminus\{i\}\)|该候选入场资格（§13.1）|
|\(CCS\_rank_i\)|候选股|K 组内分位|EntryScore 排序、tie-break|
|\(CCS\_level_i\)|候选股|对 null 标准化|个股门槛、GCC 构造、Leg I、Study 4|
|\(RMI^E,CapacityBreadth^E,LDS^E,CDS^E,HHI^E\)|题材|Entry Cohort 的冻结 L/K/B 组|入场状态机、EntryScore（经 Pctl_cross）、迁移研究|
|LACBreadth、LACTurnBreadth、LACRelReturn5、LACAttrition10、LACDecay|题材持仓 episode|Live Active Core 日快照|COLLAPSE 的 LAC 独立分支、Leg X、运行报告|
|FutureAttention、FutureCapacity|候选股研究 outcome|冻结 L/K 的 D+1:D+5 数据，对匹配 null 标准化|仅 H-ROLE / Gate 3；禁止入场与退出|
|\(\theta^{ROLE}_A,\theta^{ROLE}_C\)|题材 episode 研究 estimand|冻结 L/K 未来功能 contrast|仅 Gate 3 三态判词|
|EntryScore|候选股|§13.2|组合分配排序|
|目标权重、现金预留|组合|§15|账本|

状态机（题材级对象）**禁止**消费任何候选级变量；唯一合法伪代码见 §25。

## 3.5 确认前后正式集合与 null 唯一映射（v8）

|阶段|IES / IDS 成员集合|注意力 L|容量 K|null|角色冻结|
|---|---|---|---|---|---|
|确认前（无 episode）|当日 Base Universe 与当日临时 L 导出的 \(N^{\theta}/N^{-i}\)|§7.3 每日临时识别|§9.1 每日临时|DAILY_NULL（每日生成）|不冻结|
|确认日 \(\tau_{confirm}\)|当日用于确认的集合计算出 IES/IDS 后，立即冻结|冻结为 \(L^E\)|冻结为 \(K^E\)；\(B^E=\)确认日 Base Universe|生成并冻结 EPISODE_NULL|一次性写入，此后只读|
|确认后（有 episode）|**只允许**使用冻结 \(B^E,L^E\)：\(N^{\theta}=B^E\setminus L^E\)，\(N^{-i}=B^E\setminus(L^E\cup\{i\})\)|禁止重新识别生产 L|\(K^{trade}\subseteq K^E\) 仅交易过滤|只加载 EPISODE_NULL，禁止重新抽样|禁止混用动态 Base/L|

实现者不得在确认后用“当日 vendor Base Universe + 重新 identify_attention_roles”计算生产 IES/IDS/状态机/退出。研究诊断若报告动态口径，必须标记 `DIAGNOSTIC_DYNAMIC_SET`，不得进入信号或 Gate 主样本。

### 3.5.1 FrozenMetricBackcast（v12；确认量尺永冻）

**确认量尺永冻规则（A-01 Closure）：**

1. \(\tau_{confirm}\) 的三条件（§11.4）**永久且仅**使用确认前已归档的临时量尺：当日临时 Base/L + 已归档 `DAILY_NULL` 下计算并写入 ARCHIVE 的 \(IDS^{\theta}\) 历史；
2. 一旦 \(\tau_{confirm}\) 在该临时量尺下成立，episode 创建事实**不得**因后续 FrozenMetricBackcast 被撤销、改写或重新判定；
3. FrozenMetricBackcast **禁止**回写、覆盖或替代“确认斜率所用 IDS 历史”，也**禁止**用冻结 cohort 重算确认谓词。

在 \(\tau_{confirm}\) 日完成 \(B^E/L^E/K^E\) 与 EPISODE_NULL 冻结后，对**确认后**需要历史差分/均值的冻结变量，用该冻结 cohort 与 EPISODE_NULL 对所需历史窗口做 PIT 回算：

- **允许覆盖：** \(GCC^{\theta/-i}\)、\(GCCSlope\)、\(RMI^E\)、\(CapacityBreadth^E\)、\(HHI^E\)、\(\Delta_3 EMA_3(\cdot)\) 及确认后状态机所需的其他非确认谓词输入；
- **禁止覆盖：** 确认日及以前用于判定 \(\tau_{confirm}\) 的 \(IES/IDS^{\theta}\) 历史序列与确认谓词结果；
- 回算窗口：至少覆盖该变量定义所需的最长历史（通常 D−3:D−1 或 EMA 初始化窗）；
- **禁止**将确认前临时 B/L/K + DAILY_NULL 的序列与确认后冻结量尺拼接用于确认后变量；
- 回算只使用确认日已经可知的历史市场事实，**不改写**确认日前已归档的状态史/信号史；回算结果仅供确认日及之后的状态机、入场与研究路径使用；
- 回算完成前不得判定 EARLY_CAPACITY / 主信号。


---

# 4. 数据与PIT闸门

## 4.1 必需数据

|域|字段/要求|
|---|---|
|日线|open、high、low、close、pre_close、vol、amount|
|分钟线|1分钟OHLCV，至少覆盖09:25—15:00|
|复权|逐日复权因子（后复权链，历史值不随未来公司行动重算；见 DG07 补充）|
|流通与估值|自由流通市值、流通股本、换手率（按日快照，修订不回填；见 DG14）|
|交易状态|上市、退市、ST、停复牌、所属交易板块|
|涨跌停|股票—日期级up_limit、down_limit|
|题材PIT|主题ID、发现来源、成员、生效和失效时间、原文或快照哈希|
|行业PIT|残差收益所用行业映射的按日快照（见 DG14）|
|市场基准|全A宽基及风格指数；主基准自建规则见 DG15|
|费用规则|按生效日期版本化|
|文本源|发布时间、抓取时间、文本哈希、解析模型版本|
|机制诊断（仅研究）|龙虎榜席位快照，published_at/available_at 完整|
|公司行动PIT|分红、送转、拆合股、配股、代码变更、退市价值；见 §15.5 / DG16|

**成交量与金额单位（v8 冻结，Canonical 正文唯一）：**

- `vol`（日线与分钟线）= **股**（shares），不是手；若供应商以手提供，接入层必须乘 100 转为股后再进入任何正式计算；
- `amount` = 人民币元（CNY）；
- 分钟 VWAP = `amount / vol`（vol>0）；vol=0 的分钟无 VWAP、可参与量为 0；
- 参与量 `floor(vol_minute × 5%)` 后再向下取整到 100 股，输入 vol 必须已是股；
- 禁止在 `data_dictionary.yaml` 中另定义与本条冲突的单位。

**分钟 bar 时间语义（v8 冻结）：**

- 标签为时刻 \(t\) 的 1 分钟 bar 覆盖半开闭区间 \([t:00,\ (t+1):00)\)，即 \([t:00,\ t:59.999\ldots]\)；
- 例：`09:30` bar = \([09:30:00,\ 09:31:00)\)；`09:34` bar = \([09:34:00,\ 09:35:00)\)；
- 09:35 决策使用且仅使用标签 09:30–09:34 的五根已完成 bar；
- 集合竞价成交归入 `09:25`（或供应商开盘拍卖 bar），不得并入 `09:30` 连续竞价 bar；复牌首分钟成交归入该分钟标签 bar。

## 4.2 核心信号禁止依赖的数据

第一版核心入场不使用：

- 龙虎榜；
- 融资净买入；
- 第三方主力资金流；
- 机构专用席位；
- 订单金额推断的资金身份。

这些数据只能作为机制诊断（§17.5）。

原因：

- 龙虎榜是异常交易触发披露，不是全量订单流；
- 融资标的资格与市值、流动性高度相关；
- 订单金额法无法可靠区分散户与机构；
- 公布时间可能与信号封存时间冲突。

## 4.3 可用时间

所有记录必须有：

    event_time
    published_at
    ingested_at
    available_at
    source_version
    content_hash

**三时钟分离（时区均为 Asia/Shanghai）：**

    INPUT_SNAPSHOT_CUTOFF = D 20:00
    COMPUTE_DEADLINE      = D 21:00
    ARCHIVE_DEADLINE      = D 21:30

- D 日信号链只能读取 \(available\_at \le\) `INPUT_SNAPSHOT_CUTOFF` 的记录；20:00 后到达的记录 `available_at` 一律记为下一交易日，不得进入 D 日任何计算；
- 输入快照在 20:00 冻结并哈希；计算必须在 `COMPUTE_DEADLINE` 前完成（超时处置见 §8.3.9）；信号在 `ARCHIVE_DEADLINE` 前只增不改归档；
- 三时钟顺序为不变量：快照冻结后的任何数据到达（无论多合法）都不得触发当日重算。两个实现对同一到达序列必须产生相同输入集。

核心信号不依赖D日两融数据。龙虎榜 D 日 20:00 后发布的数据，available_at 记为下一交易日，只允许进入 §17.5 的事后机制研究，与 D+1 决策在工程上物理隔离（独立库表、独立任务、信号任务无读取权限）。

## 4.4 数据闸门

必须全部通过：

- DG01：历史任意日期可重建当时题材Base Universe；
- DG02：不存在当前成员回填历史；
- DG03：题材发现源的发布时间不晚于发现截止时间；
- DG04：涨跌停价格按股票、日期和交易板块正确恢复；
- DG05：分钟线与日线OHLCV聚合一致；
- DG06：退市股票、停牌股票和ST状态完整；
- DG07：复权研究价格与真实成交价格不混用；**复权方向声明：** 研究收益使用后复权链，历史复权价不得随未来公司行动重算；前复权价禁止进入任何信号；
- DG08：任何信号字段均有available_at；
- DG09：未来列置空后，信号哈希不变；
- DG10：生产 ETL 输出必须由 §26.3 独立重发审计从只读原始快照完整重建，且有效宇宙与快照哈希一致；
- DG11：vendor成员无法PIT恢复时，历史样本自动截断至归档起点；
- DG12：文本模式、hashing配置、IDF状态或embedding版本完全冻结；
- **DG13（Track B 文本 PIT 闸门）：**
  1. 语义相似度计算只可使用 \(published\_at \le D-1\) 且 \(available\_at \le D-1\ 20{:}00\) 的文本快照；
  2. B-LEX 的 hashing 维度、算法、seed、字符 n-gram 范围、IDF 截止日与状态哈希必须归档；未来文本置空后历史向量哈希不变；
  3. B-EMB 的 training cutoff 必须登记，且严格早于该 measurement regime 起点；不要求早于 B-LEX 历史样本起点；
  4. 每条文本须有可验证的公开来源与时间戳；无法验证 published_at 的语料不得进入 \(SemanticSim\)；
  5. B-EMB 模型权重/分词器/配置哈希变更即新 regime；禁止跨 regime 无条件池化；
  6. Track B 可评估 episode 少于120或不足3年时输出 `TRACK_B_INSUFFICIENT`，不以放宽 PIT 纪律换取样本。
- **DG14（行业与股本 PIT）：** 行业分类映射与自由流通股本/市值均按日（至少按公告生效日）快照保存；分类改版（如申万 2014/2021）与股本修订不得回填历史；行业残差 \(x_{i,t}\) 与 TurnValue 只可使用 t 日当时快照。历史快照不可恢复的区间，按 DG11 截断。
- **DG15（基准事实源）：** 组合层基准"全A等权指数"为自建：成分 = 当日满足 §5.1 前四条资格（A股普通股票、上市满120交易日、非ST/退市整理、当日正常交易）的全部股票，等权、每日再平衡、含退市股票至其最后交易日；构造代码与逐日成分哈希入冻结物。次要对照（沪深300、中证1000）须使用 PIT 成分。
- **DG16（公司行动 PIT）：** §15.5 所需字段齐全；`available_at` 合法；抽样持仓区间过账后 NAV 恒等式与股数勾稽通过；迟到行动不得回写历史已归档账本日。

关键项失败：

    VERDICT = INVALID_DATA

不得用收益回测替代数据合法性。

---

# 5. 股票与题材样本

## 5.1 股票基础样本

D日股票必须满足：

- A股普通股票；
- 上市满120个交易日；
- 非ST、非退市整理；
- D日正常交易；
- 过去20日有效交易日不少于15日；
- 过去20日平均成交额足以支持策略订单；
- 股票所属交易板块和当日涨跌停制度可确定。

**信号侧容量资格（只使用 D 日可知信息，v7 修复 B-01）：**

\[
SignalCapacityFloor_i:\quad
0.5\%\times ADV20_{i,D}
\ge
MinPositionValue
\]

其中 \(MinPositionValue = 2\%\times\) 研究资本（标准化1000万元下为20万元）。不满足者不具容量资格。订单上限：

\[
OrderValue_i \le 0.5\%\times ADV20_{i,D}
\]

**执行侧参与约束（D+1 实时，见 §14.1）：** 每分钟成交量的 5% 参与率在执行过程中实时限制成交数量。**D+1 成交量只能影响实际成交结果，禁止反向进入 D 日任何资格、角色、状态或信号判定**（v6 将 D+1 早盘成交额写入 D 日容量约束，构成未来信息，已废止）。

研究阶段按标准化1000万元组合计算，并同时报告容量曲线。

## 5.2 题材基础样本

题材Base Universe必须满足：

- 有效成员数不少于8；
- D日正常交易成员比例不少于70%；
- 剔除注意力核心后剩余成员不少于5；
- 有效成员数：

\[
N_{eff}
=
\frac{1}{\sum_j w_j^2}
\ge 5
\]

其中 \(w_j\) 为过去20日成交额份额。

## 5.3 高重叠题材簇与代表题材（v8 唯一算法）

同日题材成员 Jaccard：

\[
J(A,B)=\frac{|A\cap B|}{|A\cup B|}
\]

**簇构造时点：** 每个交易日 D，在完成当日全部题材的临时 IES / \(IDS^{\theta}\) 计算之后、确认冻结与入场之前执行。输入集合 = 当日处于进行中 episode 的题材 ∪ 当日首次满足或评估确认的题材候选（含 DISCOVERED/IGNITION/CONFIRMED_* 评估对象）。

**簇构造算法（连通分量，唯一；与 §3.2 Track B 股票行为聚类的 complete-linkage 不是同一对象）：**

1. 构造无向图：节点 = 上述题材；边 = \(J(\text{BaseUniverse}_A,\text{BaseUniverse}_B)\ge 0.60\)（进行中 episode 用其冻结 \(B^E\)，未确认用当日 Base Universe）；
2. 题材簇 = 该图的**连通分量**（Jaccard 非传递；禁止完全连接、禁止贪心两两合并的其他实现）；
3. 簇 ID = 分量内全部 `theme_id` 字典序最小者；同一连通分量每日重算，但已冻结 episode 的历史簇归属只增不改归档。

**代表题材确认质量（唯一公式）：**

\[
ConfirmQuality_{u,D}
=
\min(IES_{u,D}, IDS^{\theta}_{u,D})
\]

同簇入场与新确认只保留 ConfirmQuality 最高的代表题材；并列 tie-break 固定序：

1. 更高 \(IES\)；
2. 更高 \(IDS^{\theta}\)；
3. 更早 \(\tau_{confirm}\)（已确认者优先；均未确认则跳过本条）；
4. `theme_id` 字典序更小。

**消费规则（v9）：**

- **全部进行中 episode（含非代表）：** 每日必须更新冻结度量（IES/IDS/GCC/RMI）、LAC、状态机与退出腿；
- **仅簇代表题材：** 可新确认、可开主信号窗；非代表当日不得新确认、不得新开入场；
- 相同股票不重复持仓（§15）；
- 统计推断聚类单元 = §20.2.1 persistent overlap component × episode；
- 冷却期同簇封锁（§12.9）使用同一连通分量定义。

## 5.4 episode 合并与分裂

### 合并

两个进行中 episode（\(u_1\) 的 \(\tau_{confirm}\) 更早）满足：

\[
J(A_{u_1,t}, A_{u_2,t}) \ge 0.60
\quad\text{连续2个交易日}
\]

则 \(u_2\) 在账务上合并入 \(u_1\)（v10 measurement 不合并；v11 风险归属不迁移）：

- 保留 \(u_1\) 与 \(u_2\) 各自的 episode_id、冻结角色、EPISODE_NULL 与状态机；\(u_2\) 登记 `MERGED_INTO = u_1`（**仅**执行汇总与报告指针）；
- Entry Cohort / \(B^E/L^E/K^E\) **各自保持冻结存档，不得取并集后重算 level 变量**；
- **ThemeRoom / 题材簇风险归属永久按 origin 入场时簇计入，不得因 MERGED_INTO 迁移到承接 episode；** `MERGED_INTO` 不得改变 §15.0 ThemeRoom 分子；
- 退出仍由各 origin 状态机触发，只卖出该 origin 持仓；
- 高优先级退出（Leg X 等）若两 origin 同日触发，按腿优先级合并执行，数量为各 origin 应卖之和；
- H-ROLE/H-MIG 等对两 origin 均在合并日前一日右删失；合并后运行片段不作为新的科学 episode；
- 每笔持仓保留 `origin_episode_id`；组合净值只计一次。合并片段只进组合政策/风险/执行报告。

### 分裂

单 episode 内出现两个子簇，满足：

    簇内平均两两相关 >= 0.60
    簇间平均相关 < 0.30
    持续5个交易日

则登记 `SPLIT` 事件：

- 账务不拆分，仍属原 episode；
- 归因报告中两个子簇分别标记，供机制研究使用；
- 状态机与退出腿仍以原 episode 整体判定。

### 同股多 episode

同一股票同时属于多个进行中 episode 时只持有一份（§15 已有），其退出跟随首个持有它的 episode。

---

# 6. 基础变量与统一算子

## 6.1 收益残差

股票收益：

\[
r_{i,t}
=
\frac{Close_{i,t}}{Close_{i,t-1}}-1
\]

行业残差：

\[
x_{i,t}
=
r_{i,t}
-
r_{industry(i,t),t}
\]

**行业收益唯一事实源（v7 冻结，A-01）：**

\[
r_{industry(i,t),t}
=
\frac{1}{|G_{i,t}|}
\sum_{j\in G_{i,t}} r_{j,t}
\]

其中 \(G_{i,t}\) 为 t 日按 DG14 PIT 映射属于股票 i 的申万一级行业、且满足 §5.1 前四条资格并当日有成交的全部股票（**含 i 自身**，规则唯一优先于剔除微小自影响）。不使用官方行业指数。\(|G_{i,t}|<5\) 时 \(x_{i,t}\) 记缺失：以其为输入的 Active、CumX、DirectedDemand、Rel 等条件一律按不满足/零贡献处理并登记。行业映射按 DG14 的当日 PIT 快照。不使用包含候选股自身的题材指数计算"纯度"。

## 6.2 异常成交

\[
TurnValue_{i,t}
=
\log\left(
\frac{Amount_{i,t}}{FreeFloatMV_{i,t}}
\right)
\]

\[
TurnZ_{i,t}
=
\frac{
TurnValue_{i,t}
-
Median(TurnValue_{i,t-60:t-1})
}{
1.4826\times MAD(TurnValue_{i,t-60:t-1})
}
\]

使用稳健Z分数，D日不进入自身历史基线。FreeFloatMV 按 DG14 的当日 PIT 快照。

## 6.3 收盘承接

\[
CLV_{i,t}
=
\begin{cases}
\frac{Close_{i,t}-Low_{i,t}}{High_{i,t}-Low_{i,t}}, & High>Low\\
0.5, & High=Low
\end{cases}
\]

3日均值：

\[
CLV3_{i,D}=Mean(CLV_{i,D-2:D})
\]

## 6.4 当日强势状态

股票满足任一条件即为Active：

    A. D日收盘涨停；
    B. 行业残差收益位于相同交易板块当日85%分位以上
       AND TurnZ >= 0.5
       AND CLV >= 0.70。

按交易板块分层，避免10%、20%、30%制度混在同一绝对标准内。

## 6.5 新参与者

\[
NewJoin_{i,D}
=
Active_{i,D}
\land
\sum_{k=1}^{3}Active_{i,D-k}=0
\]

"新参与者"比"连续同一批股票涨停"更接近题材扩散。

## 6.6 变化量算子

\[
\Delta_3\,EMA3(X)_D
=
EMA3(X)_D - EMA3(X)_{D-3}
\]

其中 \(EMA3\) 为 span=3 的指数移动平均。凡文中出现 \(\Delta\) 而未带下标者，均指 \(\Delta_3\,EMA3\)。

## 6.7 分位算子 Pctl（全文唯一定义）

对标量 \(x\) 与参照多重集 \(S\)（\(|S|=N\)）：

\[
Pctl(x;S)
=
\frac{
\#\{s\in S: s<x\}
+
0.5\times\#\{s\in S: s=x\}
}{N}
\]

即 midrank 经验分位，不插值。本定义**无例外地**适用于全文全部分位：

- \(Pctl_K\)（容量组内分位，§9.2.2）；
- \(Pctl_{null}\)（匹配伪题材 null 分位，§8.4、§8.5、§9.2.3）；
- \(Pctl_{cross}\)（同日跨题材分位，§13.2）；
- 个股历史分位（250日、§9.3）与板块内当日分位（§6.4）；
- REJECT_GAP 的经验分位 Q0.85（§14.1：\(Q_{0.85}\) 取满足 \(Pctl(g;S)\ge 0.85\) 的最小样本值）。

**离散变量并列处理：** Persist∈{0,1,2,3} 等离散变量直接按上式 midrank 处理并列，**不做任何缩放**。单调正变换不改变经验分位，因此不得以缩放替代并列规则。

被评对象 \(x\) 自身不进入参照集 \(S\)（自排除），除非该处显式声明池化定义。

## 6.8 EMA 初始化与最短观测

- 递归式：\(EMA_t = \alpha X_t + (1-\alpha)EMA_{t-1}\)，\(\alpha = 2/(span+1)\)（span=3 时 \(\alpha=0.5\)）；
- 初始化：\(EMA_{t_0} = X_{t_0}\)，\(t_0\) 为该变量序列在本 episode 观察窗内的首个可用日；
- 最短观测：\(\Delta_3 EMA3\) 需要不少于 4 个可用观测；不足时该变化量记为**不可判定**，凡以其为条件的判定一律按"条件不满足"处理（保守方向），并登记 `INSUFFICIENT_WINDOW`。

## 6.9 通用 Hash 与随机流协议（全文唯一；v7 引入，v8 维持）

全文任何 \(Hash(f_1,f_2,\dots,f_n)\) 形式的种子或分片函数均按以下唯一协议实现：

    payload = UTF8( str(f_1) + "\x1f" + str(f_2) + "\x1f" + ... + str(f_n) )
    其中：日期序列化为 YYYYMMDD；整数为十进制无前导零；
          字符串原样；spec_version 为完整版本串（如 "SFL-1_v12.0"）
    digest = SHA-256(payload)                      # 32 字节
    seed   = unsigned_big_endian(digest[0:16])     # 128 位无符号整数
    rng    = PCG64(seed)                           # NumPy PCG64 位流

派生规则：

- 分片/fold 类映射（如 P10 fold）：\(fold = unsigned\_big\_endian(digest[16:24]) \bmod n_{fold}\)，与 seed 使用不重叠字节；
- 同一 payload 的 rng 只允许消费一次完整随机流；重放必须从 seed 重建；
- 文件与数据快照哈希继续使用 SHA-256 全摘要十六进制；
- 该协议本身为冻结物；变更即新 SPEC 版本。

---

# 7. 注意力角色识别

## 7.1 点火时序

过去10日首次Active日期：

\[
FirstActive_{i,D}
=
\min\{t\in[D-9,D]:Active_{i,t}=1\}
\]

越早，点火得分越高。

## 7.2 注意力角色分数 ALS

六个原始特征唯一定义如下（v7 修复 A-01；观察集合为 \(B_{u,D-1}\) 的 §5.1 合格成员，Pctl 按 §6.7 取题材内分位）：

\[
EarlyIgnition_{i,D}
=
\begin{cases}
D - FirstActive_{i,D}, & \exists\,FirstActive\\
-1, & \text{10日内无 Active}
\end{cases}
\]

（单位交易日，越大越早点火；−1 使无点火者恒居最低分位。）

\[
BoardHeight_{i,D}
=
\max\{k\ge0:\ \text{D−k+1..D 每日收盘价}=up\_limit\}
\]

（截至 D 收盘的连续收盘涨停天数；D 日未收盘涨停记 0；停牌日中断连续计数。）

\[
CumX5_{i,D}=\sum_{t=D-4}^{D}x_{i,t}
\]

（\(x\) 缺失日按 0 计并登记；5 日全缺失该股票当日不参与 ALS，题材内分位在其余成员上计算。）

\[
AmountShare3_{i,D}
=
\frac{\sum_{t=D-2}^{D}Amount_{i,t}}
{\sum_{j\in B}\sum_{t=D-2}^{D}Amount_{j,t}}
\]

（停牌日 Amount=0；分母为 0 时全体记题材内分位 0.5。）

\[
PositiveReturnShare3_{i,D}
=
\frac{\sum_{t=D-2}^{D}\max(x_{i,t},0)}
{\sum_{j\in B}\sum_{t=D-2}^{D}\max(x_{j,t},0)}
\]

（分母为 0 时全体记 0.5。）

\[
Defense^{ALS}_{i,D}
=
Median_{t\in T^{def}_B}(x_{i,t}),
\quad
T^{def}_B=\{t\in[D-9,D]:\ \bar r_{B,t}<0\}
\]

（\(\bar r_{B,t}\) 为 \(B_{u,D-1}\) 合格成员等权收益；\(T^{def}_B=\varnothing\) 时该分量记题材内分位 0.5。与 §9.2.1 容量轨 Defense 的区别：ALS 版用全 B 等权判定分歧日、以 \(x\) 计值，容量轨用剔除自身的 \(r^{-i}\) 与 \(Rel\)。）

\[
ALS_{i,D}
=
Mean(
Pctl(EarlyIgnition),
Pctl(BoardHeight),
Pctl(CumX5),
Pctl(AmountShare3),
Pctl(PositiveReturnShare3),
Pctl(Defense^{ALS})
)
\]

不优化权重。**层级声明：** ALS 是题材内横截面分位合成，属 rank 类变量，唯一合法用途是在同一题材同一日内识别注意力角色与计算 IES 的输入分量；禁止跨题材比较 ALS 的绝对水平，禁止用 ALS 的绝对值构造门槛。

## 7.3 注意力核心集合 \(L_{u,D}\)

- ALS最高者必选；
- 第二名若ALS不低于第一名90%，一并纳入；
- 最多两只；
- 容量标的不得属于该集合；
- 注意力核心只做传感器，不假定可买。

---

# 8. 发现后的独立确认

## 8.1 两级观察集合

成员集合按 §3.5 阶段映射取值；下列符号中的 \(B,L\) 在确认前为当日临时集合，确认后为冻结 \(B^E,L^E\)，禁止混用。

### 题材级集合

\[
N^{\theta}_{u,D}
=
B_{u,D}\setminus L_{u,D}
\]

其中确认前 \(B_{u,D}=B_{u,D-1}\)（Base Universe 截止 D−1），\(L_{u,D}\) 为当日临时注意力核心；确认后 \(B_{u,D}:=B^E\)，\(L_{u,D}:=L^E\)。

用于题材级变量（\(IDS^{\theta}\)），供状态机、Whipsaw5 与退出腿消费。

### 候选级 Leave-One-Out 集合

对于候选容量股票 \(i\)：

\[
N^{-i}_{u,D}
=
B_{u,D}
\setminus
(L_{u,D}\cup\{i\})
\]

该候选的入场确认只能使用该集合，防止候选股和龙头自我认证。确认后候选级集合同样只使用冻结 \(B^E,L^E\)。

## 8.2 独立扩散变量

以下变量对给定观察集合 \(N\)（\(N^{\theta}\) 或 \(N^{-i}\)）定义：

### 活跃广度

\[
Breadth_{u,D}(N)
=
\frac{
\sum_{j\in N}Active_{j,D}
}{
|N|
}
\]

### 新参与者率

\[
NewJoinRate_{u,D}(N)
=
\frac{
\sum_{j\in N}NewJoin_{j,D}
}{
|N|
}
\]

### 正异常成交广度

\[
TurnBreadth_{u,D}(N)
=
\frac{
\sum_{j\in N}\mathbf 1(TurnZ_{j,D}>0.5)
}{
|N|
}
\]

### 同向性

\[
SignConcord_{u,D}(N)
=
\left|
\frac{
\sum_{j\in N}\operatorname{sign}(x_{j,D})
}{
|N|
}
\right|
\]

只在 \(N\) 上的等权残差收益为正时使用，否则置零。

### 炸板失败率

\[
FailureRate_{u,D}(N)
=
\frac{
BrokenLimitCount(N)
}{
\max(1,TouchLimitCount(N))
}
\]

**计数唯一定义（v7 补，A-01，日线域）：**

\[
TouchLimitCount(N)=\#\{j\in N:\ High_{j,D}\ge up\_limit_{j,D}\}
\]

\[
BrokenLimitCount(N)=\#\{j\in N:\ High_{j,D}\ge up\_limit_{j,D}\ \land\ Close_{j,D}<up\_limit_{j,D}\}
\]

停牌股不计入分子分母；比较使用交易所申报价精度（0.01元），不做浮点近似。

## 8.3 匹配伪题材生成器

### 8.3.1 排除集与候选池

对真实题材 \(u\)、日期 \(D\)，令 \(CC(u,D)\) 为 §5.3 Jaccard 图中含 \(u\) 的**连通分量**（非仅直接邻居）。

\[
X_{u,D}
=
B_{u,D-1}
\cup
\bigcup_{v\in CC(u,D)} B_{v,D-1}
\cup
\bigcup_{\text{进行中 episode } e} E_{e}
\cup
\text{当日组合持仓股}
\]

其中进行中 episode 的 Base 取冻结 \(B^E\)（若有），否则取当日 Base Universe。

候选池 \(P_{u,D}\) = §5.1 合格股票 \(\setminus X_{u,D}\)。伪题材成员不得与真实题材、同连通分量题材、任何进行中 episode 或持仓共享。

**黄金样例（v9）：** 若 \(J(A,B)=0.65\)、\(J(B,C)=0.65\)、\(J(A,C)=0.40\)，则 \(CC(A)=\{A,B,C\}\)。为 A 构造 null 时，B 与 C 的成员**都必须**进入 \(X_{A,D}\)。仅排除直接邻居（只排除 B）为非法实现。

### 8.3.2 匹配维度（冻结清单）

- 成员数量；
- 申万一级行业构成；
- 交易板块构成；
- 自由流通市值分布；
- ADV20分布；
- 前20日收益和波动率；
- 前20日换手率水平分布（TurnValue 中位数）；
- 前20日涨停频次分布与 Active 率（注意力/拥挤度基线）。

理由：若不匹配拥挤度与注意力基线，确认分位可能机械地反映"该题材本来就比对照更热"，而非"今日发生了独立扩散事件"。匹配维度清单属冻结物，变更即新版本。

### 8.3.3 生成算法（确定性）与两阶段生命周期（v7 修复 A-04；v8 维持并补 load-or-build）

**null 双阶段语义：**

1. **确认前逐日 null（DAILY_NULL）：** 题材尚无 episode 时，每个题材日按本节算法生成，seed 含当日 D，只用于**当日** IES、\(IDS^{\theta/-i}\) 与确认判定；伪题材的注意力集合 \(L_c\) 按 §7.3 在其成员上同构临时构造；**不得**单独用未冻结的 DAILY_NULL 跨日追踪未来路径；
1b. **发现研究 null（DISCOVERY_RESEARCH_NULL，v9）：** 当题材日 D 进入 Gate 1 / H-DISC 样本时，于 D 日按本节算法生成一次并冻结成员矩阵与各伪题材临时 \(L_c\)（seed 含 `("DISC_NULL",theme_id,D,c)`）；D+1:D+5 的 NewJoin/IDS AUC **只使用该冻结 membership 与冻结 \(L_c\)**，观察集合分母固定为冻结 \(|B_c\setminus L_c|\)；窗内合并按 §17 通用规则整条剔除；不得每日重抽 DAILY_NULL 替代本对象；
1c. **发现研究真实 cohort（DISCOVERY_RESEARCH_COHORT_D，v12 唯一）：** 当题材日 D 进入 Gate 1 / H-DISC 样本时，于 D 日同步冻结真实路径测量制度：`B^{DR}_D` = 当日 Base Universe 快照；`L^{DR}_D` = 当日临时注意力核心；`N^{DR}_D=B^{DR}_D\setminus L^{DR}_D`；观察集合分母固定为 \(|N^{DR}_D|\)；Gate 1 的 D+1:D+5 真实 NewJoin/IDS AUC **只使用该冻结 cohort**，禁止窗内切换到确认后 \(B^E/L^E\)、禁止消费窗内 vendor 新增成员；与 1b 的 null 路径使用同一冻结制度语义。正文、§17.1、附录 B、§25 伪代码必须回引本对象，不得另建平行“动态真实路径”。
2. **episode 冻结 null（EPISODE_NULL）：** \(\tau_{confirm}\) 日按本节算法生成一次（seed 含 \(\tau_{confirm}\)），成员矩阵冻结，episode 生命周期内每日复用；**伪确认日 := 配对真实题材的 \(\tau_{confirm}\)**——每个伪题材 c 在该日按 §7.3/§9.1 同构规则一次性冻结 \(L^E_c/K^E_c\)（不要求伪题材自身满足确认条件；其角色是"若这组随机匹配股票是题材，其角色会是谁"的反事实）。\(|K^E_c|<3\) 的伪题材从 CCS/GCC/收益对照 null 池剔除（有效数量按 §8.3.4 阶梯复核）；
3. CCS_level、GCC、H-ROLE/H-MIG null、NetAlpha20 episode 基准一律使用 EPISODE_NULL；确认后不得再为该 episode 重新抽样；
4. **load-or-build 协议（v8，唯一）：** 伪代码与生产入口函数必须实现为 `load_or_build_nulls_v12(theme, D, ...)`：
   - 无 episode / 未确认：build DAILY_NULL(D)，不持久化为 EPISODE_NULL；
   - D = \(\tau_{confirm}\) 且尚无冻结矩阵：build 一次 EPISODE_NULL(\(\tau_{confirm}\)) 并持久化；
   - D > \(\tau_{confirm}\) 且 episode 仍开放：只加载冻结成员矩阵，仅用当日行情更新 PseudoThemeStats；**禁止**按 D 重新抽取 membership；
   - 函数名含日期参数不得被解释为“每日重建 membership”；
5. P10 中假真实题材（control）的角色锚点 = 其在完整链中的**假确认日**（链内首次满足 \(\tau_{confirm}\) 三条件之日），同构冻结其伪角色。

对每个 \(c\in\{1,\dots,200\}\)：

1. 以 \(Seed=Hash(spec\_version,theme\_id,D_{anchor},c)\)（协议按 §6.9；\(D_{anchor}\) 为 DAILY_NULL 的当日 D 或 EPISODE_NULL 的 \(\tau_{confirm}\)）初始化 PCG64 随机流；
2. 真实成员按（申万一级行业 × 交易板块）联合分层；
3. 层内按股票代码升序逐成员匹配：对真实成员 \(m\)，在 \(P_{u,D}\) 中同层、且未被本伪题材已用的股票中，按 7 维马氏距离（log自由流通市值、log ADV20、前20日累计收益、前20日日收益波动率、前20日TurnValue中位数、前20日涨停次数、前20日Active日数；协方差为当日全体合格股票的样本协方差，对角加 \(10^{-6}\) 正则）取最近 20 只，随机流等概率抽 1；
4. 同层候选不足 20 只时取全部；同层候选为空时放宽为同交易板块任意行业中最近 20 只（该次匹配登记 `RELAXED_MATCH`，比例入报告）；仍为空 → 该伪题材构造失败；
5. 同一伪题材内成员不放回；不同伪题材之间允许成员重复（跨篮子依赖在 §20.2 声明并由题材簇聚类吸收）。

### 8.3.4 可行性阶梯与失败处置

设成功构造的伪题材数为 \(n_c\)：

- \(n_c = 200\)：正常；
- \(100 \le n_c < 200\)：继续，登记 `PARTIAL_NULL(n_c)`；
- \(n_c < 100\)：该题材日记 `NON_EVALUABLE(NULL_INFEASIBLE)`：不产生任何 level 变量、不产生新信号；正常状态转移按 §12.0 冻结，但 LACDecay/执行/安全 Kill 分支不冻结（§12.0）。

### 8.3.5 平衡性闸门

伪题材池按**成员出现次数**加权：成功的每个 control 权重 \(1/n_c\)，其内部每个成员权重 \(1/|B|\)；同一 donor 在不同 controls 重复出现时保留每次出现，不去重。真实成员各权重 \(1/|B|\)。

连续 balance features 固定为：

    log(FreeFloatMV)
    log(ADV20)
    CumReturn20
    Volatility20
    MedianTurnValue20
    LimitUpCount20
    ActiveDays20

对每个连续特征 k：

\[
SMD_k
=
\frac{\mu^R_k-\mu^C_k}
{\sqrt{((s^R_k)^2+(s^C_k)^2)/2}}
\]

每组权重先归一化为和1。唯一方差口径为 population weighted variance：

\[
\mu_g=\sum_i w_{g,i}x_{g,i},\qquad
s_g^2=\sum_i w_{g,i}(x_{g,i}-\mu_g)^2
\]

不使用无偏或有效样本量修正。分母为0且均值相同则 SMD=0；分母为0且均值不同则 \(|SMD|=\infty\)。

类别 balance features 为每个实际出现的“申万一级行业”与“交易板块”独立 dummy。对类别 a：

\[
SMD_a
=
\frac{p^R_a-p^C_a}
{\sqrt{(p^R_a(1-p^R_a)+p^C_a(1-p^C_a))/2}}
\]

零分母规则同连续特征。成员数量必须与真实题材完全相等，不使用 SMD。对连续特征另报告 q10/q50/q90 差除以真实组 IQR 的诊断，但不进入主闸门。

任一连续或类别 \(|SMD|>0.25\)，或成员数不等 → 该题材日记 `NON_EVALUABLE(BALANCE_FAIL)`：不产生任何 level 变量、不产生新信号；正常状态转移按 §12.0 冻结（LACDecay 等风险分支不冻结），并登记首个失败维度。全部 SMD 与诊断分位入 Gate 报告。

### 8.3.6 生成器验收套件 NG01–NG07（DESIGN 区执行，任一失败 → 生成器 FAIL，修复后重跑）

- **NG01 排除零违例：** 全样本抽查，伪题材成员与排除集交集恒为空；
- **NG02 平衡审计：** SMD 分布全量报告，超限率与 RELAXED_MATCH 率披露；
- **NG03 null 校准：** 将伪题材自身代入 IES/IDS 计算（参照集为同题材日其余伪题材），其 null 分位应近似均匀分布（分层抽样 ≥500 个题材日，KS 检验 p>0.01）——这是"名义 0.80 分位就是真实 0.80 分位"的直接检验；
- **NG04 牛熊分段：** 按 §15.4 市场状态变量分段重复 NG03，两段均须通过；
- **NG05 种子重放：** 同种子重放输出哈希一致；
- **NG06 优化等价：** 在按板块、市场状态、题材规模分层抽取的至少100个题材日上，§8.3.8 优化 DAG 与无缓存 naive reference 逐对象比较：集合、排序、状态、信号完全一致；浮点满足 §26.3 容差；
- **NG07 P10 近似精度：** 在至少100个题材日上，以独立生成第二层200个 controls 的 brute-force P10 为基准，§19 P10 cross-fit 的年化假信号率绝对差 ≤1个百分点，全部 level 分位 Spearman \(\rho\ge0.99\)。未通过则增加 fold/reference 数或停用 cross-fit，不得放宽阈值。

### 8.3.7 用途

每个扩散变量转换为相对于伪题材的经验分位（Pctl 按 §6.7），防止题材规模和板块制度机械抬高信号。

### 8.3.8 冻结计算 DAG、缓存与精确 LOO

统计定义仍以 §8.3.1–§8.6 为准；本节规定唯一允许的等价执行图，禁止实现者自行采用改变样本或精度的近似。

1. **StockDayFeatureStore（全市场每日一次）：** 计算并按 `(trade_date, stock_code, raw_snapshot_hash, feature_code_hash)` 缓存与题材无关的股票日特征：r、x、TurnValue、TurnZ、CLV、Active、NewJoin 及 §9.2 原始特征中不依赖题材成员的项。**禁止**缓存 \(Rel_{i,t}=r_{i,t}-r^{-i}_{u,t}\) 或任何依赖 `theme_id` / `episode_id` / \(B^E\) 的相对量；此类量必须在题材上下文按 `(trade_date, stock_code, theme_id, episode_id, membership_hash)` 计算或缓存。滚动窗用 ring buffer 增量更新；结果必须与全窗口重算满足 §26.3 容差。
2. **TextFeatureStore（每条新文本一次）：** B-LEX 只增量更新 hash term frequency、expanding document frequency/IDF 与受影响公司120日衰减向量；B-EMB 只编码新增文本。每日公司向量必须与从该 regime 起全部 PIT 文本重算满足 §26.3 容差。
3. **DonorIndex（每日每层一次）：** 对行业×交易板块构建标准化向量、协方差和确定性近邻索引；同日所有真实题材共享。排除集在查询后施加，禁止为了共享索引而跳过 §8.3.1 排除。
4. **NullMembership：** 严格按 §8.3.3 load-or-build：确认前每题材日一次 DAILY_NULL；确认日一次并冻结 EPISODE_NULL；确认后只读加载。生成最多200个伪题材的 `recipient_member → donor_member` 对齐矩阵；IES、IDS、CCS、GCC、Gate 对照和 P10 必须共享对应阶段的冻结矩阵，不得各自重新抽样。
5. **PseudoThemeStats（每伪题材一次）：** 一次识别注意力核心与容量集合，保存 Breadth/NewJoin/Turn/Sign/Failure 的计数或和、ALS maxima、五个 CCS 原始特征及排序数组。
6. **候选 LOO 精确更新：**
   - Breadth、NewJoinRate、TurnBreadth、SignConcord、FailureRate 使用总计数减去对齐候选贡献后重算，时间复杂度 O(1)；
   - \(GCC^{-i}\) 从排序后的 CCS_level 数组精确删除 i 后取中位数，时间复杂度 O(log|K|)；
   - 非可加统计量不得用近似减法，必须从对应成员行精确重算。
7. **分位索引：** 每个 null 指标排序一次；§6.7 midrank 用左右二分边界计算，禁止逐候选全表扫描。
8. **并行确定性：** 并行分片键固定为 `Hash(theme_id,D) mod worker_count`；归约前按 `(theme_id,D,control_index,stock_code)` 稳定排序；worker_count 不得影响输出。

### 8.3.9 缓存合法性与运行 SLO

每层缓存键必须包含：

    spec_hash
    code_commit
    params_hash
    raw_snapshot_hash
    PIT_theme_hash
    text_mode_and_model_hash
    trade_date
    upstream_membership_hash
    random_seed_rule_version

任一字段变化即整层失效；禁止读取不完整缓存、跨 measurement regime 缓存或上一交易日 null 输出。缓存只减少重复计算，不得改变控制样本、随机流、分位或浮点精度。

运行目标与失败处置：

- **Tier A（决策关键）：** 真实题材的 200 controls、IES/IDS/CCS/GCC、状态、信号与退出，必须在 D 21:00 前完成；
- **Tier B（审计）：** P10 cross-fit、独立重发与全量诊断在信号归档后运行，最晚 D+1 12:00 完成；不得占用 Tier A 预留 worker。P10 当日未完成记审计故障并暂停次日新入场，但不得回写 D 日信号；
- **Tier C（离线法庭）：** bootstrap、5,000次角色置换、Shapley、HOLDOUT Gate 与敏感性只在研究/开庭集群运行，不进入每日21:30关键路径；
- D 日正式全链（数据落地后至信号归档）p95 ≤60分钟、p99 ≤90分钟；
- 20:45 运行预警；21:00 仍未完成的题材记 `COMPUTE_TIMEOUT`，不产生信号，不降采样、不减少 controls、不使用陈旧缓存；
- 当日 `COMPUTE_TIMEOUT` 题材占可评估题材 >10%，触发 `QUARANTINE_COMPUTE`，全日不发新单；
- 每日输出各 DAG 节点 wall time、CPU、内存、cache hit、重算数量、超时题材与哈希清单；
- FORWARD 前必须用过去60个交易日影子负载证明 p99 SLO；不达标不得 Freeze。

## 8.4 独立扩散分数 IDS（两级）

对观察集合 \(N\in\{N^{\theta}, N^{-i}\}\)：

\[
IDS_{u,D}(N)
=
Median(
Pctl_{null}(Breadth),
Pctl_{null}(NewJoinRate),
Pctl_{null}(TurnBreadth),
Pctl_{null}(SignConcord),
1-Pctl_{null}(FailureRate)
)
\]

其中每个 null 分位的参照集为 200 个伪题材上以同构规则（伪题材内同样构造 L 与观察集合）计算的同名变量。

- \(IDS^{\theta}_{u,D} := IDS_{u,D}(N^{\theta})\)：题材级，供状态机、Whipsaw5、Leg B2/X；
- \(IDS^{-i}_{u,D} := IDS_{u,D}(N^{-i})\)：候选级，仅供该候选入场资格与 \(CONF^{-i}\)。

**层级声明：** IDS 是对匹配 null 标准化的 level 类变量，可跨题材、跨日期比较，可构造门槛。这是 level 类变量的构造范式，\(CCS\_level\) 遵循同一范式（§9.2）。

## 8.5 点火证据 IES

**LeaderAmountShare（全文唯一定义）：**

\[
LeaderAmountShare_{u,D}
=
\frac{\sum_{j\in L_{u,D}} Amount_{j,D}}
{\sum_{j\in B_{u,D}} Amount_{j,D}}
\]

其中 \(L_{u,D},B_{u,D}\) 按 §3.5 / §8.1 阶段映射；分母为 0 → 该题材日记 `NON_EVALUABLE(AMOUNT_ZERO)`。Amount 为单位人民币成交额（与 §4.1 一致）。

点火证据同样相对匹配伪题材标准化：

\[
IES_{u,D}
=
Median(
Pctl_{null}(MaxALS),
Pctl_{null}(MaxBoardHeight),
Pctl_{null}(LeaderAmountShare)
)
\]

IES 为题材级变量（owner=题材）。MaxALS / MaxBoardHeight 在 \(L_{u,D}\) 上取最大；伪题材对照使用其同构临时或冻结 \(L_c\)。

## 8.6 确认条件

对候选股 \(i\)：

    IES >= 0.80
    IDS(-i) >= 0.80
    IDS(-i)_D > mean(IDS(-i)_{D-3:D-1})
    NewJoinRate(N^{-i}) > 0
    |N^{-i}| >= 5

题材协调强度：

\[
CONF^{-i}_{u,D}
=
\min(IES_{u,D},IDS^{-i}_{u,D})
\]

使用最小值，禁止强龙头与弱扩散互相补偿。

---

# 9. 容量角色与候选集合

## 9.1 容量是约束，不是预测信号

首次确认前，临时容量集合 \(K_{u,D}\) 按下列规则构造；首次确认时将其冻结为 \(K^E\)（§3.3）：

- 不属于注意力核心；
- 自由流通市值位于题材前40%；
- ADV20位于题材前40%；
- 满足 §5.1 信号侧容量资格（\(0.5\%\times ADV20_{i,D}\ge MinPositionValue\)，仅用 D 日可知信息）；
- D日未收盘涨停；
- 非停牌、非ST；
- 题材内至少3只容量候选。

大市值和流动性只决定"能否承载资金"，不决定"是否该买"。

首次确认后不再重选结构角色。每日只从 \(K^E\) 取满足当日可交易、订单容量、非涨停/停牌/ST及 §9.3 条件的 \(K^{trade}_{u,D}\)；新成员和原 \(K^E\) 外股票不得进入候选。

## 9.2 股票容量载体分数：双轨制

**构造约束：** 组内分位的横截面均值由构造锚定在 0.5 附近，只能表示同组相对顺序，不能表示需求绝对强度。因此组内 rank 与对匹配 null 标准化的 level 必须分轨构造，任何门槛或状态变量禁止消费 rank 轨。

**原则：** rank 与 level 分离，各归其位。

- **rank 类变量**（组内分位合成）：只回答"同一题材同一日内谁更强"，用于排序，不得跨组比较、不得构造门槛、不得聚合为状态变量；
- **level 类变量**（对匹配伪题材 null 标准化，同 §8.4 IDS 范式）：回答"相对于随机匹配股票异常多少"，可跨题材比较、可构造门槛、可聚合为 GCC 与状态输入。

### 9.2.1 原始特征（两轨共用）

相对题材加速度，题材收益剔除股票自身：

\[
r^{-i}_{u,t}
=
\frac{1}{|B^E\setminus\{i\}|}
\sum_{j\in B^E\setminus\{i\}}r_{j,t}
\]

\[
Rel_{i,t}=r_{i,t}-r^{-i}_{u,t}
\]

\[
RelAccel_{i,D}
=
Mean(Rel_{i,D-2:D})
-
Mean(Rel_{i,D-9:D-3})
\]

分歧日防守：**分歧日判定口径：** 过去10日中 \(r^{-i}_{u,t}<0\)（剔除自身的题材等权收益为负）的日期集合 \(T^{def}_i\)：

\[
Defense_{i,D}
=
Median_{t\in T^{def}_i}(Rel_{i,t})
\]

\(T^{def}_i=\varnothing\) 时记为中性（null 分位 0.5），不得填充高分。

需求持续性：

\[
Persist_{i,D}
=
\sum_{k=0}^{2}
\mathbf 1(
TurnZ_{i,D-k}>0
\land
Rel_{i,D-k}>0
\land
CLV_{i,D-k}>0.5
)
\]

五特征集合：\(\{TurnZ,\ RelAccel,\ CLV3,\ Defense,\ Persist\}\)。

### 9.2.2 CCS_rank（排序轨）

当日可交易候选集合 \(K^{trade}_{u,D}\) 内横截面分位等权（Pctl 按 §6.7）：

\[
CCS\_rank_{i,D}
=
Mean(
Pctl_{K}(TurnZ),
Pctl_{K}(RelAccel),
Pctl_{K}(CLV3),
Pctl_{K}(Defense),
Pctl_{K}(Persist)
)
\]

**唯一合法用途：** EntryScore 的选股排序成分（§13.2）与同分打破规则。禁止用于任何门槛、状态机、GCC 或跨题材比较。

### 9.2.3 CCS_level（层级轨）

每个 \(K^E\) 成员的原始特征先对匹配伪题材标准化：取该 episode 的 EPISODE_NULL（§8.3.3：\(\tau_{confirm}\) 日冻结的 200 个伪题材，伪确认日 = \(\tau_{confirm}\)）中各伪题材冻结的 \(K^E_c\) 全体成员作为 null 池，按 §6.7 的 midrank Pctl 计算经验分位：

\[
CCS\_level_{i,D}
=
Mean(
Pctl_{null}(TurnZ),
Pctl_{null}(RelAccel),
Pctl_{null}(CLV3),
Pctl_{null}(Defense),
Pctl_{null}(Persist)
)
\]

**并列处理：** Persist 为 {0,1,2,3} 离散变量，null 池必然大量并列，一律按 §6.7 midrank 处理；不得对 Persist 做不改变排序的缩放来替代并列规则。

**冻结成员缺失/停牌语义：**

- \(K^E\) 成员当日停牌、ST、退市整理或无成交时保留角色但不进入 \(K^{trade}\)，其当日 `CCS_level=0`、`DirectedDemand=0`，在 \(GCC^\theta\) 与迁移变量中按零需求计入；
- 正常交易但任一必需原始字段缺失属于数据故障，该题材日记 `NON_EVALUABLE(CAPACITY_FIELD_MISSING)`，不产生新信号；持仓退出仍可由已有 LAC/交易状态规则触发；
- 不得删除冻结成员后重算中位数或分母，不得以前值填充 CCS。

**合法用途：** 个股层级门槛、GCC 构造（§11）、状态机输入（经 GCC）、事件研究分组。

### 9.2.4 资格条件

个股层级门槛（level 轨）：

    CCS_level >= 0.60
    TurnZ > 0
    RelAccel > 0
    Persist >= 2

显式声明：`TurnZ>0 / RelAccel>0 / Persist>=2` 是成员资格门槛，承担闸门功能；`CCS_level>=0.60` 是层级门槛；`CCS_rank` 只排序。三者分工明确，不再有"偷偷承担闸门"的隐含条件。

## 9.3 过度扩张否决

股票满足任一项则不允许新买：

- 5日累计行业残差收益位于自身过去250日90%分位以上；
- D日收盘涨停；
- 最近3日中两日收盘涨停；
- D日跳空加当日涨幅位于相同交易板块99%分位；
- 5日涨幅达到注意力核心同期涨幅80%以上，且TurnZ处于自身90%历史分位以上。

目的不是寻找低位，而是排除已经完成主要扩张的容量股。

---

# 10. 角色迁移测量

本章全部指标固定使用 Entry Cohort，统一以上标 \(E\) 标识；它们是 H-MIG 与入场状态的正式变量，不随 LAC 成员变化。

## 10.1 定向需求

对股票 \(j\)：

\[
DirectedDemand_{j,D}
=
\max(TurnZ_{j,D},0)
\times
\mathbf 1(x_{j,D}>0)
\times
CLV_{j,D}
\]

这不是资金净流入，只是"异常成交伴随正向价格承接"的日频代理。
冻结角色成员当日停牌、ST、退市整理或无成交时，按 §9.2.3 定义 \(DirectedDemand=0\)。

## 10.2 需求份额

### 龙头需求份额

\[
LDS^E_{u,D}
=
\frac{
\sum_{j\in L^E}DirectedDemand_{j,D}
}{
\sum_{j\in B^E}DirectedDemand_{j,D}
}
\]

### 容量需求份额

\[
CDS^E_{u,D}
=
\frac{
\sum_{j\in K^E}DirectedDemand_{j,D}
}{
\sum_{j\in B^E}DirectedDemand_{j,D}
}
\]

## 10.3 正向价格贡献份额

\[
PriceShare^E_{g,D}
=
\frac{
\sum_{j\in g^E}\max(x_{j,D},0)
}{
\sum_{j\in B^E}\max(x_{j,D},0)
}
\]

分别令 \(g^E=L^E,K^E\) 计算。LDS、CDS 或 PriceShare 的分母为0时，对应份额定义为0，不使用 \(\epsilon\) 近似。

## 10.4 迁移指数 \(RMI^E\)

先取3日EMA，再取 \(\Delta_3\)（§6.6、§6.8 定义）：

\[
RMI^E_{u,D}
=
\frac12
\left[
\Delta_3 EMA_3(CDS^E)
-
\Delta_3 EMA_3(LDS^E)
\right]
+
\frac12
\left[
\Delta_3 EMA_3(PriceShare^E_K)
-
\Delta_3 EMA_3(PriceShare^E_L)
\right]
\]

\(RMI^E>0\) 表示冻结 Entry Cohort 内的定向需求和正向价格贡献正在相对向容量组移动。

## 10.5 容量广度

\[
CapacityBreadth^E_{u,D}
=
\frac{
\sum_{j\in K^E}
\mathbf 1(TurnZ_j>0\land RelAccel_j>0)
}{
|K^E|
}
\]

## 10.6 需求集中度与健康迁移/崩溃

### 需求集中度

\[
HHI^E_{u,D}
=
\sum_{j\in B^E}
\left(
\frac{DirectedDemand_{j,D}}{\sum_{k\in B^E}DirectedDemand_{k,D}}
\right)^2
\]

当 \(\sum_{k\in B^E}DirectedDemand_{k,D}=0\) 时，\(HHI^E=0\)。

"正向需求集中度上升"在全文中定义为 \(\Delta_3 EMA3(HHI^E) > 0\)。

### 题材级相对收益与成交额份额（全文唯一定义）

以下四个题材级变量供状态机与退出腿消费，集合在确认前用 \(B_{u,D-1}\) 合格成员、确认后用 \(B^E\)：

1. **题材等权残差收益：** \(\bar x_{B,D}=Mean_{j}(x_{j,D})\)（当日有 \(x\) 的成员等权；全缺失记不可判定，条件按不满足）；
2. **题材 k 日相对市场收益：**

\[
RelMkt_k(u,D)
=
\prod_{t=D-k+1}^{D}(1+\bar r_{B,t})
-
\prod_{t=D-k+1}^{D}(1+r^{\text{全A等权}}_t)
\]

（\(\bar r_{B,t}\) 为成员等权收益，基准按 DG15；"题材5日相对市场收益" = \(RelMkt_5\)，"题材20日相对宽基收益" = \(RelMkt_{20}\)；窗内成员集合固定为当日口径的 B/\(B^E\)，逐日用其当日可交易成员等权。）

3. **题材成交额市场份额：**

\[
AmtShare_{u,D}
=
\frac{\sum_{j\in B}Amount_{j,D}}{\text{当日全市场 §5.1 合格股票 Amount 总和}}
\]

其"自身250日分位"（§12.7）按当日成员集合固定回算过去250个交易日的份额历史后取 §6.7 分位；有效历史（成员非全停牌日）不足120日时该条件按不满足并登记 `INSUFFICIENT_HISTORY`。

### 健康迁移 Healthy Transfer

    RMI^E > 0
    CapacityBreadth^E >= 0.40
    Δ3 EMA3(CapacityBreadth^E) > 0
    IDS^θ 较过去3日最高值下降不超过0.10
    题材等权残差收益 >= 0

### 共振扩张 Co-expansion

龙头和容量需求同时上升：

    Δ3 LDS^E >= 0
    Δ3 CDS^E > 0
    CapacityBreadth^E >= 0.40
    Δ3 EMA3(IDS^θ) > 0

共振扩张也允许入场，因为容量需求已经开始，但龙头尚未交接。

### 题材崩溃 Collapse

    Δ3 LDS^E < 0
    Δ3 CDS^E < 0
    Δ3 EMA3(IDS^θ) < 0
    CapacityBreadth^E < 0.30
    题材5日相对市场收益 < 0

"龙头断板"单独不构成崩溃。

---

# 11. 容量时钟

## 11.1 两级容量时钟

### 题材级（供状态机）

\[
GCC^{\theta}_{u,D}
=
Median_{j\in K^E}(CCS\_level_{j,D})
\]

### 候选级（供入场资格）：候选股不能批准自己的容量阶段

对候选股 \(i\)：

\[
K^{-i}_{u,D}=K^E_u\setminus\{i\},
\quad |K^{-i}|\ge2
\]

\[
GCC^{-i}_{u,D}
=
Median_{j\in K^{-i}}(CCS\_level_{j,D})
\]

## 11.2 时钟斜率

\[
GCCSlope^{\bullet}_{u,D}
=
GCC^{\bullet}_{u,D}
-
Mean(GCC^{\bullet}_{u,D-3:D-1})
\]

（\(\bullet\in\{\theta,-i\}\)，两级各自计算。）

由于 \(CCS\_level\) 对匹配 null 标准化，GCC 不被锚定：容量组整体需求强于随机匹配股票时 GCC 上移，弱于时下移。GCC 是 level 类变量，可跨题材比较、可构造启动带。

## 11.3 容量时钟启动

启动判据对两级各自适用（状态机用 \(\theta\) 级，入场资格用 \(-i\) 级）：

    GCC ∈ [0.55, 0.85]        （level 尺度）
    GCCSlope > 0
    对应集合（K^E 或 K^{-i}）中至少1只股票满足：
        TurnZ > 0
        RelAccel > 0
        Persist >= 2

语义：下限 0.55 要求容量组中位个股的异常需求特征超过 55% 的匹配 null（区别于随机），上限 0.85 排除容量组中位个股已超过 85% null 的全面拥挤状态。启动带数值为预注册先验，其敏感性必须在 Gate 5 报告中给出；不允许在评估样本中选择"最优"带边界替换主口径。

## 11.4 容量事件时间

- **\(\tau_{confirm}\)（v7 唯一定义，A-02）：** 首次同时满足以下三个题材级条件的交易日：

\[
IES_{u,D}\ge0.80
\ \land\
IDS^{\theta}_{u,D}\ge0.80
\ \land\
IDS^{\theta}_{u,D}>Mean(IDS^{\theta}_{u,D-3:D-1})
\]

  斜率条件所需的 3 日 IDS 历史不足（含 NON_EVALUABLE 缺口）时按不满足。该 IDS 历史**必须**取自确认前已归档的临时 DAILY_NULL 量尺（§3.5.1）；FrozenMetricBackcast **不得**改写本谓词。该定义即 §12.3 CONFIRMED_EXPANSION 的前三行；**不包含**容量时钟条件，**不包含**任何候选级（\(IDS^{-i}\)、NewJoinRate、\(|N^{-i}|\)）条件——候选级条件只作用于该候选的入场资格（§8.6、§13.1），不影响 \(\tau_{confirm}\)、Entry Cohort 与 \(B^E/L^E/K^E\) 的冻结日。episode 于 \(\tau_{confirm}\) 日创建，全部冻结集合于当日收盘一次性锁定；
- \(\tau_{capacity}\)：首次题材级容量时钟启动日；
- \(\tau_{migration}\)：首次 \(RMI^E>0\) 且 \(CapacityBreadth^E\) 达标日。

记录：

\[
Lag_C=\tau_{capacity}-\tau_{confirm}
\]

\[
Lag_M=\tau_{migration}-\tau_{confirm}
\]

不预设固定D3—D5。事件研究检验实际分布。

---

# 12. 题材生命周期状态机

    DORMANT
      ↓
    DISCOVERED
      ↓
    IGNITION
      ↓
    CONFIRMED_EXPANSION
      ↓
    EARLY_CAPACITY / HEALTHY_TRANSFER
      ↓
    BROAD_TREND
      ↓
    CONCENTRATED_CLIMAX
      ↓
    COLLAPSE
      ↓
    COOLDOWN
      ↓
    DORMANT

## 12.0 每日判定规则（每日输入 → 唯一状态）

状态机为**题材级对象**，入场与迁移状态只消费冻结题材级变量（\(IES\)、\(IDS^{\theta}\)、\(GCC^{\theta}\)、\(GCCSlope^{\theta}\)、\(RMI^E\)、\(CapacityBreadth^E\)、\(HHI^E\)）；持仓后另允许消费 `LACDecay` 进入 COLLAPSE 独立分支。归属表见 §3.4。每日按以下**固定优先序**自上而下判定，取第一个满足者：

    1. COOLDOWN（处于冷却计数期，见 §12.9）
    2. COLLAPSE（吸收规则，见 §12.8）
    3. CONCENTRATED_CLIMAX
    4. BROAD_TREND
    5. HEALTHY_TRANSFER
    6. EARLY_CAPACITY
    7. CONFIRMED_EXPANSION
    8. IGNITION
    9. DISCOVERED
    10. DORMANT（不满足任何在前状态）

多状态条件同日并存时由优先序唯一裁决。

**NON_EVALUABLE 冻结范围（v8 唯一）：** 题材日为 `NON_EVALUABLE`（§8.3.4/8.3.5/§9.2.3）时：

- **冻结：** IES / IDS / GCC / RMI / CapacityBreadth / HHI 驱动的正常状态转移，以及一切新入场信号；
- **不冻结（仍按优先序执行）：** `LACDecay=1` → COLLAPSE（Leg X）；停牌与执行状态更新；已有退出订单与升级（§14.3）；安全 Kill；账本恒等式；
- 登记 `NON_EVALUABLE` 事件，并记录是否触发了不冻结风险分支。

## 12.1 DISCOVERED

题材已在D−1以前进入Base Universe，但D日尚未满足点火和扩散确认。

## 12.2 IGNITION

    IES >= 0.80
    IDS^θ < 0.80

有人点火，但独立扩散不足，不买。

## 12.3 CONFIRMED_EXPANSION

    IES >= 0.80
    IDS^θ >= 0.80
    IDS^θ 斜率 > 0（IDS^θ_D > mean(IDS^θ_{D-3:D-1})）
    题材级容量时钟尚未启动

题材已确认，但容量载体尚未形成，不买。

## 12.4 EARLY_CAPACITY

    CONFIRMED_EXPANSION 的前三行成立
    GCC^θ 启动（§11.3，level 尺度）
    CapacityBreadth^E >= 0.40
    RMI^E >= 0 或处于Co-expansion
    未过度拥挤（GCC^θ <= 0.85）

允许入场。

## 12.5 HEALTHY_TRANSFER

    IES >= 0.80 且 IDS^θ 较过去3日最高值下降不超过0.10
    GCC^θ 启动
    RMI^E > 0
    Δ3 EMA3(CapacityBreadth^E) > 0

允许入场或继续持有。

## 12.6 BROAD_TREND

    IDS^θ >= 0.70
    GCC^θ >= 0.70（level 尺度）
    RMI^E >= -0.02
    CapacityBreadth^E >= 0.40

允许持有，不再追加入场。\(RMI^E\) 下限 −0.02 为登记先验，入 §27。

## 12.7 CONCENTRATED_CLIMAX

同时满足：

    题材成交额占全市场份额处于自身250日90%分位以上
    IDS^θ 斜率 <= 0
    GCCSlope^θ <= 0
    炸板失败率处于匹配伪题材80%分位以上
    Δ3 EMA3(HHI^E) > 0

单纯高成交占比不构成高潮。

**触发频率预注册：** 五条件 AND 的触发频率（占全部状态日比例与各 episode 触发率）必须作为 Gate 报告的强制项。若全样本触发频率 < 1% 状态日，CLIMAX 定义标记为 `LOW_POWER_CLIMAX`，并激活预注册的敏感性方案：报告 4/5 条件降级版本的触发频率与 Leg B1 效果对比。降级版本只出现在敏感性报告中，不替换主口径；若需替换，走规格修订流程。

## 12.8 COLLAPSE（吸收态）

进入条件（满足任一）：

    §10.6 Collapse 五条件；或
    连续2日 IDS^θ < 0.30 AND GCCSlope^θ < 0；或
    持仓 episode 的 LACDecay = 1

进入规则：

- 进入当日触发 Leg X 退出（§16.2），优先级最高；
- **吸收：** 进入后每日保持 COLLAPSE，直至该 episode 全部持仓清算完成（含卖出顺延），随后转入 COOLDOWN。

## 12.9 COOLDOWN

- 自触发关闭事件（§12.10）次一交易日起计数 10 个交易日；
- 冷却期内该题材及其按 §5.3 连通分量定义的同簇题材**不得确认新 episode**、不得产生任何入场信号；
- 冷却期满：episode 关闭归档，题材回 DORMANT，新 episode 可从 DISCOVERED 重新评估。

## 12.10 episode 关闭路径穷举（v7 新增 A-03；v8 维持）

每个 episode 必须且只能经由下列之一进入 COOLDOWN；关闭次日起退出 §8.3.1"进行中 episode"排除集：

|关闭码|触发条件|COOLDOWN 起点|
|---|---|---|
|CLOSE_COLLAPSE|COLLAPSE 且全部持仓清算完成（§12.8）|清算完成次日|
|CLOSE_ARGUMENT_EXPIRY|Leg C 60 日到期平仓完成|平仓完成次日|
|CLOSE_ALL_LEGS|全部持仓经个股腿（Leg I/B2 等）退出、无剩余持仓且无有效信号|最后平仓完成次日|
|CLOSE_NO_ROLE_COHORT|\(\tau_{confirm}\) 日 \(|K^E|<3\)（NO_ROLE_COHORT）|\(\tau_{confirm}\) 次日|
|CLOSE_ENTRY_TIMEOUT|自 \(\tau_{confirm}\) 起连续 15 个交易日未进入 {EARLY_CAPACITY, HEALTHY_TRANSFER}|第 15 日次日|
|CLOSE_SIGNALS_VOID|主信号窗已开，且（主信号日未产生任何信号）或（全部信号作废，含有效期耗尽），且无任何成交、无持仓|有信号：最后一个信号作废次日；零信号：主信号日次日|

规则：

1. 多个关闭条件同日成立时按上表自上而下取第一个；
2. 无持仓关闭（后三行）同样进入 10 个交易日 COOLDOWN——防止同题材立即重新确认造成的重复点火；
3. `CLOSE_NO_ROLE_COHORT` 与 `CLOSE_ENTRY_TIMEOUT` 的发现/确认观测保留在 Gate 1/2 样本，其后续容量/交易观测不存在；
4. 开窗超时参数 15 个交易日为登记先验（§27）；
5. 除本表与 §5.4 合并外，episode 不存在其他退出"进行中"状态的方式。

本 §12.8–§12.10 是 episode 关闭与重新点火的唯一合法路径。

---

# 13. 入场规则

## 13.1 信号日D

**主信号窗：** 每个 episode 仅有一次主信号窗：episode 内**首次进入集合 {EARLY_CAPACITY, HEALTHY_TRANSFER}（以先发生者计）**的当日为唯一主信号日 D。此后无论状态如何往返，该 episode 不再产生主信号。

主信号日只对 \(K^{trade}_{u,D}\) 中每只容量候选执行（题材级状态由 §12 判定，候选级资格如下）：

    CONF(-i) 达标（§8.6 五条件）
    GCC(-i) 启动（§11.3，候选级）
    CCS_level(i) >= 0.60 且满足 §9.2.4 资格门槛
    未过度扩张（§9.3）
    D日可交易

## 13.2 排序

候选分数：

\[
EntryScore_i
=
0.40\times CCS\_rank_i
+
0.25\times CONF^{-i}
+
0.20\times Pctl_{cross}(RMI^E)
+
0.15\times Pctl_{cross}(GCCSlope^{\theta})
\]

**分位基准（v9 唯一）：** \(Pctl_{cross}\)（按 §6.7）的参考集 = 同日所有**已确认且仍开放**的 episode，且其 \(RMI^E\) 与 \(GCCSlope^{\theta}\) 均可按冻结 \(B^E/L^E/K^E\) 合法计算者。禁止将仅 DISCOVERED/IGNITION、缺少冻结角色、或 RMI/GCCSlope 缺失的题材填 0 / 剔除后再混入同一分位池。若参考集 < 5，当日全部候选 EntryScore 中 \(Pctl_{cross}\) 两项记中性值 0.50 并登记 `PCTL_CROSS_SPARSE`。题材级变量每 episode 日只贡献一个观测，取 episode 间分位后映射到其候选股。\(CCS\_rank\) 为组内排序成分（§9.2.2），\(CONF\) 本身已是 level 尺度。

这些权重是冻结前登记的工程权重，不通过网格优化。敏感性仅比较等权版本，不选择较优者替换主口径。

每题材选择最多2只。

并列顺序：

1. CCS_rank高；
2. ADV20高；
3. 股票代码升序。

## 13.3 同一episode规则与信号有效期

- 首次入场后不加仓；
- 平仓后不在同一episode重新入场；
- **信号有效期：** D 日信号在 D+1 执行；若因 §14.1 拒绝、部分成交余额（EXPIRED_REMAINDER）、§15 组合拒绝（REJECT_PORTFOLIO_FULL / REJECT_MIN_WEIGHT）或未成交，允许在 D+2、D+3 重试剩余目标金额，每个重试日开盘前重新校验：题材状态仍为 EARLY_CAPACITY/HEALTHY_TRANSFER、个股未触发过度扩张否决、CONF 仍达标。D+3 收盘仍有未成交余额，余额作废；
- **入场权消耗：** 每只候选股在一个 episode 生命周期内只有一次主信号入场权；首次非零成交或全部余额作废均消耗该权利（部分成交后余额重试不产生新入场权）。**组合拒绝不提前消耗入场权**（有效期内仍可重试；有效期尽则作废并消耗）。episode 入场归因以信号日 D 计；**\(\tau_{entry}\) := 首次非零成交日**；入场成本 = 全部有效期内成交的数量加权净成本，持有计数自 \(\tau_{entry}\) 起（后续追补成交共享同一退出时钟）。差异逐笔登记并纳入执行分析；
- 未成交不改写状态史：episode 状态史的首次进入日以状态机实际首次进入日计，与成交无关；
- episode 结束按 §12.8–12.9 闭环（COLLAPSE 清算 + COOLDOWN 期满，或 Leg C 到期平仓后进入 COOLDOWN 同规则）；
- 新episode可重新评估（受 §12.9 冷却约束）。

---

# 14. 执行规则

**执行时钟不变量（v7 B-02；v8 维持）：** 任何交易决策只能使用其决策时点前已经完整可见的行情；禁止使用尚未结束窗口的 VWAP、成交量或任何聚合量决定同一窗口内的成交。v6 以 09:35–10:00 完整 VWAP 决定是否在同一窗口按该 VWAP 成交，属执行时钟穿越，已废止。

## 14.1 买入执行算法（决策 → 流式参与）

首个执行日为 D+1；重试执行日为 D+2、D+3。

**决策时点：09:35 整。** 此刻可见信息 = 集合竞价结果 + 09:30–09:34 五根完整 1 分钟线。

**条件跳空（决策时点可见）：**

\[
g_{i,t}
=
\frac{VWAP^{09:30-09:34}_{i,t}}{Close_{i,t-1}}-1
\]

09:30–09:34 无成交时用开盘价替代分子；开盘亦无成交（全程停牌/一字无量）→ 当日按 UNFILLED_SUSPENSION 进入重试。选择首 5 分钟 VWAP 而非开盘价单点：单点易受集合竞价噪声支配，5 分钟 VWAP 在冻结数据域（1 分钟线）内可复现且 09:35 已完整可见。

**拒绝判定（全部在 09:35 决策时点完成）：**

- 09:30—09:34 期间停牌 → UNFILLED_SUSPENSION；
- **开盘涨停拒单（v12 唯一）：** 当且仅当 09:30—09:34 五分钟**始终**满足 \(Low_m = up\_limit\)（连续无量/无开板封死）→ `UNFILLED_LIMIT`；若 09:30 以涨停价开盘但 09:30—09:34 任一分钟出现 \(Low_m < up\_limit\)（已开板），则**不得**因此拒单，应从开板后首个可参与分钟起流式参与；
- 订单超过 §5.1 信号侧上限 → REJECT_CAPACITY；
- **REJECT_GAP：**

    g > Q0.85(g | 历史同状态信号, 同交易板块)

其中 Q0.85 按 §6.7 唯一定义。阈值分布使用与本节相同的 g 定义，取自历史同状态（EARLY_CAPACITY / HEALTHY_TRANSFER 分别统计）、同交易板块信号的条件跳空经验分布。

**阈值版本规则（v8 唯一）：**

- 执行日 t 的阈值版本只使用**执行日期严格早于 t**、且已于 ARCHIVE 时钟归档的理论信号（含被 REJECT_GAP 拒绝者；不含从未生成理论订单的题材日）；
- 同一交易日全部订单共享该日开盘前冻结的阈值版本，禁止按订单处理顺序更新；
- 累计归档样本从 39 增至 40 时，**下一交易日**才由冷启动先验切换为经验 Q0.85；当日不切换；
- 未成交（停牌/涨停）但已生成理论信号且 g 可计算者仍进入历史分布；g 不可计算者不进入。

**09:35 资金释放与再分配（v8 唯一）：**

1. 先对全部当日待执行买单（新单+重试单）完成 §14.1 拒绝判定（含 REJECT_GAP）；
2. 被拒订单立即释放其 ReservedCash 与题材簇槽位占用；
3. 释放后的 FreeCash 按 §15.1 统一排序，在同一 09:35 决策时点重新分配给仍合格且尚未获得足额预留的订单（含原排在后面的新单）；
4. 再分配不得改变已通过拒绝判定的订单的合格状态；不得引入新的未来信息；
5. 完成再分配后进入流式参与；当日 09:35 之后不再因新的拒绝做第二次全市场再分配（流式过程中的部分成交只按 §15.2 调整该订单自身预留）。

**人民币母订单与流式参与（v12 唯一；09:35–10:00，共 25 根分钟线）：**
母订单以人民币为意图单位，字段：

    ParentOrderTargetValue   # 再分配后的 TargetValue
    RemainingTargetValue     # 09:35 初值见下；跨日继承
    CommissionReserve        # 当日最低佣金预留
    signal_id / parent_order_id

**禁止**在信号生成时把 TargetValue 一次性固定为不可变 TargetQty 作为跨日母目标。

**日内最低佣金预留（v12）：**

1. 09:35 对每个日子订单设定 `CommissionReserve = minimum_commission_cny`（按当日 fee_schedule）；
2. `RemainingTargetValue` 初值 = `ParentOrderTargetValue - CommissionReserve`（跨日重试日对剩余目标同样预留一次当日最低佣金）；
3. 分钟推进时，费用中的佣金部分**只按比例佣金** `OrderValue_m × commission_rate` 从 Remaining 扣减，**禁止**在分钟内重复扣最低佣金；
4. 日终一次结算：令 `DayOrderValue` = 该日子订单全部分钟成交金额之和（按执行价 \(P^{ex}\) 计）；`DayCommission = max(DayOrderValue×commission_rate, minimum_commission_cny)`；将已按比例累计扣减的佣金与 `DayCommission` 的差额从 Cash 补扣或释放；若补扣将使净成本超过原 `ParentOrderTargetValue`，则按时间倒序减少最后一笔 fill_qty（100 股步进）直至满足，差额记 `COMMISSION_TAIL_ADJUST`；
5. 未使用的 `CommissionReserve` 在日终释放回 FreeCash（若最终适用比例佣金且低于最低佣金则按上款补齐）；
6. **预算闭包：** 日终调整后，该母订单生命周期净买入成本（含全部费用与冲击）\(\le ParentOrderTargetValue\)（允许因 100 股取整产生的 \(\le\) 一手金额的未使用余额，不得上超）。

每分钟 m（时间顺序，**顺序冲击 + 单次计入**）：

1. 若 \(Low_m=up\_limit\)（全程封板）→ 可参与股数 = 0；否则 `particip_qty = floor(0.05×vol_m / 100)×100`（vol 已为股）；
2. 令 `CumOrderValue_{m-}` = 本日子订单截至上一分钟的累计成交金额（首分钟为 0）；先用无冲击价估计本分钟拟成交金额 `TentativeValue_m`；
3. `ImpactBps_m` 按 §14.4，以 `OrderValue_m^{impact} = CumOrderValue_{m-} + TentativeValue_m` 代入；**禁止**使用全日终总额或尚未发生的未来分钟金额；**禁止**回写已完成分钟的成交价或数量；
4. **执行价（唯一）：** 买入 \(P^{ex}_m = VWAP_m × (1 + ImpactBps_m/10^4)\)；卖出同构用 \(1 - ImpactBps_m/10^4\)。若 VWAP=0 则该分钟不成交；用 \(P^{ex}_m\) 重算数量一次（单次前向修正）；
5. `max_qty_by_value = floor(RemainingTargetValue / P^{ex}_m / 100)×100`；
6. `fill_qty = min(max_qty_by_value, particip_qty)`；**成交记账价 = \(P^{ex}_m\)**（已含冲击）；再仅加法定/经纪费用（佣金比例部分、印花税、过户/交易所费）；
7. **Impact Single-Count Rule（v12）：** 冲击只通过 \(P^{ex}\) 进入净成本一次；§14.4 输出的“容量冲击/固定滑点”金额仅为同一楔形的分解披露，**禁止**在 Cash/\(R^{net}\)/PnL 中再次扣除；
8. `RemainingTargetValue -=` 该分钟实际净买入成本（\(fill\_qty×P^{ex}_m\) + 比例费用；不含日终最低佣金差额）；不得为负（若因取整将越过则减少 fill_qty）；
9. 累计至 10:00 或 RemainingTargetValue < 一手最低金额；
10. 全部分钟可参与为 0 → UNFILLED_LIMIT；
11. 部分完成 → `PARTIALLY_FILLED`，当日 `EXPIRED_REMAINDER`，**跨日重试继承 RemainingTargetValue（人民币）**，重试日 09:35 重新做拒绝判定；禁止把首日股数冻结为跨日目标；
12. 有效期尽仍有剩余 → 余额作废。

**"涨停可成交"的唯一判据（v12）：** 分钟线 \(Low_m<up\_limit\) 的分钟才有非零可参与量；开盘涨停后若在 09:30—09:34 已开板，09:35 不得拒单，只能从开板分钟起流式参与；不允许回填为开盘价或窗口 VWAP 成交。

**冷启动兜底：** 同状态同板块累计信号数 < 40 时，使用临时阈值：主板 0.06，创业板/科创板 0.09，北交所 0.12（固定冷启动先验，入 §27 参数表，样本达标后自动切换为经验分位，切换事件登记）。

**不顺延追买：** 因 REJECT_GAP 未成交的信号按 §13.3 在 D+2、D+3 重试，重试日重新计算 g 与拒绝判定；不允许以提高限价或放宽参与率的方式追买。

拒绝/未成交记录集：

    UNFILLED_LIMIT
    UNFILLED_SUSPENSION
    REJECT_GAP
    REJECT_CAPACITY
    REJECT_PORTFOLIO_FULL
    REJECT_MIN_WEIGHT
    EXPIRED_REMAINDER

## 14.3 卖出执行

D日收盘后形成退出信号，D+1 14:30—15:00 与买入同构的流式参与执行：

1. 决策时点 14:30；14:30 前出现一字跌停（\(High=down\_limit\)）或停牌 → 顺延；
2. **T+1 可卖库存（v12 唯一）：**
   \[
   SellableQty_t
   =
   SettledShares_{t}^{open}
   -
   PendingSellReserved_t
   \]
   **开盘前结算顺序（强制）：**（i）对公司行动按 §15.5 在开盘前生效；（ii）将**昨日** `UnsettledBuyShares` roll 入 `SettledShares`；（iii）再更新 `SellableQty`。当日新买写入新的 `UnsettledBuyShares`，**当日不得**被 roll 为 Settled，也不得进入当日 `SellableQty`。卖出订单只能消费 `SellableQty`；计划卖出超过可卖时只下达可卖部分，差额顺延并登记 `T1_UNSELLABLE`；
3. 14:30–15:00 逐分钟参与：分钟 \(High_m=down\_limit\) 可参与量为 0，否则成交量 × 5%；成交数量 ≤ 剩余可卖；**成交记账价 =** \(VWAP_m×(1-ImpactBps_m/10^4)\)（冲击单次计入，与 §14.1/§14.4 同构）；
4. 未完成余额记 `PARTIALLY_FILLED` 并逐日顺延；顺延期间继续计入净值与风险；
5. 不允许使用窗口结束后才可知的聚合量回填成交。

**未完成卖单升级（v8 唯一）：** 若某股票仍有未完成卖出母订单（PARTIALLY_FILLED / 顺延中），且收盘后触发更高优先级退出腿（优先级 X > I > B2 > B1 > C）：

1. 取消原母订单未完成余额（状态 → `CANCELLED_UPGRADE`），已成交数量保留；
2. 按新腿目标持仓计算剩余应卖数量 = min(当前持仓 − 新腿目标持仓, SellableQty)（Leg X/B2/C/I 目标为 0；Leg B1 目标为触发前持仓的 50%，若已卖出超过半数则不再新建）；不可卖部分顺延；
3. 若同股票存在多个待售母订单，先按创建时间升序全部取消未完成余额，再合并为一个新母订单，数量为上款剩余应卖数量；
4. 最低佣金按新母订单在新执行日重新计一次，不追溯已取消母订单；
5. 升级事件必须入账本审计。

## 14.4 费用与冲击

所有法定与经纪费用从按日期版本化的 `fee_schedule.yaml` 读取。该文件属于 PIT 输入事实，其唯一 schema 为：

|字段|类型|含义|
|---|---|---|
|effective_from / effective_to|交易日（闭区间）|费率生效区间，不得重叠|
|board|枚举|MAIN / CHINEXT / STAR / BSE / ALL|
|side|枚举|BUY / SELL / BOTH|
|commission_rate|非负小数|经纪佣金率|
|minimum_commission_cny|非负人民币元|每个交易日每股票每方向子订单的最低佣金|
|stamp_tax_rate|非负小数|印花税率|
|transfer_fee_rate|非负小数|过户费率|
|exchange_fee_rate|非负小数|交易所及监管费率合计|
|source_url / published_at / available_at|字符串/时间戳|官方或经纪合同事实源与 PIT 时钟|
|source_hash|SHA-256|事实源内容哈希|

计费单元为**日子订单**：同股票、同方向、同交易日的全部分钟成交合并为一个子订单（跨日重试的成交分属各日子订单，最低佣金按日各自计收）。对每个日子订单：

\[
Fee
=
\max(OrderValue\times commission\_rate,\ minimum\_commission\_cny)
+OrderValue\times(stamp\_tax\_rate+transfer\_fee\_rate+exchange\_fee\_rate)
\]

按订单交易日、板块和方向选择唯一有效记录；没有唯一记录或 `available_at` 晚于回测当时可用时钟，记 `INVALID_DATA`。费率值来自当日可知的制度事实，不得由策略收益校准。

冲击模型：

\[
ImpactBps
=
\underbrace{fixed\_slippage\_bps}_{\text{固定滑点}}
+
\underbrace{10^4\lambda\sigma_{20}
\sqrt{
\frac{OrderValue}{ADV20}
}
}_{\text{容量冲击}}
\]

其中：

    fixed_slippage_bps = 5
    lambda = 0.50

\(\sigma_{20}\) 使用日收益率小数尺度；乘以 \(10^4\) 后容量冲击单位为 bps。**顺序冲击 OrderValue（v12）：** 买入流式参与中，第 m 分钟的 `OrderValue` = 截至该分钟（含本分钟拟成交）的累计成交金额；卖出同构。**禁止**用日终总成交额决定盘中分钟的 \(P^{ex}\) 或 fill_qty。日终可另输出“事后全日冲击诊断”，不得回写成交。

**Impact Single-Count（与 §14.1 同构）：** 买入执行价 \(=VWAP×(1+ImpactBps/10^4)\)，卖出执行价 \(=VWAP×(1-ImpactBps/10^4)\)；该楔形即冲击成本的**唯一**计入点。敏感性固定报告 `fixed_slippage_bps ∈ {3,10}` 与 `lambda ∈ {0.25,1.00}`，不得择优替换主值。

必须分别输出（分解披露，非叠加扣费）：

- 佣金；
- 印花税；
- 交易所及过户费用；
- 固定滑点（已含于 \(P^{ex}\) 的分解项）；
- 容量冲击（已含于 \(P^{ex}\) 的分解项）；
- 未成交机会损失；
- 总净成本（= 执行价名义 + 法定/经纪费用；**不得** = 执行价名义 + 费用 + 再加一轮冲击）。

---

# 15. 组合构建与交易账本

## 15.0 组合硬约束

- 每只股票初始上限：10%；
- 每题材簇上限：20%；
- 同时持有题材簇上限：3；
- 总风险仓位上限：60%；
- 无合格信号时持有现金；
- 不强制满仓；
- 同一股票属于多个题材时只持有一份；**其全部权重唯一计入 origin episode（首个持有它的 episode）入场时所属题材簇的 ThemeRoom**；多题材成员身份逐日登记披露；账务 `MERGED_INTO` **不得**迁移该归属（§5.4）；
- 组合已满时后续信号进入有效期重试，不跨有效期排队。

**题材簇上限时点语义（v12 唯一）：** ThemeRoom 在**入场分配时**按当日 §5.3 连通分量计算（entry-time constraint），并按 origin 永久归属。若持仓后因连通分量动态合并导致某 persistent/当日簇合计暴露 >20%：

1. **禁止**对该簇任何题材新开买入或提高预留；
2. **不强制**减仓到 20%（非 continuous forced delever）；
3. 依赖既有退出腿自然降低暴露；
4. 事件登记 `CLUSTER_OVER_CAP_NO_NEW_BUY`。

禁止实现为“按比例强制卖出”或“只保留较早 episode”而未预注册。

## 15.1 目标权重与订单金额

新信号的基础目标权重固定为 NAV 的 10%。对候选 i 的最终目标订单价值：

\[
TargetValue_i
=
\min(
0.10\times NAV,\,
StockRoom_i,\,
ThemeRoom_u,\,
GrossRoom,\,
CapacityLimit_i,\,
AvailableCash
)
\]

其中：

- \(StockRoom_i=0.10NAV-CurrentValue_i\)；
- \(ThemeRoom_u=0.20NAV-CurrentThemeValue_u\)；
- \(GrossRoom=0.60NAV-CurrentGrossValue\)；
- \(CapacityLimit_i\) 按 §5.1；
- \(AvailableCash\) 已扣除 §15.2 的现金预留。

若 \(TargetValue_i < 0.02NAV\)，不创建碎片仓位，记 `REJECT_MIN_WEIGHT`；否则订单金额 = TargetValue。该 2% 下限为工程先验，入 §27。

同日全部新信号与重试信号按以下统一顺序分配：

1. 当日重新计算的 EntryScore 降序（重试信号不复用此前分数）；
2. 信号原始日升序（较早信号优先）；
3. ADV20 降序；
4. 股票代码升序。

## 15.2 现金、槽位与重试预留

- D 日收盘后生成信号时，按 §15.1 排序写入初始预留（ReservedCash）；该预留是 D+1 09:35 前的占位，**不是**最终成交分配；
- D+1 09:35 按 §14.1 先完成拒绝判定，释放被拒预留，再对仍合格订单统一再分配（§14.1）；
- D+1 因执行拒绝或组合拒绝未成交后，为该信号预留其当日（再分配后）TargetValue 至下个执行窗口；
- 每日收盘后按最新 NAV 与硬约束重算预留，取原预留与新 TargetValue 较小者；
- 预留资金计现金、不计风险仓位；在下一 09:35 再分配时点前不得分配给其他信号；
- 信号全额成交、全部余额作废、状态复核失败或有效期耗尽时立即释放预留；部分成交后按已成交金额等额转出预留，剩余预留跟随余额重试；
- 预留信号占用题材簇槽位，但不占用股票持仓权重；同一题材的预留+持仓合计不得超过20%；
- 若多个重试预留与硬约束冲突，按 §15.1 统一排序保留，后序记 `REJECT_PORTFOLIO_FULL`，仍可在剩余有效期内再次竞争。

## 15.3 账本恒等式与 CI 闸门

每个时点必须满足：

\[
NAV_t = Cash_t + \sum_i PositionValue_{i,t}
\]

以及：

\[
FreeCash_t = Cash_t - ReservedCash_t \ge 0
\]

`ReservedCash` 是现金子账，不重复计入 NAV。每个交易日收盘后运行恒等式检查，容差为 NAV 的 \(10^{-10}\)；失败记 `LEDGER_INVALID`，该次运行不得产生正式判词。

必须保存订单状态机：

    CREATED → RESERVED → SUBMITTED
      → FILLED / PARTIALLY_FILLED / REJECTED / EXPIRED / CANCELLED / CANCELLED_UPGRADE
    PARTIALLY_FILLED → (RemainingTargetValue 重试) SUBMITTED / EXPIRED

每位证券持仓子账必须含：`SettledShares`、`UnsettledBuyShares`、`SellableQty`、`PendingSellReserved`、`AvgCost`、`trade_lots`、FIFO `tax_lots`。母订单必须含：`ParentOrderTargetValue`、`RemainingTargetValue`、`CommissionReserve`、`signal_id`、`parent_order_id`。

### 15.3.1 Lot 会计唯一口径（v12 Closure 4）

三类对象同源（每次 fill 生成），职责不得混淆：

|对象|用途|规则|
|---|---|---|
|`trade_lots`|成交数量、\(P^{ex}\)、费用、结算日、T+1 / SellableQty|每笔非零成交一条；卖出按结算可用库存扣减|
|`AvgCost`|组合 PnL / \(R^{net}\) 的卖出成本基础|**移动平均**：买更新 \(\mathrm{AvgCost}=(旧市值+新净成本)/(旧股+新股)\)；卖按 AvgCost 结转成本，**禁止**用 FIFO 作为 \(R^{net}\) 成本|
|`tax_lots`|分红税持有期限与公司行动税务|FIFO；字段 `{lot_id, shares, cost_basis, acquire_date, settle_state}`|

**分红税（v12）：** 税率 = `tax_rate(holding_days, board, tax_rule_version)`，PIT 取 `payment_date` 当日可知版本。缺省在 `payment_date` 开盘前按 lot **预扣**税后现金入账；仅当税表显式要求“卖出时追缴”时才在卖出日调整 Cash，并登记 `DIV_TAX_CLAWBACK`。禁止用单一 AvgCost 替代 tax_lots 算税。

**零碎股（v12）：** 送转/拆合后允许非 100 股整数记账；卖出数量对 100 向下取整；不可卖残留按最新收盘价计入 NAV；仅当公司行动事实表现金兑付或再次行动使其合并为整手时清除。

每次转移保存时间、原因、前后现金、预留、仓位、可卖库存与槽位快照哈希。

## 15.5 公司行动与证券身份账本（v12）

正式交易账本必须消费 PIT `corporate_actions` 事实表（字段：`security_id`、`effective_date`、`ex_date`、`record_date`、`payment_date`、`action_type`、`cash_amount_per_share`、`stock_ratio`、`split_ratio`、`rights_price`、`tax_rule_version`、`successor_security_id`、`delisting_cash_value`、`available_at`）。研究复权价（§4 DG07）**不得**替代本账本。

**生效时钟（v11 唯一）：**

1. 股数类行动（送转、拆合股、配股登记、代码变更）在 `max(ex_date 或 effective_date, available_at 对应的下一交易日开盘)` 的**开盘前**过账，使当日交易使用调整后股数、前收与涨跌停；
2. 现金分红：除息日仅调整成本基础（按 `tax_rule_version`）；`Cash += 税后股息` **仅在 `payment_date` 开盘前**入账；无 `payment_date` 的记录 → `INVALID_DATA`，不得用 ex_date 替代；
3. 迟到（`available_at` 晚于本应生效开盘）的行动于 `available_at` 后首个交易日开盘前补过账，并登记 `CORP_ACTION_LATE`；不得在盘中或收盘后改变当日已发生成交的股数基准。

处置规则（唯一）：

|action_type|账本效果|
|---|---|
|CASH_DIVIDEND|除息日：按 tax lot（下款）调整成本基础；`payment_date` 开盘前：`AvailableCash +=` 各 lot 税后现金；未结算股按制度是否分红见税表，缺省不分红|
|STOCK_DIVIDEND / TRANSFER|除权开盘前：`SettledShares`、`UnsettledBuyShares` 按 `stock_ratio` 增加；成本基础每股下调使总成本不变；产生的零碎股见下款|
|SPLIT / REVERSE_SPLIT|生效开盘前：股数 × `split_ratio`；成本基础按倒数调整|
|RIGHTS_ISSUE|**默认政策 = NEVER_EXERCISE（永不行权）**：仅登记权利，不扣现金、不增股份；禁止实现为“有现金就行权”或“按理论价值行权”，除非未来 SPEC 显式改写本默认|
|CODE_CHANGE / ABSORPTION|开盘前映射至 `successor_security_id`；持仓与成本整体迁移|
|DELISTING / LONG_SUSPENSION_EXIT|按 `delisting_cash_value`（可为0）变现入 Cash，股数清零；登记 `POSITION_CLOSED_CORP_ACTION`|

**分红税与零碎股：** 唯一口径见 §15.3.1（v12）。CASH_DIVIDEND 的税后每股现金 = `cash_amount_per_share × (1 - tax_rate(lot, tax_rule_version, payment_date))`。

DG16：公司行动漏过账或破坏 NAV 恒等式 → `LEDGER_INVALID`。

## 15.4 市场状态层（Gate S 可选叠加层）

市场情绪、全市场成交和小盘相对强弱定位为**可选叠加层（overlay）候选**，与退出系统同级、在入口独立通过之后评估（Gate S，§18）：

- 预注册三个候选叠加规则：市场状态差时总仓位上限 60%→30%；状态差时暂停新入场；状态极差时收紧退出（Leg B2 的 IDS 回撤门槛 0.20→0.15）；
- 市场状态变量定义：全A等权指数 20 日收益 < 其 250 日 20% 分位，且全市场成交额 20 日均值 < 其 250 日 30% 分位；
- Gate S 通过前，叠加层只作分层报告变量，不进入任何生产决策；
- Gate S 未通过，则生产系统永久无叠加层，仅保留报告。

---

# 16. 验证阶段与正式退出

## 16.1 入口验证阶段

在H-DISC至H-INT验证期间：

    从实际成交日 τ_entry 起固定持有20个交易日
    第20个持有交易日收盘产生退出信号
    下一交易日14:30—15:00 按 §14.3 流式参与执行

同时报告5、10、40、60日结果，但20日是唯一主终点。所有期限均从 \(\tau_{entry}\) 对齐。

## 16.2 正式交易多层退出

只有入口和交互价值通过后启用。

### Leg X：题材崩溃退出（优先级最高）

持仓 episode 状态机进入 COLLAPSE（§12.8 任一分支）：

    该 episode 全部持仓退出
    无持仓期限制（入场次日即生效至平仓）

### Leg I：载体失败

题材仍未Collapse，但个股连续2日满足：

    3日累计相对题材收益 <= -4%
    CCS_level < 0.35
    TurnZ < 0

只退出该股票。

### Leg B1：集中高潮减仓

首次进入CONCENTRATED_CLIMAX：

    仓位减半
    仅触发一次

### Leg B2：顶部/退潮确认

    IDS^θ <= episode内最高 IDS^θ - 0.20
    AND GCCSlope^θ < 0
    AND 题材20日相对宽基收益 < 0

余仓全部退出。

### Leg C：论点过期

持有满60个交易日：

    全部退出

优先级：

    X > I > B2 > B1 > C

v8 删除原 Leg H：正式退出腿中不存在“仅因龙头走弱而卖出”的规则，故“健康交接不减仓”为无操作陈述，保留易误导实现者。

---

# 17. 必须先完成的机制研究

在正式组合回测前，先做事件研究。未经机制闸门，不得直接跑端到端策略并解释市场故事。

**通用观测规则：**

1. **episode 等权：** 除非某 Gate 显式声明其他口径，全部事件研究先在 episode 内聚合成一个数，再对 episode 等权求效应；禁止股票池化改变隐含权重；
2. **合并删失（唯一规则，A-07/层级2）：** 若 episode 在某 outcome 的固定未来窗（D+1:D+5 或 D+1:D+20 等）内发生合并或被合并，该条 outcome 观测**整条剔除**并登记 `CENSORED_BY_MERGE`；不得部分窗使用、不得跨合并拼接；组合账务不受影响；
3. 窗内 NON_EVALUABLE 缺失处理按各指标定义（如 §17.1 AUC 规则）。

## 17.1 Study 1：发现—确认分离研究

### 样本

每个D−1已经存在、且满足 §8.3 共同支持的题材候选。

### 处理组（题材级；v7 A-02）

**D 日满足题材级确认（§11.4 \(\tau_{confirm}\) 三条件）的题材日。** 候选级确认（\(IDS^{-i}\) 等）只是股票入场资格，不定义处理组——同题材内候选级结果不一致不产生歧义。

### 控制组生成器（真实对照完整匹配算法；v8 补 Control Measurement Cohort）

- **风险集：** 同日 D 处于共同支持域、D−1 已存在、未满足题材级确认三条件、且与任何处理题材及进行中 episode 的 Jaccard < 0.60 的真实题材；
- **匹配时点：** D 日收盘；**协变量只允许 D−1 及以前**（禁止任何 D 日确认变量，防 post-treatment matching）：成员数、交易板块构成（各板块占比）、log 成员市值中位数、log 成员 ADV20 中位数、前 20 日题材等权累计收益、前 20 日等权收益波动率、前 20 日成员 Active 率；
- **距离与算法：** 7 维马氏距离（当日全体风险集题材协方差，对角加 \(10^{-6}\)），对每个处理题材取最近 1 个，处理题材按 theme_id 字典序依次匹配，**同日无放回**；
- **控制复用（v9）：** `Gate2_ControlRegistry` 独立维护；同一控制题材在 Gate 2 正式样本中至多使用一次（跨日亦不可复用）。Gate 7 使用独立 registry（§18），互不抢占；
- **卡尺：** 马氏距离 ≤ VALIDATION 区全部配对距离经验分布的 90% 分位（冻结）；超出 → 该处理题材记 `UNMATCHED` 剔除，剔除率入报告；
- **平衡审计（v8）：** 在**全部成功配对样本**上计算各协变量 SMD（等权口径，公式同 §8.3.5）。若任一 |SMD|>0.25：整次 Gate 2 匹配样本记 `BALANCE_FAIL`，不得开庭；禁止以“删除单个配对”的非唯一规则局部修补。允许的唯一补救是放宽前预先注册的卡尺敏感性（85%分位）并作为新 SPEC；
- **Control Measurement Cohort（v8，关闭 B-02）：** 匹配日 D 为每个成功控制题材一次性冻结研究用 `B^{C}, L^{C}, Null^{C}`（Null 按 §8.3 同日算法生成并冻结；L/K 按 §7.3/§9.1 同构临时识别后冻结）。D+1:D+5 的 Whipsaw / IDS 测量**只使用**该冻结测量制度，禁止控制侧继续每日 DAILY_NULL + 动态 Base/L；
- **依赖：** 配对差为观测单位，聚类按 §20.2。

### 主指标（AUC 唯一定义）

全文"未来5日 X AUC"唯一定义为**离散求和**：

\[
AUC_5(X)=\sum_{t=1}^{5}X_{D+t}
\]

不含 D 日、不插值；窗内某日该题材为 NON_EVALUABLE 或停牌覆盖导致 X 缺失 → 该日缺失；缺失 >1 日整条观测剔除并登记，缺失 ≤1 日按剩余日求和乘 5/有效日数标准化。

- 未来5日非龙头 NewJoin AUC（\(X=NewJoinRate(N^{\theta})\)）；
- 未来5日 \(IDS^{\theta}\) AUC；
- 未来5日假突破率；
- 未来20日可交易容量篮子净收益。

### 假突破

\[
Whipsaw5
=
\mathbf 1(
\text{未来5日内连续2日 }IDS^{\theta}<0.30
)
\]

### 通过条件

- **Gate 1 / H-DISC：** 真实路径使用 `DISCOVERY_RESEARCH_COHORT_D`（§8.3.3-1c）冻结的 Base/L/分母，计算未来5日 NewJoin AUC；减去其 `DISCOVERY_RESEARCH_NULL_D`（§8.3.3-1b）冻结伪题材路径 AUC；真实与 null **必须**同属 D 日冻结测量制度，按 Gate 1 MES/CI 裁决；
- **Gate 2 / H-CONF：** 已确认处理组与同日未确认匹配题材的 Whipsaw 率差（控制−处理）按 Gate 2 MES/CI 裁决，MES=10个百分点；
- 未来5日 \(IDS^\theta\) AUC 为 Gate 1/2 的强制次要机制读数，不替代上述唯一主 estimand；
- LOO后仍成立；
- Track S和Track B分别报告。

## 17.2 Study 2：角色可分离性研究（H-ROLE）

本研究独立回答：注意力角色 \(L\) 与容量角色 \(K\) 是否具有不同的未来市场功能，而不是同一涨幅排序的两个名称。为避免定义自证，所有主 outcome 只使用 D+1:D+5 数据，不使用构造 D 日角色的变量值。

### 个股未来功能分数

对 \(i\in L^E\cup K^E\)，每个分量均对 §8.3 匹配伪题材中**全部角色成员池** \(L^E_c\cup K^E_c\)（所有成功伪题材 c 的并集，按成员出现次数加权 midrank，§6.7）取分位——**不**按“同为 L / 同为 K / 同市值层”分子池；真实 L 与真实 K 使用同一参考池，以保证 contrast 可比。

\[
FutureAttention_i
=
Median\left(
Pctl_{null}(ActiveDays5_i),
Pctl_{null}(TouchLimitDays5_i),
Pctl_{null}(MaxCumResidualReturn5_i)
\right)
\]

\[
FutureCapacity_i
=
Median\left(
Pctl_{null}(CumDirectedDemand5_i),
Pctl_{null}(PositiveDemandDays5_i),
Pctl_{null}(MorningExecutableValueShare5_i)
\right)
\]

其中：

- `ActiveDays5`：D+1:D+5 的 Active 日数；
- `TouchLimitDays5`：D+1:D+5 触及涨停日数；
- `MaxCumResidualReturn5`：从 D+1 起累计行业残差收益的5日内最大值；
- `CumDirectedDemand5`：§10.1 DirectedDemand 的5日和；
- `PositiveDemandDays5`：DirectedDemand>0 的日数；
- `MorningExecutableValueShare5`（v9 流式公式）：对 \(t=D+1,\ldots,D+5\)，令研究目标股数 \(Q\)（机制口径：\(0.10\times\) 研究隔离 NAV / 开盘价，向下取整到 100 股；与生产 TargetValue 脱钩）。在 09:35–10:00 按时间顺序流式更新剩余量：
  \[
  fill_m=\mathbf{1}(Low_m<up\_limit)\cdot\min(0.05\cdot vol_m,\ remaining_{m}),\quad
  remaining_{m+1}=remaining_m-fill_m
  \]
  初值 \(remaining_{09:35}=Q\)。则
  \[
  MES5_{i,t}
  =
  \frac{\sum_m fill_m\cdot VWAP_m}{ADV20_{i,D}}
  \]
  全日停牌日该项为 0。禁止对每分钟独立使用 \(\min(0.05 vol_m,Q)\) 而不递减剩余量。`MorningExecutableValueShare5_i=\sum_t MES5_{i,t}`。

### 两个共同主 estimand（episode 等权）

对每个 episode e 先计算 episode 内 contrast：

\[
\theta^{ROLE}_{A,e}
=
Mean_{i\in L^E_e}(FutureAttention_i)
-
Mean_{i\in K^E_e}(FutureAttention_i)
\]

\[
\theta^{ROLE}_{C,e}
=
Mean_{i\in K^E_e}(FutureCapacity_i)
-
Mean_{i\in L^E_e}(FutureCapacity_i)
\]

正式 estimand 为 episode 等权均值：

\[
\theta^{ROLE}_{A}=Mean_e(\theta^{ROLE}_{A,e}),
\qquad
\theta^{ROLE}_{C}=Mean_e(\theta^{ROLE}_{C,e})
\]

**禁止股票池化**（各 episode 的 \(|K^E|\) 不同会改变隐含权重）。窗内合并按 §17 通用规则整条剔除。

### 判读

- Gate 3 PASS：两个 estimand 的单侧 \(1-\alpha_v\) CI 下界均 > 0.05，复合 p 值按 §18.0 IUT 定义；
- Gate 3 FAIL：任一 estimand 的 Bonferroni 单侧 \(1-0.05/(9\cdot m)\)（m=2，即 1−0.05/18）上界 < 0；
- 其余为 INCONCLUSIVE；
- **P5 角色随机化（v8 地位）：** 强制报告的稳健性 guardrail，**不是** Gate 3 共同主 estimand，不进入 IUT。协议：以 \(Seed=Hash(spec\_version,"P5",episode\_id,rep)\)（rep=1..5000，§20.2.3）对 \(L^E\cup K^E\) 做标签置换 5,000 次；若真实两个 contrast 未同时优于其置换分布的 97.5% 分位，而正式 IUT 为 PASS，则 Gate 3 降级为 INCONCLUSIVE；
- 至少60%的自然年度两个 contrast 同时为正，否则即使统计 PASS 也降级为 INCONCLUSIVE；
- H-ROLE 失败不自动否定发现/确认，但阻止 H-MIG、容量时钟和正式交易继续开庭。

## 17.3 Study 3：角色迁移事件研究（H-MIG）

### 事件时间

\[
t=0=\tau_{confirm}
\]

观察：

\[
t=-10,\ldots,+20
\]

### 输出路径

- \(LDS^E\)；
- \(CDS^E\)；
- Leader与Capacity \(PriceShare^E\)；
- \(RMI^E\)；
- \(CapacityBreadth^E\)；
- \(GCC^{\theta}\)（level 尺度）；
- 注意力组和容量组累计异常收益；
- 注意力组和容量组异常成交。

### 通过条件（相对匹配 null）

对每个真实 episode 的匹配伪题材运行同构题材级容量时钟，定义：

\[
\Delta StartRate_5
=
P(\text{真实题材确认后5日内 }GCC^{\theta}\text{ 启动})
-
P(\text{匹配伪题材对应5日内启动})
\]

\[
\Delta CDS_5
=
\left(CDS^E_{D+5}-CDS^E_D\right)_{real}
-
\left(CDS^E_{D+5}-CDS^E_D\right)_{null}
\]

\[
\Delta PriceShare_5
=
\left[(PriceShare^E_K-PriceShare^E_L)_{D+5}
-(PriceShare^E_K-PriceShare^E_L)_D\right]_{real}
-
\left[(PriceShare^E_K-PriceShare^E_L)_{D+5}
-(PriceShare^E_K-PriceShare^E_L)_D\right]_{null}
\]

- Gate 4 的三个共同主 estimand 为 \(\Delta StartRate_5,\Delta CDS_5,\Delta PriceShare_5\)；
- MES_pass 分别为 0.10、0.05、0.05；MES_fail 均为0；
- 三者按 §18.0 IUT 全部通过才 PASS；任一有反向证据才 FAIL；
- **伪题材 GCC 参考池（v8 唯一）：** 计算匹配伪题材 c 的 CCS_level / GCC 时，其 null 分位参照为同 EPISODE_NULL 池中其余伪题材（leave-one-out，\(n_c-1\)）；禁止使用原真实题材 pooled 分布，禁止为每个伪题材再生成第二层 null，禁止 P10 五折 cross-fit 替代本条；
- **附加读数（稳健性 guardrail，非 IUT）：** （i）容量组未来20日净异常收益 − 容量匹配随机组；（ii）剔除最大市值股与最高成交额股后三个主 estimand 方向。二者必须强制报告；若正式 IUT PASS 但任一 guardrail 方向为负，Gate 4 降级为 INCONCLUSIVE；guardrail 不单独产生 FAIL。

## 17.4 Study 4：容量时钟事件研究

仅在已确认题材中，将候选容量股按容量时钟与 CCS_level 分组。

### 主检验（闭合回归对象）

**观测单位：** 候选股 × episode（\(i\in K^E_e\)）。**因变量唯一定义：**

\[
GrossAlpha20_{i,e}
=
R^{gross}_{i,\tau^{h}:\tau^{h}+19}
-
\frac{1}{|C^{valid}_e|}\sum_{c}R^{gross}_{basket(c)}
\]

其中 \(\tau^h=\tau_{confirm}+1\)，买入价 = \(\tau^h\) 日 09:35–10:00 全窗 VWAP 的**机制口径假想成交**（仅同构适用停牌/一字涨停剔除，剔除观测登记；不含 REJECT_GAP、参与率与成本——执行与成本的可实现性由 Gate 8 独立裁决）；对照为该 episode 冻结 null 篮子同构假想收益。机制口径与交易口径（§0.3）不得混用。

**预测变量测量日（v9 唯一）：** 回归中的 \(CCS\_level_i\)、\(GCC^{-i}\)、全部连续 Controls 一律取 **predictor_date = \(\tau_{confirm}\)**（确认日收盘可知值）。禁止使用主信号日、容量启动日或 \(\tau_{confirm}\) 之后任何日期的 CCS/GCC 作为预测变量。Outcome 窗自 \(\tau^h=\tau_{confirm}+1\) 起，保证严格前瞻。

**主时钟使用候选剔除版 \(GCC^{-i}\)（容量组仅3只时候选可经 \(GCC^{\theta}\) 抬高自身题材时钟）：**

\[
GrossAlpha20_{i,e}
=
\alpha
+\beta_1 \widetilde{CCS\_level_i}
+\beta_2 \widetilde{GCC^{-i}}
+\beta_3 \widetilde{CCS\_level_i}\times \widetilde{GCC^{-i}}
+\gamma'\widetilde{Controls}
+\epsilon
\]

\(\widetilde{\cdot}\) 表示按 VALIDATION 区冻结均值/标准差标准化；**CCS_level、GCC 与全部连续 Controls 一律标准化**，交互项为标准化后乘积。\(GCC^{\theta}\) 版本为强制诊断回归，不进入正式判词。

**Controls 冻结清单（全部取值日 = \(\tau_{confirm}\)）：**

    log(FreeFloatMV_{τ_confirm})
    log(ADV20_{τ_confirm})
    CumReturn20_{τ_confirm-20:τ_confirm-1}
    Volatility20_{τ_confirm-20:τ_confirm-1}
    TurnValueMedian20_{τ_confirm-20:τ_confirm-1}
    BoardFixedEffects
    CalendarMonthFixedEffects

禁止增删控制项；任何修改即新 SPEC。

**量纲与 MES：** \(\beta_3\) 单位 = "每 1σ×1σ 的 20 日毛超额收益"；MES_pass = 0.005（0.5%，登记先验），MES_fail = 0。观测等权，依赖由题材簇×自然月双向聚类吸收（§20.2）。

### 通过条件与法庭分层（v10）

- **共同主 estimand（唯一正式 IUT）：** \(\beta_3\)（\(\widetilde{CCS}\times\widetilde{GCC^{-i}}\)，predictor=\(\tau_{confirm}\)）按 Gate 5 MES/CI 与 §18.0 裁决；
- **稳健性 guardrail（非 IUT）：** （i）CCS_level 顶四分位毛超额 − 容量匹配随机组 > 0；（ii）四分位毛超额对 CCS_level 分位非严格单调不减；（iii）在 \(\tau_{confirm}\) 日 \(GCCSlope^{\theta}>0\) 的子样本上 \(\beta_3\) 点估计 > 0（验证生产“仍在上升”过滤）。若正式 IUT PASS 但任一 guardrail 失败 → INCONCLUSIVE；guardrail 不单独 FAIL；
- **强制诊断：** \(GCC^{\theta}\) 版本回归交互项方向。

## 17.5 Study 5：买家结构诊断（EXPLORATORY）

容量时钟的叙事是“边际需求从注意力载体向容量载体迁移”。该叙事需要至少一个方向一致的观测锚，否则“容量”只是市值/流动性的同义反复。龙虎榜数据仅允许进入本研究，全部结果标记 `EXPLORATORY`，不进入 Holm 家族，不得升级为信号。

诊断项：

1. 确认后 5 日内容量组龙虎榜出现率 vs 匹配伪题材容量组；
2. 容量组上榜席位的机构专用席位占比 vs 注意力核心组；
3. 买方席位集中度（HHI）在 \(\tau_{confirm}\) 前后的路径；
4. 上榜后 5 日容量组的净收益与未上榜容量组对比。

判读纪律：本研究只回答“容量需求的大资金叙事方向是否一致”，不一致不单独否决 Gate 4/5，一致也不单独构成通过证据。

## 17.6 走廊测量

以下读数列入全部 Gate 报告的强制项：

- **MFE / MAE：** 每笔入场在持有期内的最大有利/不利偏移（对匹配对照同期收益标准化）；
- **达峰时间分布：** MFE 出现日相对 \(\tau_{entry}\) 的分布（中位数与四分位）；
- **可实现捕获率：** 实际净收益 / MFE，按退出腿触发路径分层；
- **反事实走廊：** 固定 5/10/20/40/60 日持有的 MFE/MAE 对照表。

这些读数是 Leg B1/B2/X 参数与 H-EXIT 判词的直接输入，不得只出现在附录。

---

# 18. 逐层验证 Gate 与三态判词

## 18.0 通用三态决策规则

每个 Gate 的 Population、Estimand、Null、MES、Sample、Dependence、Multiplicity、Sequential、Decision、Failure state 十项见附录 B。方向统一为“数值越大越好”的效果量 \(\theta_g\)：

- **版本 α-wealth：** SFL-1 项目全生命周期的正式 PASS 预算按解封 HOLDOUT 的版本序号 k 分配：\(\alpha_v(k)=0.05\times2^{-k}\)（\(\sum_{k=1}^{\infty}\alpha_v(k)=0.05\)，故项目级错误 PASS 概率 ≤5%）。**v10 登记 k=1，\(\alpha_v=0.025\)**；v1–v9 均未解封 HOLDOUT，不消耗预算（登记于触碰日志）。后续任何新 SPEC 版本自动取下一序号，禁止重置；
- **固定顺序：** Gate 1→2→…→9；Gate g 仅在全部前序 Gate PASS 后开庭。固定顺序 gatekeeping 在每 Gate \(\alpha=\alpha_v\) 下控制该版本核心家族强 FWER≤\(\alpha_v\)，项目级"任一版本错误 PASS"概率 ≤5%；
- **单一 estimand Gate：** \(p_g\) 为检验 \(H_0:\theta_g\le MES^{pass}_g\) 的单侧 p 值；
- **复合 Gate：** 对 m 个共同主 estimand 分别得到 \(p_{g,k}\)，使用 intersection-union test：\(p_g=\max_k p_{g,k}\)。Gate 3 的 m=2；Gate 4 的 m=3；Gate 8 的 m=2；Gate 9 的 m=2；
- **PASS：** \(p_g<\alpha_v\)，等价地所有共同主 estimand 的单侧 \(1-\alpha_v\) 置信下界均高于各自 \(MES^{pass}\)；
- **FAIL 与版本级 false-death 预算：** 对反向命题 \(\theta_{g,k}<MES^{fail}_{g,k}\)，每个 elementary 反向检验使用单侧置信度 \(1-0.05/(9\cdot m_g)\) 的 Bonferroni 同时上界；任一共同主 estimand 的该上界低于其 \(MES^{fail}\) 才 FAIL。由此**单版本内**"至少一个真机制被错误 FAIL"的概率 ≤5%（独立于 PASS 预算）。**v8 明确：不建立跨版本 β-wealth**；多版本依次运行时项目生命周期错误关线概率可累积，false-death 只作为版本级治理属性披露，不得宣称项目级错误关线率受控；
- **INCONCLUSIVE：** 其余全部情形（含 \(L_g\le MES^{pass}\) 且 \(U_g\ge MES^{fail}\)）。

\(MES^{fail}_g \le MES^{pass}_g\)。FAIL 表示“证据支持效果低于最低可接受区间”，不再以“不满足 PASS”代替。HOLDOUT 上 Gate 1–8 INCONCLUSIVE → 终局 `INSUFFICIENT_EVIDENCE`；Gate 9 与 Gate S 的特殊终局见 §22。样本/装置失败同理。

## Gate 0：数据、PIT 与生成器

通过§4全部闸门（含 DG13–15）及 §8.3 NG01–NG07。

失败：

    INVALID_DATA

## Gate 1：题材发现有效

真实候选题材的后续独立扩散高于匹配伪题材。按附录 B 三态裁决。

FAIL：

    FAIL_DISCOVERY_UNIT

只否定当前发现方法，不永久否定所有题材思想。

## Gate 2：独立确认有效

确认降低假突破并提高未来扩散；主效果为假突破率差（控制−处理），MES_pass = 10个百分点。按附录 B 三态裁决。

FAIL：

    FAIL_CONFIRMATION

## Gate 3：角色可分离

按 §17.2 的两个未来功能 contrast 独立检验 H-ROLE；不得使用构造角色的 D 日变量作为主 outcome。两个 contrast 均达到 MES 才 PASS。按附录 B 三态裁决。

FAIL：

    FAIL_ROLE_SEPARATION

## Gate 4：角色迁移存在

容量需求份额与容量相对正向价格贡献在确认后相对匹配 null 上升；共同主 estimand 为 \(\Delta StartRate_5,\Delta CDS_5,\Delta PriceShare_5\)。CapacityBreadth 路径为强制诊断，不进入正式 Gate 4 IUT。按附录 B 三态裁决。

FAIL：

    FAIL_ROLE_MIGRATION

## Gate 5：容量时钟有效

\(GCC^{-i}\) 和 CCS_level（均冻结于 \(\tau_{confirm}\)）的标准化交互能够预测确认后 20 日机制口径毛超额（§17.4）；正式 IUT 仅 \(\beta_3\)；顶组与单调性为 guardrail（§17.4）；必须同时报告启动带 [0.55, 0.85] 敏感性与 \(GCC^{\theta}\) 诊断回归。按附录 B 三态裁决。

FAIL：

    FAIL_CAPACITY_CLOCK

## Gate 6：容量载体选择特异性

同一题材、同一入场日的四臂比较。

**共同支持域（v9）：** 令 `EligibleProductionCandidates` = \(K^{trade}\) 中通过 §13.1 全部入场条件（CONF/GCC/CCS/Persist/未过度扩张等）的股票集合；`NonSelectedEligibleCandidates` = Eligible 中去掉 EntryScore 最高的 min(2,|Eligible|) 只之后的剩余。正式 Gate 6 要求：

\[
|EligibleProductionCandidates|\ge 2
\quad\text{且}\quad
|NonSelectedEligibleCandidates|\ge 2
\]

不满足者登记 `OUT_OF_SUPPORT_G6`，不进入正式判词。仅 \(|K^{trade}|\ge4\) 但 Eligible<2 不得外推。

**四臂统一净执行假想账本（v10 资本规则）：**

- 每臂总资本 = Eligible 中 EntryScore 前 2 的理论 TargetValue 之和（两只均按 §15.1 在隔离研究 NAV 上计算；不依赖“真实只成交1只”的特判）；四臂总资本相同；
- 臂内成员等权拆分人民币；未成交部分为现金、期内收益 0；
- 2 名臂：至少 1 只非零成交则该臂有效；全容量臂：至少 1 只非零成交有效；无效臂的 episode 整条剔除并登记；
- 按 §14.1 完整执行语义假想成交，非生产组合账本。

臂定义：

1. **EntryScore 前 2：** 在 EligibleProductionCandidates 内按 §13.2 取前 2；
2. **MatchedRandom 2 只（主对照，唯一生成器）：** 仅在 `NonSelectedEligibleCandidates` 中匹配抽样（检验 EntryScore 排序特异性，而非资格闸门本身）。对前 2 逐一：以（log自由流通市值, log ADV20）VALIDATION 标准化欧氏距离取最近 5 只，`Seed=Hash(spec_version,"G6",theme_id,D,k)`，等概率抽 1，无放回；候选不足 5 取全部；
3. **全部 Eligible 等权：** EligibleProductionCandidates 全体（总资本同臂 1）；
4. **题材内 RS 前 2：** 在 Eligible 内按 \(RS_i=\sum_{t=D-19}^{D}x_{i,t}\) 取最高 2 只。

**强制诊断对照（非主 Gate）：** `MatchedRandom_AllK` = 从 \(K^{trade}\setminus\){EntryScore 前2} 抽样（可含未过资格闸门者），仅报告，用于分解“资格闸门 vs 排序”贡献。

**臂收益口径 \(R^{arm}_{net}\)（v10 唯一正式）：**

1. 买入：按 §14.1 人民币母订单 + 分钟流式参与（含 REJECT_*、部分成交、跨日 RemainingTargetValue 补单，有效期同生产 D+1:D+3）；
2. 持有：自该臂首次非零成交日 \(\tau_{entry}\) 起 20 个持有交易日；
3. 卖出：第 20 持有日收盘产生退出，下一交易日按 §14.3 流式卖出（含顺延与 T+1 SellableQty）；
4. 成本：§14.4 全部费用、固定滑点与冲击；未成交部分为现金、期内收益 0；
5. \(R^{arm}_{net}\) = 臂净值相对其配置总资本的净收益。

机制毛口径（不计成本、固定窗口收盘卖出）仅作诊断，标记 `DIAGNOSTIC_GROSS_ARM`，不进入 Gate 6/7 正式判词。

主统计：

\[
Alpha20_{EntryScore}
=
R^{arm1}_{net}
-
R^{arm2}_{net}
\]

（臂 3、4 为强制对照报告。）每 episode 一个观测、episode 等权，依赖按 §20.2。成分归因中的“净收益变化”必须回引本 \(R^{arm}_{net}\)。

**成分归因（预注册，强制报告）：** 对 EntryScore 四个成分做留一消融，报告净收益变化；并报告 Shapley 分解。若某成分边际贡献的 95% CI 不含正数，登记 `SIMPLIFICATION_CANDIDATE`，触发规格修订讨论；不允许在评估样本中直接替换生产权重。

诊断对照：CCS_rank 单成分前 2。

FAIL：

    FAIL_CARRIER_SELECTION

## Gate 7：过滤交互价值

2×2实验：

| |EntryScore选择|容量匹配随机|
|---|---:|---:|
|确认+容量启动题材|A|B|
|同日匹配控制题材|C|D|

**Gate 7 独立生成器（v10；禁止回引 §17.1 / Gate 2）：**

- **事件时点 S：** episode 唯一主信号日 = 该 episode 首次开主信号窗之日（§13.1）；
- **处理组：** S 日首次开主信号窗的真实 episode；
- **独立控制登记簿：** `Gate7_ControlRegistry` 与 `Gate2_ControlRegistry` **完全分离**；“已使用”仅指控制身份；
- **控制风险集：** S−1 已存在、已确认（\(\tau_{confirm}\le S-1\)）、满足 **NoCapacityStartThroughS**（截至 S 日历史上**从未**进入 {EARLY_CAPACITY, HEALTHY_TRANSFER}，不是“S 日当前不属于”）、与处理题材 Jaccard < 0.60、与其他进行中 episode \(e\neq\)候选控制 Jaccard < 0.60、且不在 `Gate7_ControlRegistry`；
- **匹配顺序：** 交易日升序，日内处理 theme_id 字典序；成功配对写入 registry；
- **匹配变量截止 = S−1**（防 post-treatment matching）；马氏距离、卡尺同 §17.1；
- **控制角色：** 使用控制自身原有 \(B^E/L^E/K^E\) 与 EPISODE_NULL；禁止 Temporary Cohort；
- **四臂信号日期（v10 关键）：** 配对完成后，A/B/C/D **全部**使用 **S 日收盘可知** 的 CONF/CCS/GCC/RMI/EntryScore（处理与控制同尺同日），假想执行统一在 **S+1**；禁止把匹配信息截止误写成控制策略信号截止；
- **共同支持：** 双方均满足 Gate 6 Eligible≥2 且 NonSelectedEligible≥2；
- **列与账本：** 四格使用 Gate 6 的 \(R^{arm}_{net}\) 假想账本（§18 Gate 6），不占用生产资金/槽位。

\[
\Delta_{INT}
=
(A-B)-(C-D)
\]

每"处理-控制配对"一个观测、配对等权。按附录 B 三态裁决。FAIL：

    FAIL_FILTER_INTERACTION

## Gate 8：交易可实现

加入分钟执行、涨跌停、未成交和全部成本后，按下列 **两个法庭对象** 报告；不得混成同一 CI。

### Gate 8A：Conditional Execution Court（正式主 estimand）

- **Population_fill：** 有效期内至少一笔非零成交的理论信号；
- **Outcome：** NetAlpha20（§0.3；含冲击单次计入与真实费用）；
- **Null / MES：** 见附录 B；进入 §18.0 IUT 主终局；
- **禁止**把未成交记 0 后并入本样本。

### Gate 8B：Execution Policy Diagnostic（强制政策诊断，非第二主 Gate）

- **Population_all：** 全部理论信号（含未成交）；
- **Outcome：** `PolicyReturn(s)` = 成交则 NetAlpha20，完全未成交则 **现金收益 0**；
- 正式汇总：`PolicyNetAlpha20 =` 信号等权均值；与 FillRate、未成交假想收益同列首页（§20.6）；
- **不得**替代 Gate 8A 主终点，**不得**与 8A 混同一置信区间；标记 `DIAGNOSTIC_POLICY_RETURN`；
- 若 8A PASS 但 FillRate 过低或 PolicyNetAlpha20 显著恶化，登记 `POLICY_IMPLAUSIBLE`（不自动改写 8A 判词，但阻断宣称“整套执行政策已被 8A 证明”）。

### 强制逆向选择对照（隔离研究账户；与 8A/8B 并行）

对每个理论信号 s：

1. 构造隔离研究账户：固定该信号 `ParentOrderTargetValue`；**不**消费生产组合 `AvailableCash` / `GrossRoom` / `ThemeRoom` / 同日排序槽位；
2. `Outcome_actual(s)` = 隔离账户实际规则（含 REJECT_GAP）净结果：成交用 NetAlpha20，未成交为 0；
3. `Outcome_no_gap(s)` = 同一隔离账户仅移除 REJECT_GAP 后的净结果；
4. \(\theta_{ADV,s}=Outcome_{no\_gap}(s)-Outcome_{actual}(s)\)；正式 estimand = 全理论信号等权 \(\theta_{ADV}\)；非劣界 1 个百分点；
5. **禁止**完整生产组合逐信号 rerun 作为主 guard（标记 `DIAGNOSTIC_PORTFOLIO_RERUN`）；
6. guard PASS：\(\theta_{ADV}\) 单侧 \(1-\alpha_v\) 上界 < 0.01；
7. guard FAIL：Bonferroni 单侧 \(1-0.05/(9\cdot m)\)（m=2）下界 > 0.01 → `FAIL_ADVERSE_SELECTION`；
8. 其余 `INCONCLUSIVE_ADVERSE_SELECTION` → Gate 8 整体 INCONCLUSIVE；
9. **Gate 8 IUT 共同主 estimand** =（8A 上 NetAlpha20）与（\(-\theta_{ADV}\)）；8B 不进入 IUT。

FAIL：

    FAIL_EXECUTION

唯一失败码优先级：若逆向选择 guard FAIL → `FAIL_ADVERSE_SELECTION`；否则若 8A NetAlpha20 FAIL → `FAIL_EXECUTION`。任一共同主 estimand INCONCLUSIVE 且无 FAIL 证据时，Gate 8 为 INCONCLUSIVE。

## Gate 9：退出增量

动态退出相对固定20日持有是否改善风险调整后净收益。

**科学问题选择（v8 唯一）：纯退出增量。** 不回答“端到端有限资金政策孰优”。

**外生冻结入场 tape（v10 唯一生成器）：**

1. **Tape 来源：** 隔离研究账户上运行**固定 20 日入口政策**（§16.1）的生产入场成交序列；强制固定退出生成；哈希冻结后方可开庭；
2. **行粒度：** 每个成交分钟/日子成交可多行，但必须带母信号键。每行字段：
   `signal_id, parent_order_id, attempt_date, fill_date, stock, fill_qty, fill_net_cost, origin_episode_id, original_target_value, remaining_target_before, remaining_target_after`
3. **DeployedCapitalBase（v12 冻结唯一资本分母）：**
   - 对 tape 按交易日 t 定义 `ConcurrentGrossNotional_t` = 当日仍持有的全部 tape 仓位按买入净成本计的总名义；
   - **唯一：** `DeployedCapitalBase = max_t ConcurrentGrossNotional_t`；
   - 两条影子组合的初始现金 = `DeployedCapitalBase`；两路径 NAV 由此现金 + tape 持仓构成；
   - \(\Delta CumExcess\) = 动态路径累计暴露匹配超额 − 固定20日路径累计暴露匹配超额；`MES_pass=0.02` 表示该差 ≥ **2 个百分点**（非绝对货币 2%）；
   - **禁止**替代分母：全样本 Σ `original_target_value`、任意外部固定资本、累计部署资本之和（可作诊断）、TWR（可作诊断）；
   - 仍须对每个 `parent_order_id` 只计一次 `original_target_value` 以生成 tape，该和只作诊断；
4. 两条影子组合无条件接受同一 tape，只允许退出不同；
5. **强制 tape 与动态退出冲突（v10）：** 若动态组合在后续追补 fill 之前已对该 signal 清仓，仍必须按 tape 接受后续买入（隔离资本装置）；因此产生的持仓按动态退出规则继续管理；T+1：当日 tape 买入不可用于当日卖出；
6. 不受 §15 簇槽位/60% 硬约束；禁止声称完整有限资金生产政策；端到端对照标记 `DIAGNOSTIC_E2E_POLICY`。

**推断装置：**

1. 构造两条配对隔离组合 NAV 日收益序列（同 tape、仅退出腿不同）；
2. \(Sharpe = \sqrt{250}\times Mean(r^{exc}_t)/Std(r^{exc}_t)\)，\(r^{exc}_t\) 为对暴露匹配基准（§0.3.3）的日超额，Std 为样本标准差（ddof=1）；
3. 两个共同主 estimand：\(\Delta Sharpe\)（动态−固定，MES_pass=0.10）与 \(\Delta CumExcess\)（累计净超额差，MES_pass=0.02）；MES_fail 均为 0；
4. CI 与 p 值：同步 21 交易日 circular block bootstrap，seed = `Hash(spec_version,"G9_BOOT",partition_id)`（§6.9），10,000 次（可升 50,000，§20.2）；
5. 题材簇×月双向聚类仅用于 Gate 9 episode 层诊断读数。

按附录 B 三态裁决。FAIL：

    FAIL_EXIT_INCREMENT

退出失败不推翻入口，可退回固定持有或另立退出项目。

## Gate S：市场状态叠加层

**科学问题（v9）：端到端 overlay 政策比较**——允许入口与退出序列因 overlay 规则而不同；**不是**“同一入口/退出序列”比较。

评估 §15.4 三个预注册叠加规则：各自完整政策（含可能暂停入场、收紧退出、降仓）相对无叠加完整政策的 Sharpe 差 / 最大回撤，Holm 校正后三态裁决；PASS 方可进入生产；FAIL 或 INCONCLUSIVE 均不进入。Gate S 不影响入口与退出主终局。

---

# 19. 对照、安慰剂与零流水线审计

必须全部执行。

## P1：候选股Leave-One-Out

候选股不能进入自己的候选级确认或 \(GCC^{-i}\)。

## P2：去龙头确认

剔除ALS前2后重新计算独立扩散。

## P3：发现日期错位

题材发现时间分别平移：

    -5、-3、+3、+5交易日

对各错位版本重算完整信号链与 NetAlpha20：

|衰减模式|判读|
|---|---|
|仅 \(\tau_0\) 有效，±k 对称衰减|与发现时序的因果一致（PASS 方向）|
|−k 保持有效|发现单元无增量：质疑 Gate 1|
|+k 保持有效|机械趋势/延迟动量代理，质疑确认与容量时钟增量|
|±k 对称缓慢部分保留|与宽时间窗动量一致，需 Gate 7 鉴别题材特异性|

未来错位信号同样有效时，优先怀疑前视或机械趋势。

## P4：成员随机化

保持行业、板块、市值、流动性、拥挤度、注意力基线和成员数量，随机重排成员。

## P5：角色随机化

在每个题材的 \(L^E\cup K^E\) 内保持 \(|L^E|\) 与 \(|K^E|\) 不变，以 \(Seed=Hash(spec\_version,"P5",episode\_id,rep)\)（§20.2.3，rep=1..5000）进行5,000次无放回标签置换；每次重算 \(\theta^{ROLE}_A,\theta^{ROLE}_C\)。真实双 contrast 必须同时优于置换分布 97.5% 分位，作为 Gate 3 的稳健性 guardrail（§17.2）。不得以角色构造资格重新筛选置换标签。

## P6：供应商交叉

可获得第二套PIT题材数据时独立重复。

## P7：板块制度分层

主板、创业板、科创板、北交所分别报告。

## P8：执行压力

- 成本提高50%和100%；
- 成交量参与率减半；
- 所有开盘涨停股票视为未成交；
- 退出延迟一日。

## P9：极端episode剔除

剔除收益最高5%的episode，主结论仍应保持方向。

## P10：完整零流水线审计

### P10_NULL_EPISODE（v12 Identity Protocol）

P10 不得以每日新建 `(c,D)` 当作独立完整题材路径。必须构造稳定伪 episode。

**Pseudo-theme identity key（冻结）：**

    (spec_version, real_theme_id, fold, control_index, spawn_index)

|字段|定义|
|---|---|
|`P10_NULL_EPISODE_ID`|`Hash(spec_version,"P10EPI",real_theme_id,fold,spawn_index)`|
|`discovered_at`|伪题材被纳入发现审计的起始交易日|
|`base_membership`|discovered_at 日对齐矩阵冻结的成员（确认前不得每日重抽）|
|`base_fingerprint`|`ControlFingerprint` at discovered_at|
|确认前 null|固定 reference fold 的统计更新，**membership 不变**|
|`τ_fake_confirm`|链内首次满足 τ_confirm 三条件之日；当日冻结伪角色与伪 EPISODE_NULL|
|`main_signal_used`|每伪 episode 最多一次主信号（布尔）|
|`closed_at` / `close_code`|关闭日与关闭码（含 `CLOSED_SUPERSEDED`）|

**生成 / 新发现 / spawn 算法（v12 唯一）：**

对每个真实可评估题材日 D、每个 cross-fit control `c`（`control_index=c`）：

1. `Members(c,D)` = 该 control 在 D 日对齐矩阵成员集合；`ControlFingerprint(c,D)=Hash(spec_version,"P10FP",real_theme_id,fold,c,sorted(Members))`；
2. 令 `open_e` = 同 `(real_theme_id,fold,c)` 下唯一开放伪 episode（若有；任何时候最多一个开放）；
3. **Continuation（不 spawn）：** 若存在 `open_e` 且 \(Jaccard(Members(c,D), open_e.base_membership)\ge 0.80\)，则推进 `open_e` 状态机；当日不计新分母；
4. **Supersede：** 若存在 `open_e` 且 Jaccard < 0.80，则立即关闭 `open_e`（`close_code=CLOSED_SUPERSEDED`，`closed_at=D`），**然后**进入步骤 5；禁止同一 `(real_theme_id,fold,c)` 并存两个开放伪 episode；
5. **Spawn / respawn：** 若不存在开放伪 episode：若同 `(real_theme_id,fold,c)` 存在上一已关闭伪 episode，则要求 `D ≥ closed_at + 5` 个交易日（cooldown），否则当日不 spawn、不计新分母；通过后 `spawn_index` 在 `(real_theme_id,fold)` 内从 0 严格递增，写入 `discovered_at=D`、`base_membership=Members(c,D)`、`base_fingerprint`；
6. **禁止：** 仅因日历日推进把同一 control 计为新假 episode；跨 `real_theme_id` 合并相似 membership；把每日 `(c,D)` 键当作 episode 计数单位。

将伪 episode 运行发现→确认→角色→容量时钟→主信号完整链（不交易真实资金），估计假 IGNITION / 假确认 / 假主信号及分层分布。

### 假信号率唯一定义

- `P10_EVALUABLE_EPI(e)=1`：伪 episode e 在其生命周期评估窗内 reference≥100、SMD 通过、链无缺失；
- `P10_FAKE_SIGNAL_EPI(e)=1`：e 在其生命周期内产生至少一次主信号（每 episode 最多计 1）。

\[
FalseSignalRate(W)
=
\frac{\sum_{e:\,overlap(W)} P10\_FAKE\_SIGNAL\_EPI(e)}
{\sum_{e:\,overlap(W)} P10\_EVALUABLE\_EPI(e)}
\]

分母为0记 `P10_INSUFFICIENT`。年化假 episode 数：

\[
AnnualFalseEpisodes_y
=
\#\{\,e:\,P10\_FAKE\_SIGNAL\_EPI(e)=1,\ discovered\_at\in y\,\}
\]

按**唯一伪 episode**计数，禁止把每日 recipient 隐含当成新 episode。板块/规模分层规则同前（按 spawn 日真实题材属性）。

年度覆盖：若某年 `discovered_at∈y` 的可评估伪 episode 数 < 该年真实可评估题材日数的对应下限（或有效评估日覆盖 < 该年交易日90%），记 `P10_INSUFFICIENT_YEAR`，不得报告 AnnualFalseEpisodes、不得用于 Phase-I 或漂移。覆盖达标时按唯一伪 episode 计数，并报告未覆盖日数；禁止把每日 `(c,D)` 当成新 episode 求和。

Phase-I 99% 预测上界：在 VALIDATION 日级 `(fake_count,evaluable_count)` 上按自然月做10,000次有放回 block bootstrap；每次计算全部连续28交易日 pooled ratio 的最大值，取这些最大值的99%分位。总体与每个分层（累计 evaluable≥500）分别冻结；FORWARD 任一有效层超过其上界即 `NULL_RATE_DRIFT`。

**绝对假信号预算（v7 新增，B-03：仪器有效性闸门，非漂移闸门）：**

预注册上限（登记先验，§27）：

    FalseSignalRate（VALIDATION 全样本 pooled） <= 5%
    AnnualFalseEpisodes（VALIDATION 各年度）     <= 20 / 年

任一超限 → 判 `INVALID_MEASUREMENT`（按 Gate 0 处置）：确认链在纯 null 下点火过于频繁，测量仪器无效；**不得以真实信号收益结果豁免、不得以"FORWARD 未漂移"豁免**。修复只能通过修订确认定义并创建新 SPEC。该预算与漂移监控相互独立：漂移监控回答"仪器是否变了"，本预算回答"仪器本来是否合格"。

执行规则：

1. 200 个 controls 按 `Hash(theme_id,D,control_index) mod 5` 固定分为5 fold；
2. 将 control c 当作假真实题材时，只使用其他4个 fold（目标约160个 controls）作为其 null reference；禁止使用 c 所在 fold，防止自参照；
3. 复用 §8.3.8 的 StockDayFeatureStore、NullMembership 对齐关系和 PseudoThemeStats，不为每个假真实题材再生成第二层200个 controls；
4. 每个 control 都恰好作为假真实题材一次；五 fold 结果合并前按 control_index 稳定排序；
5. DESIGN 与 VALIDATION 各执行一次，并通过 NG07 的独立二阶 null 精度基准；
6. VALIDATION 输出冻结为 Phase-I null 基线，FORWARD 只监控不重估；
7. FORWARD 28交易日滚动假主信号率超过 Phase-I 同层 99% 预测上界 → `NULL_RATE_DRIFT`，按 §21.6 QUARANTINE；
8. P10 是仪器审计，不进入 Holm 科学假设家族；其失败使测量制度无效，按 Gate 0 处置。

### NG07 二阶 brute-force recipient 规则

仅用于 NG07 基准。将第一层 control c 定义为稳定 recipient：

    recipient_id = "P10|" + real_theme_id + "|" + D + "|" + control_index

其排除集为 §8.3.1 原真实题材排除集并上 c 的全部成员；第二层 control k 的 seed 为 `Hash(spec_version,recipient_id,D,k)`，按 §8.3.2–§8.3.5 从头生成200个 controls，不复用第一层 donor membership。NG07 比较的“信号率差”即上述 FalseSignalRate 的绝对百分点差。

---

# 20. 统计装置

## 20.1 主终点

唯一主终点：

\[
NetAlpha20
\]

episode 层基准为匹配伪题材容量篮子同构执行，组合层基准为暴露匹配全A等权（§0.3）。其他期限和指标为次要或机制指标。

## 20.2 主推断装置与依赖结构（v8 正式算法规范）

**正式主装置：** 题材簇 × 自然月双向聚类稳健标准误，由其生成 Gate 主效果 CI 与 p 值（Gate 9 与 Gate S 使用 §18 声明的组合级 bootstrap 装置，属显式例外）。

### 20.2.1 观测、权重与聚类 ID

- 默认观测单位见各 Gate / 附录 B；除非另有声明，观测等权；
- 聚类 ID1 = **分区内 persistent overlap component（v9）：** 在该正式样本分区（DESIGN/VALIDATION/HOLDOUT）内，若两 theme/episode 在各自风险窗内任一日按 §5.3 规则曾落入同一日连通分量，则合并为同一统计 cluster（并查集，按 theme_id 最小根命名）。禁止仅用信号日当日连通分量导致同故事跨日断开；
- 聚类 ID2 = `YYYY-MM(index_date)`，其中 `index_date` 由附录 B 每 Gate **唯一**指定；禁止对无信号日 Gate 擅自改用候选日/确认日/outcome 起始日的多种实现；
- 配对研究（Gate 2/7）：每个配对一行；处理题材簇进入 ID1；控制题材因禁止复用不再另设第三维；
- 同一题材多只股票不得当独立样本。合并：两 origin episode 于合并日前一日右删失，各为聚类单元；合并后片段不进 episode 科学推断。

### 20.2.2 Two-way cluster covariance（CGM，唯一）

对线性/均值估计量，使用 Cameron–Gelbach–Miller (2011) inclusion-exclusion：

\[
\widehat{V}_{CGM}
=
\widehat{V}_{cl1}
+
\widehat{V}_{cl2}
-
\widehat{V}_{cl1\cap cl2}
\]

其中每一项为对应聚类维的聚类稳健协方差，**有限样本修正固定为 HC1**：乘以 \(\frac{G}{G-1}\cdot\frac{N-1}{N-K}\)（G=该维聚类数，N=观测数，K=回归参数数；对纯均值估计 K=1）。禁止改用 HC0/CR2 作为主装置。

**参考分布与自由度：** 使用 Student-t；自由度 = \(\min(G_1,G_2)-1\)。有效题材簇 < 30 → 该 Gate `INCONCLUSIVE`（与 §20.4 一致）。

**非正定处理：** 若 \(\widehat{V}_{CGM}\) 不正定，投影到最近正定矩阵（特征值负部置 \(10^{-12}\) 后重构），并登记 `COV_PSD_PROJECTED`；投影后主对角仍非正 → INCONCLUSIVE。

### 20.2.3 Bootstrap / 随机化 seed payload（§6.9）

|程序|Payload 字段（按序）|
|---|---|
|题材簇 episode 块 bootstrap|`spec_version,"EPI_BOOT",gate_id,partition_id,rep`|
|Gate 9 circular block bootstrap|`spec_version,"G9_BOOT",partition_id,rep`|
|P10 Phase-I month bootstrap|`spec_version,"P10_BOOT",stratum_id,rep`|
|P5 角色置换|`spec_version,"P5",episode_id,rep`|
|Gate 6 MatchedRandom|`spec_version,"G6",theme_id,D,k`|
|稳健性随机化检验|`spec_version,"RAND",gate_id,partition_id,rep`|

rep 从 1 递增；未列出的正式随机程序必须先修订本表再实现。

### 20.2.4 强制稳健性与冲突规则（v9 算法）

1. **persistent-component 块 bootstrap（v12 唯一正式稳健性单位）：** 以聚类 ID1（persistent overlap component）为**整块**有放回抽样；抽中某 component 则其内全部 episode 一并进入；抽取次数使 episode 总数期望等于原样本。**禁止**逐 episode 独立有放回；**禁止**抽中 component 后再随机丢弃其中部分 episode；**禁止**“先逐 episode bootstrap 再事后贴 cluster SE”作为正式稳健性。seed 见 §20.2.3；5,000 次（可升 50,000）。
2. **匹配集合内随机化检验：** seed=`Hash(spec_version,"RAND",gate_id,partition_id,rep)`。置换对象按 Gate 固定映射：Gate2/7=配对内处理/控制标签；Gate3=P5 角色标签（§17.2）；Gate6=臂1/臂2 标签；其他 Gate 若无配对结构则跳过本项并登记 `RAND_N/A`。真实效应须 ≥ 置换分布 \(1-\alpha_v\) 分位，否则该稳健性失败。
3. **周聚类敏感性：** 将 ID2 由自然月改为 `index_date` 所属日历周，其余同主装置 CGM，重算 CI。

**“方向相反”（v9 唯一）：** 稳健性点估计与主装置点估计符号相反，**且**稳健性效应未超过对应 \(MES^{pass}\)（即稳健性不支持同向 PASS）。仅看 p 值或仅看 CI 不含 0 不足以判定。

主装置与稳健性冲突时标 `INFERENCE_CONFLICT` 并首页披露；**若主装置 PASS 而上述稳健性 1 与 2 均方向相反，或正式 IUT PASS 而各 Gate 声明的 guardrail 失败，一律降级为 INCONCLUSIVE**。不得以稳健性单独宣布 PASS。

伪题材 donor 复用导致的基准相关由真实题材簇聚类吸收，不把 200 个篮子当独立样本。

**Monte Carlo 精度闸门：** bootstrap/随机化不少于 5,000 次；若 \(\alpha_v\) / false-death / Holm 阈值两侧 Monte Carlo 95% 二项 CI 跨越决策阈值，增至 50,000；仍跨越则 INCONCLUSIVE。

## 20.3 多重检验

核心假设家族：

    H-DISC
    H-CONF
    H-ROLE
    H-MIG
    H-CLOCK
    H-CARRIER
    H-INT
    H-EXEC
    H-EXIT

上述9个正式 Gate 使用 §18.0 固定顺序 gatekeeping（每 Gate 使用版本预算 \(\alpha_v\)）；每个复合 Gate 使用 IUT；单版本核心家族强 FWER≤\(\alpha_v\)，项目级任一版本错误 PASS 概率 ≤5%（因 \(\sum_k\alpha_v(k)=0.05\)），单版本 false-death FWER≤5%。跨版本 false-death 不受控（§18.0）。不得在前序未 PASS 时计算或解释下游正式 p 值。主法庭仅 Track S。

Gate S 在核心 Gate 完成后独立评估三个 overlay 候选，使用 Holm FWER \(\alpha_v\)；Study 5 为 EXPLORATORY，不进入正式家族。

探索结果必须标记：

    EXPLORATORY

不得在当前评估样本中升级为主规则。

## 20.4 样本充分性

满足任一项则判定：

    INSUFFICIENT_EVIDENCE

**成交率（全文唯一定义，v8）：**

\[
FillRate
=
\frac{\sum FilledValue}
{\sum TargetValue}
\]

求和遍及考察窗内全部理论买单（含 REJECT_* / 部分成交）。部分成交按实际成交金额计入分子。某日若理论订单数为 0，该日 FillRate 记 `NA`，**不进入**“连续10日成交率”分母，也不按 0 触发安全 Kill。

**收益贡献（v8）：** 仅当考察窗总净超额 \(R^{tot}>0\) 时评估集中度；否则跳过年度/题材簇贡献条款并登记 `CONTRIB_SKIPPED_NONPOSITIVE_TOTAL`。定义正贡献份额：

\[
Share_g
=
\frac{\max(PnL_g,0)}{\sum_{g'}\max(PnL_{g'},0)}
\]

条件：

- 可评估独立episode少于120；
- 样本少于3个自然年度；
- \(R^{tot}>0\) 且单一自然年度正贡献份额 >50%；
- \(R^{tot}>0\) 且单一题材簇正贡献份额 >20%；
- FillRate < 70%；
- `NON_EVALUABLE` 题材日超过所有合格题材日30%；
- 任一正式 Gate CI 因有效题材簇少于30个而不可估计。

## 20.4-bis DESIGN 期信号频率可行性核查

此核查只回答“按当前信号定义是否有可能在墙上时间内获得足够样本”，不得使用收益方向调阈值。

- 在 DESIGN 全区运行完整信号链，报告年化独立 episode 数及其 Poisson 95% 区间；
- 若年化点估计 < 40 或区间上界 < 60，当前设计记 `INFEASIBLE_SIGNAL_RATE`，不得进入 VALIDATION；可修订信号定义，但必须创建新 SPEC 版本；
- 若 40–60，记 `LOW_SIGNAL_RATE`，允许进入 VALIDATION，但必须在项目墙上时间上限内证明可满足 §20.4；
- 核查只使用信号数量，不读取任何未来收益、方向或 Gate 效果。

## 20.5 经济门槛

固定20日入口政策取得 `PASS_ENTRY_ONLY` 资格同时要求下列全部成立（公式 v10 唯一）：

1. episode 层 NetAlpha20 按附录 B Gate 8 的 PASS 规则通过；
2. **年化净超额：** \(\widehat{AnnExc}=250\times Mean_t(r^{exc}_t)\)，其中 \(r^{exc}_t\) 为组合对暴露匹配基准（§0.3.3）的日超额；要求 \(\widehat{AnnExc}\ge 0.05\)。禁止改用 NAV 比值 CAGR 作为主口径（CAGR 仅诊断）；
3. 至少60%的自然年度 \(\sum_{t\in year}r^{exc}_t>0\)；
4. **最大回撤倍数：** 令 \(MDD_p=\max_{u\le v}(1-NAV_v/NAV_u)\)（组合 NAV），\(MDD_b\) 为同区间暴露匹配基准净值同一定义；要求 \(MDD_p\le 1.25\times MDD_b\)。若 \(MDD_b=0\)：当 \(MDD_p=0\) 通过，否则失败并登记 `MDD_BENCH_ZERO`；
5. **成本翻倍：** 将 §14.4 的佣金、印花税、交易所费用、固定滑点与冲击 **全部×2** 后重跑完整账本，要求累计净超额 \(\sum r^{exc}>0\)；
6. **剔除最佳5% episode：** 按 episode 层 NetAlpha20 排序删除最高5%（向上取整至少一个）后，**重跑完整现金账本与后续订单**（非事后从总 PnL 做减法），要求累计净超额仍 >0；
7. FillRate（§20.4）≥70%；
8. §15.3 账本恒等式零失败。

最终 `PASS` 要求：Gate 1–9 全 PASS，**且**动态退出完整组合独立满足上列第2–8项。若 Gate 9 PASS 但动态完整组合第2–8项失败 → 终局 `PASS_ENTRY_ONLY`（不是 `FAIL_ECONOMIC_GUARDRAIL`，也不是 `PASS`）。固定20日政策不满足第2–8项 → `FAIL_ECONOMIC_GUARDRAIL`。以上为工程门槛；修改即新 SPEC。

## 20.6 一等读数清单

以下读数与 NetAlpha20 同列主报告首页：

- 走廊测量全表；
- 执行拒绝分解；
- REJECT_GAP 逆向选择对照；
- Gate 8B PolicyNetAlpha20 与 FillRate；
- CLIMAX 触发频率；
- 容量曲线；
- `NON_EVALUABLE` 分解与共同支持覆盖率；
- P10 年化假点火/假主信号率；
- 风险仓位路径与暴露匹配基准路径；
- Gate 三态判词及 CI 相对 MES 的位置。

---

# 21. 样本分区、Run-in、漂移与冻结

## 21.1 样本分区的确定性解析

- `DESIGN`：数据检查、指标实现、阈值语义验证；
- `VALIDATION`：检查方向与代码；
- `HOLDOUT`：参数、代码、费用和判词冻结后解封；
- `FORWARD`：冻结后每日前向归档。

不得人工选择分区日期。对同一数据快照与触碰日志，按以下算法唯一解析：

1. `USABLE_START`：DG01–DG16 全部可满足且具备最长前视无关回溯窗（250 个交易日）的首个交易日；
2. `TOUCHED_THROUGH`：冻结前任何历史研究、人工或程序曾读取结果变量的最大 `event_date`；由只增不改的 `touched_data_log.jsonl` 取最大值。没有记录时取 `USABLE_START - 1交易日`；
3. `FREEZE_DATE`：Canonical SPEC、代码、参数与容器联合哈希签署日；
4. `PRETOUCH_END = max(USABLE_START, TOUCHED_THROUGH)`。区间 `[USABLE_START, PRETOUCH_END]` 的交易日按时间升序排列，共 \(N\) 日；
5. `DESIGN` = 前 \(\lfloor0.60N\rfloor\) 个交易日；`VALIDATION` = 其余已触碰交易日。若任一区少于 252 个交易日，判 `INSUFFICIENT_PARTITION_HISTORY`，不得继续；
6. `HOLDOUT_START` = 严格晚于 `max(TOUCHED_THROUGH, FREEZE_DATE)` 的首个交易日；
7. `HOLDOUT_END` = `HOLDOUT_START` 三周年纪念日之前的最后一个交易日（固定 36 个自然月，不按事件数提前停止）；
8. `FORWARD_START` = 严格晚于 `HOLDOUT_END` 的首个交易日。

解析器输出 `sample_partition.yaml`，包含上述输入事实、解析结果、交易日历版本和 SHA-256。该文件只物化本文算法，不增加自由参数。

机器校验：

    DESIGN_END < VALIDATION_START
    VALIDATION_END <= TOUCHED_THROUGH
    max(TOUCHED_THROUGH, FREEZE_DATE) < HOLDOUT_START
    HOLDOUT_END < FORWARD_START

任何已读取结果变量的数据均不得进入 HOLDOUT；触碰日志与解析器代码是冻结物。删除或回写触碰日志判 `INVALID_DATA`。

## 21.2 冻结物

- 本SPEC SHA256；
- `params.yaml`；
- `fee_schedule.yaml`；
- 数据字典；
- Track B 文本模式；B-LEX hashing/IDF状态或 B-EMB 模型与 training cutoff；
- 伪题材生成器全配置；
- §8.3.8 计算 DAG、缓存 schema 与 naive reference 实现；
- `compute_slo.yaml` 与最近60日影子负载报告；
- §26.3 审计 runner 镜像、scheduler配置与 service account 权限清单；
- Git tag；
- 依赖锁文件；
- 原始数据快照哈希；
- 随机种子规则；
- 报告模板；
- 本文 §21.7 Kill Criteria；
- 样本分区日期与触碰日志；
- Phase-I null/漂移基线。

## 21.3 解封后禁止

- 修改阈值或权重；
- 更换主终点；
- 修改题材发现来源或伪题材生成器；
- 删除不利样本；
- 修改执行价格、费用或基准；
- 将探索变量加入主信号；
- 修改 MES、三态边界或 Kill Criteria。

任何修改：

    关闭当前实验
    登记失败或制度重置原因
    创建新SPEC
    建立新的未触碰样本/measurement regime

## 21.4 Gate×样本区矩阵

|Gate|DESIGN|VALIDATION|HOLDOUT|FORWARD|
|---|---|---|---|---|
|0 数据/PIT/生成器|开发+NG/P10|完整重跑|完整重跑|每日持续|
|1 发现|开发|主评估|终验一次|季度监控|
|2 确认|开发|主评估|终验一次|季度监控|
|3 角色可分离|开发|主评估+角色随机化|终验一次|季度监控|
|4 角色迁移|开发|主评估|终验一次|季度监控|
|5 容量时钟|开发|校准+主评估|终验一次|季度监控|
|6 载体选择|—|主评估+成分归因|终验一次|季度监控|
|7 过滤交互|—|主评估|终验一次|季度监控|
|8 执行可实现|—|主评估+逆向选择|终验一次|每月监控|
|9 退出增量|—|主评估|终验一次|季度监控|
|S 市场状态叠加|开发|评估|升级决策|季度监控|
|Study 5|—|执行一次|不进入|不进入|

铁律：

1. 每个 Gate 在 HOLDOUT 只评估一次；失败或 INCONCLUSIVE 后不得回 VALIDATION 调参重试同一假设；
2. VALIDATION 上所有看结果后修改次数计入 DoF 记账；
3. FORWARD 只做 Kill 与漂移监控，不做参数调整。

## 21.5 FORWARD 最小样本

中期判读必须同时满足：

- 累计不少于60个独立episode；
- 且不短于12个自然月。

episode 数与墙上时间必须同时满足，不采用“任一先到”规则。

未达前，FORWARD 数据只归档、执行数据/仪器安全 Kill，不作收益功效 Kill、不对外报告绩效。

## 21.6 工程 Run-in、漂移与 regime reset

### 工程 Run-in

正式 FORWARD 前连续20个交易日影子运行，要求：

- 生产实现与独立重发审计的每日正式输出满足 §26.3 一致性；
- 无 DG/NG、账本、任务超时故障；
- 信号在 D 21:30 前只增不改归档；
- 模拟订单次日可由执行器完整重放。

任一失败后修复并重新开始完整20日计数；不得用收益方向决定 Run-in 停止。

### Phase-I 基线

由 VALIDATION（不含收益方向调参）冻结：

- 每日候选题材数、共同支持覆盖率、NON_EVALUABLE率；
- IES、\(IDS^\theta\)、\(GCC^\theta\)、CCS_level 分布；
- P10 假点火/假信号率；
- 真实主信号率、FillRate（§20.4）、各拒绝率；
- Track S/B 占比、题材成员数分布；
- LACBreadth、LACTurnBreadth、LACRelReturn5、LACAttrition10 与 LACDecay 触发率；
- DAG 节点耗时、cache hit、COMPUTE_TIMEOUT 率与重发审计耗时。

### 漂移触发

任一项触发 `QUARANTINE`：

1. 28日滚动 P10 假信号率 > Phase-I 同层99%预测上界；
2. 任一核心变量分布相对 Phase-I 的 PSI > 0.25，连续5日。**PSI 唯一算法（v7 补，A-10）：** 每个监控变量使用 VALIDATION 冻结的等频10分位箱（首末箱开区间至 ±∞）；比较分布 = 最近28个交易日观测；两侧每箱频率各加 \(10^{-4}\) 后重新归一化；\(PSI=\sum_b (q_b-p_b)\ln(q_b/p_b)\)；28日窗内观测 <100 时该变量当日不判定；逐变量独立判定，任一变量触发即计数；
3. 共同支持覆盖率 < 70%，连续5日；
4. vendor schema、成员口径、文本模型、行业分类、费用制度或交易制度版本变化；
5. 生产输出与独立重发审计的信号/状态/订单不一致；
6. 账本恒等式失败；
7. §8.3.9 `QUARANTINE_COMPUTE` 触发。

### QUARANTINE 与重启

- 暂停新入场，已有仓位按冻结退出规则管理；
- 定位为数据故障可修复且不改变历史事实时：修复、独立审计 runner 重放影响区间、完成5个交易日 mini Run-in 后恢复；
- 测量制度/供应商/模型/交易制度实质变化：关闭当前 regime，创建新 SPEC/version，重新建立 Phase-I 基线；旧新 regime 的正式证据不得无条件池化；
- 全部事件只增不改归档。

### 项目墙上时间

自 FORWARD_START 起最长24个自然月。届时未同时满足 §21.5 → `INSUFFICIENT_EVIDENCE_TIMEOUT`，关闭当前实验，不得无限暂停等待。

## 21.7 FORWARD Kill Criteria

### 立即安全 Kill（不等待最小样本）

- INVALID_DATA（任何 DG/NG 失败）；
- LEDGER_INVALID；
- `REPLAY_MISMATCH` 连续2个交易日发生，或任一不一致在5个交易日内无法闭合；
- 连续10个**有理论订单**的交易日 FillRate < 30%（无订单日跳过，不打断连续计数）；
- 实际组合回撤 > 15%（相对 FORWARD 初始 NAV）。

处置：立即停止新入场、按冻结退出规则清算；判词 `KILL_SAFETY`。该判词是风险治理决策，不宣称科学机制为假。

### 绩效 Kill（仅在满足 §21.5 后季度开庭，序贯预算见下）

**序贯边界（v9）：**

1. **统计绩效 Kill（唯一宣称 false-death 受控者）：** FORWARD 期最多开庭 6 次（每季度一次，自满足 §21.5 起）；每次使用名义单侧水平 \(0.05/6\)。触发：FORWARD episode 层 NetAlpha20 的单侧 \(1-0.05/6\) 置信上界 < 0。Bonferroni 保证该统计 Kill 的版本内误杀概率 ≤5%。
2. **确定性风险止损（deterministic risk stop，v9）：** 下列路径阈值**不属于**统计 Kill，**不宣称** false-death≤5%：
   - 暴露匹配累计净超额 < −5%，且过去连续两个季度均为负；或
   - 成本翻倍压力下累计净超额 < −10%。
   触发输出 `KILL_RISK_STOP`（风险治理决策），不得写作科学 FAIL，也不得计入统计误杀预算。

处置：统计 Kill → `KILL_ECONOMIC_FAIL`；风险止损 → `KILL_RISK_STOP`；均停止新入场并清算。若机制指标正常而收益失效，登记 `ALPHA_CROWDING_CANDIDATE`；若机制指标同步失效，登记 `MECHANISM_DECAY_CANDIDATE`。二者均不得在同一 regime 内调参复活。

### 仪器漂移

按 §21.6 先 QUARANTINE；不可恢复时 `KILL_MEASUREMENT_REGIME`。不将仪器失败误写为策略科学 FAIL。

---

# 22. 终局判词

    INVALID_DATA
    INVALID_MEASUREMENT
    LEDGER_INVALID
    INSUFFICIENT_EVIDENCE
    INSUFFICIENT_EVIDENCE_TIMEOUT
    FAIL_DISCOVERY_UNIT
    FAIL_CONFIRMATION
    FAIL_ROLE_SEPARATION
    FAIL_ROLE_MIGRATION
    FAIL_CAPACITY_CLOCK
    FAIL_CARRIER_SELECTION
    FAIL_FILTER_INTERACTION
    FAIL_EXECUTION
    FAIL_ADVERSE_SELECTION
    FAIL_EXIT_INCREMENT
    FAIL_ECONOMIC_GUARDRAIL
    PASS_ENTRY_ONLY
    PASS
    KILL_SAFETY
    KILL_ECONOMIC_FAIL
    KILL_RISK_STOP
    KILL_MEASUREMENT_REGIME

报告标记（不覆盖上述唯一终局）：

    TRACK_B_INSUFFICIENT
    LOW_POWER_CLIMAX
    INCONCLUSIVE_EXIT
    OVERLAY_PASS / OVERLAY_REJECTED / OVERLAY_INCONCLUSIVE

逐层终止：

    Gate 0失败 → INVALID_DATA
    P10 绝对假信号预算超限 → INVALID_MEASUREMENT
    任一账本闸门失败 → LEDGER_INVALID
    Gate 1–8 任一 INCONCLUSIVE / 样本装置失败 → INSUFFICIENT_EVIDENCE
    Gate 1–8 任一有统计支持的 FAIL → 对应 FAIL_*
    Gate 1–8 全部 PASS，但固定20日入口政策未通过 §20.5 → FAIL_ECONOMIC_GUARDRAIL
    Gate 1–8 与固定20日入口政策通过，Gate 9 PASS，且动态完整组合 §20.5(2–8) 通过 → PASS
    Gate 1–8 与固定20日入口政策通过，Gate 9 PASS，但动态完整组合 §20.5(2–8) 失败 → PASS_ENTRY_ONLY（附 DYNAMIC_GUARDRAIL_FAIL）
    Gate 1–8 与固定20日入口政策通过，Gate 9 FAIL → PASS_ENTRY_ONLY，并附 FAIL_EXIT_INCREMENT
    Gate 1–8 与固定20日入口政策通过，Gate 9 INCONCLUSIVE → PASS_ENTRY_ONLY，并附 INCONCLUSIVE_EXIT
    Gate S 只产生 OVERLAY_* 报告标记；仅 OVERLAY_PASS 可启用叠加层，不改变主策略终局

不能用下游失败否定尚未单独失败的上游理论。**FAIL 是有反向证据的判词，不是“不够显著”的别名。**

---

# 23. 工程目录

    /sfl1_v10/
      SPEC.md
      params.yaml
      fee_schedule.yaml
      data_dictionary.yaml
      model_registry.yaml
      sample_partition.yaml
      touched_data_log.jsonl
      null_generator.yaml
      kill_criteria.yaml
      compute_slo.yaml
      ledger.csv
      /raw_snapshots/
      /pit_theme/
      /semantic_models/
      /features/
        stock_base/
        discovery/
        attention_roles/
        independent_confirmation/
        role_migration/
        capacity_clock/
      /episodes/
      /signals/
      /orders/
      /fills/
      /positions/
      /controls/
        /membership/
        /pseudo_stats/
      /cache/
        /stock_day_features/
        /text_features/
        /donor_index/
        /null_order_stats/
      /event_studies/
      /diagnostics/
      /audit_replay/
      /compute_manifests/
      /reports/
      /hashes/

---

# 24. 每日生产流程

    D−1 20:00
      └─ 冻结D日可观察的Base Universe（INPUT_SNAPSHOT_CUTOFF）

    D 开盘前（执行日时钟，强制顺序）
      ├─ 公司行动开盘前生效（§15.5；含 payment_date 现金入账）
      ├─ 昨日 UnsettledBuyShares → SettledShares
      └─ 更新 SellableQty / tax_lots

    D 15:00以后
      ├─ 落地日线、分钟线、交易状态
      ├─ 增量更新 StockDayFeatureStore 与 DonorIndex
      ├─ 更新持仓 episode 的 LAC 与五个动态健康变量
      ├─ 更新 episode 合并/分裂（ThemeRoom 不迁移）
      ├─ 输入 = D−1 Base Universe ∪ 全部 open episode 题材
      ├─ 按 §3.5：未确认识别临时角色；已确认加载冻结 L^E/K^E/B^E
      ├─ load_or_build_nulls_v12；Gate1 日另冻 DISCOVERY_RESEARCH_NULL 与 DISCOVERY_RESEARCH_COHORT_D
      ├─ 更新 PseudoThemeStats（不重建已冻结 membership）
      ├─ 全部 open/候选题材计算 IES/IDS/GCC（确认后只用冻结集合；确认谓词只用归档临时 IDS）
      ├─ 连通分量题材簇 + ConfirmQuality 代表（§5.3）
      ├─ 全部题材更新状态机；仅代表可新确认/开信号
      ├─ LOO 计算 IDS^-i、GCC^-i；Eligible 上 EntryScore（Pctl_cross 见 §13.2）
      ├─ τ_confirm 日执行 FrozenMetricBackcast（不回写确认 IDS）
      ├─ 全部 open episode 退出腿（origin 持仓；含升级与 T+1）
      ├─ 代表生成信号与人民币母订单预留
      └─ 运行 DG/NG/DG16、SLO、漂移与账本检查

    D 21:00
      └─ 未完成题材 COMPUTE_TIMEOUT；超10%则全日 QUARANTINE

    D 21:30
      └─ 信号、预留、退出订单、SHA256只增不改

    D+1 开盘前
      ├─ 公司行动开盘前生效
      ├─ 昨日 Unsettled → Settled
      └─ 更新 SellableQty

    D+1 09:35（决策时点）
      ├─ 用 09:30–09:34 数据完成 gap 与全部拒绝判定（连续封死才 UNFILLED_LIMIT）
      ├─ 释放被拒预留；设置 CommissionReserve
      └─ 对新/重试仍合格订单按统一排序再分配后进入流式参与

    D+1 09:35—10:00（流式参与）
      ├─ 逐分钟顺序冲击 + 比例费用推进 RemainingTargetValue
      ├─ 日终最低佣金结算与尾部手数调整
      └─ 当日买入写入 UnsettledBuyShares（当日不可卖）

    D+1 14:30—15:00
      └─ 执行前一日退出订单（只消费开盘前已 Settled 的 SellableQty）

    D 20:30以后（物理隔离）
      └─ 龙虎榜仅入 diagnostics

    D+1 12:00以前（独立审计环境）
      └─ 从只读原始快照重发 D 日全链并比较逐层哈希

---

# 25. 核心伪代码

    for D in trading_days:
        # --- 开盘前时钟（强制；先于任何当日交易）---
        apply_corporate_actions_pre_open(D)          # §15.5；payment_date 现金
        roll_yesterday_unsettled_to_settled(D)       # 仅昨日 Unsettled → Settled
        refresh_sellable_qty_and_tax_lots(D)

        stock_features = update_stock_day_feature_store(D)  # 无 theme-relative Rel
        donor_index = build_daily_donor_index(stock_features, D)

        # v9：候选题材 ∪ 全部仍开放 episode 题材（防 vendor 删题后持仓停更）
        themes = (
            load_base_universe_asof(D - 1)
            | themes_of(open_episodes)
        )

        update_live_active_core(open_episodes, D)
        update_episode_structure(open_episodes, D)  # MERGED_INTO 不迁移 ThemeRoom
        lac_health = compute_lac_health_and_decay(open_episodes, D)

        # Pass 1：全部 themes（含非代表 open episode）计算度量
        theme_metrics = {}
        for theme in themes:
            episode = get_open_episode(theme)
            if episode is None:
                base = eligible_base_members(theme, D)
                leaders = identify_attention_roles(base, D)
                capacity = identify_capacity_candidates(base - leaders, D)
                nulls = load_or_build_nulls_v12(theme, D, mode="DAILY_NULL",
                                              donor_index=donor_index)
            else:
                base = episode.B_E
                leaders = episode.L_E
                capacity = episode.K_E
                nulls = load_or_build_nulls_v12(theme, D, mode="EPISODE_NULL",
                                              donor_index=donor_index)

            if not pass_theme_gate(base, D):
                # v9：结构门槛失败仍必须跑风险分支，不得 continue 跳过
                apply_risk_branches_only(theme, lac_health, D)
                emit_pending_exits_and_upgrades(theme, D)
                continue

            if len(nulls) < 100 or not pass_balance_gate(nulls):
                apply_risk_branches_only(theme, lac_health, D)
                emit_pending_exits_and_upgrades(theme, D)
                log_non_evaluable(theme, D)
                continue

            pseudo_stats = update_pseudo_theme_stats(nulls, D)
            ids_theme = independent_diffusion(
                members=base - leaders, nulls=pseudo_stats, owner="theme")
            ies = ignition_evidence(theme, leaders, base, pseudo_stats)
            # Gate1：同步冻结真实 cohort 与 null（§8.3.3-1b/1c）
            maybe_freeze_discovery_research_cohort_and_null(theme, D, base, leaders, nulls)

            theme_metrics[theme] = dict(
                base=base, leaders=leaders, capacity=capacity,
                nulls=nulls, pseudo_stats=pseudo_stats,
                ies=ies, ids_theme=ids_theme, episode=episode)

        representatives = select_cluster_representatives(theme_metrics, D)

        # Pass 2a：全部有度量的题材/episode 更新状态与迁移（含非代表）
        for theme, m in theme_metrics.items():
            # τconfirm 只用归档临时 IDS 历史；不得用 backcast 重判
            role_cohort = load_or_freeze_role_cohort_on_first_confirmation(
                theme=theme, date=D, base=m["base"],
                leaders=m["leaders"], capacity=m["capacity"],
                ies=m["ies"], ids_theme=m["ids_theme"],
                confirm_ids_history_source="ARCHIVED_DAILY_NULL_SCALE",
                allow_new_confirm=(theme in representatives),
            )
            if role_cohort.just_frozen:
                frozen_metric_backcast(role_cohort, D)  # §3.5.1；不回写确认 IDS
            if role_cohort.status == "NO_ROLE_COHORT":
                update_discovery_confirmation_state_only(
                    theme, m["ies"], m["ids_theme"])
                continue

            ccs_levels = carrier_scores_level(
                role_cohort.K_E if role_cohort.is_frozen else m["capacity"],
                m["pseudo_stats"])
            gcc_theme = median(ccs_levels.values()) if ccs_levels else 0.0
            if role_cohort.is_frozen:
                rmi = role_migration_index(
                    role_cohort.B_E, role_cohort.L_E, role_cohort.K_E, D)
            else:
                rmi = None  # 未确认不得进 Pctl_cross

            state = lifecycle_state_once_per_theme(
                theme, ies=m["ies"], ids_theme=m["ids_theme"],
                gcc_theme=gcc_theme, rmi=rmi,
                lac_decay=lac_health.get(theme).decay_if_held,
                non_evaluable_freezes_normal_only=True,
                priority=PARAMS.state_priority,
            )

            # Pass 2b：仅代表可开主信号
            if theme in representatives and first_entry_window(theme, state):
                capacity_trade = current_trade_eligible_subset(role_cohort.K_E, D)
                for stock in capacity_trade:
                    ids_loo = exact_loo_from_sufficient_stats(
                        m["pseudo_stats"], removed_recipient=stock)
                    gcc_loo = exact_delete_one_median(ccs_levels, stock)
                    if pass_candidate_entry(
                        stock, ids_loo, gcc_loo, ccs_levels[stock], D):
                        emit_signal(valid_until=D + 3)

        emit_exit_orders_with_upgrade(open_episodes, D)  # 全部 open，含非代表
        reserve_initial_cash(signals, retries, PARAMS)
        assert_ledger_identities()
        enforce_compute_slo_or_quarantine(D)
        run_drift_checks()

        # --- 执行段：仅在开盘前 settle 完成之后 ---
        # D+1 09:35
        reject_and_release_reserves(orders)  # 连续封死才 UNFILLED_LIMIT
        set_commission_reserves(eligible_orders)
        reallocate_cash_at_0935(eligible_orders, PARAMS)
        fills = execute_streaming_participation_sequential_impact(orders)  # P_ex includes impact once
        settle_minimum_commission_eod(fills)  # A-09
        update_reservations_and_positions(fills)  # 新买 → UnsettledBuyShares
        assert_ledger_identities()
        verdict = three_state_gate_evaluation()  # Track S main court

---

# 26. 可复现性与独立重发审计

## 26.1 每次运行固化

- Git commit、Python、依赖、容器镜像；
- 输入快照、题材PIT、行业/股本PIT、基准成分哈希；
- 文本源/模型版本与训练截止日；
- params、fee、null_generator、sample_partition、Kill Criteria 哈希；
- 随机种子；
- 中间表行数；
- 信号、预留、订单、成交、账本和判词。

## 26.2 确定性

- 时区固定 `Asia/Shanghai`；
- 浮点统一 `float64`；
- 金额统一人民币元；
- Pctl 唯一定义按 §6.7；
- EMA 唯一定义按 §6.8；
- 全部排序以股票代码最终打破并列；
- 聚类 tie-break 按 §3.2；
- null 生成器算法按 §8.3；
- 随机对照使用 PCG64 与哈希种子；
- 相同输入必须产生相同输出哈希。

## 26.3 单实现 + 独立重发审计

### 组织裁决

- 正式系统只维护**一个 Canonical 生产实现**；不要求第二团队或第二代码实现；
- 独立重发审计用于验证输入不可变、计算确定性和正式输出可重建，**不宣称能发现生产代码与 SPEC 的共同语义错误**；
- 语义错误风险由 Canonical 静态检查、黄金样例、§8.3 NG06 naive reference 和人工 code review 控制。

### 独立性要求

审计 runner 可以使用同一冻结 Git commit 与依赖锁，但必须：

1. 由独立 scheduler 和 service account 启动；
2. 在洁净临时容器中从冻结镜像启动；
3. 只读访问原始快照、PIT 数据与正式输入 manifest；
4. 禁止读取生产中间表、生产 feature/null cache、内存状态或临时文件；
5. 从 ETL 开始重算下列全部正式对象；
6. 将输出写入独立 audit namespace，审计完成前无权覆盖生产结果。

重发比较对象：

- 输入 manifest、有效宇宙、Base Universe、Entry Cohort、LAC 与 LAC 动态变量；
- null 候选池、membership 对齐矩阵、伪题材统计量、平衡结果；
- episode 起止、合并/分裂/冷却；
- 注意力/容量集合及 H-ROLE outcome；
- 题材级与候选级 IES/IDS/GCC、\(RMI^E\)、CCS；
- 状态机、信号、预留、订单、拒绝、成交；
- 账本、Gate 判词、终局判词。

### 频率与时限

- Run-in 20个交易日：每日全链重发；
- FORWARD：每个交易日重发 D−1 全链，最晚 D+1 12:00 完成；
- 每周末额外从该周首日原始快照连续重发整周，验证跨日状态；
- HOLDOUT 解封前：全区一次完整重发，完成后才能开庭。

### 一致性与处置

|对象|容差|
|---|---|
|代码、日期、状态、集合、ID、行数|完全一致|
|分位、排名、整数计数、布尔判定|完全一致|
|价格、金额原始值|相对误差 ≤ 1e-12|
|派生浮点|相对误差 ≤ 1e-10|
|跨 BLAS/库聚合统计|相对误差 ≤ 1e-8，且排名、信号、判词不变|
|随机算法|同种子输出完全一致|

离散差异或超容差浮点差异记 `REPLAY_MISMATCH`，保存首个分叉表、首个分叉行、上下游哈希与容器日志。处置：

- 差异未闭合前暂停新入场，已有仓位按冻结退出规则管理；
- 输入 manifest 不同 → `INVALID_DATA`；
- 同输入但输出不同 → `NONDETERMINISTIC_IMPLEMENTATION`；
- 连续2日发生或5个交易日未闭合 → §21.7 `KILL_SAFETY`；
- 修复改变正式逻辑则必须新 SPEC/regime；仅修非确定性且结果语义不变时，完整重发影响区间并完成 mini Run-in。

---

# 27. 参数主表

正文每个可配置数字必须登记；CI 对 SPEC 与 `params.yaml` 做数字一致性 lint。制度事实、公式常数与章节编号不计配置参数。

|参数|主值|单位/尺度|来源|敏感性|
|---|---:|---|---|---|
|上市最短天数|120|交易日|先验|60 / 250|
|有效交易日下限|15/20|日|先验|—|
|题材最小成员 / N_eff|8 / 5|只 / —|先验|6/12；4/8|
|正常交易成员比例|70|%|先验|—|
|题材归簇Jaccard|0.60|—|先验|0.5 / 0.7|
|episode合并Jaccard/持续|0.60 / 2|— / 日|先验|—|
|episode分裂簇内/簇间/持续|0.60 / 0.30 / 5|—|先验|—|
|冷却期|10|交易日|先验|5 / 20|
|LAC相关/窗口/移出|0.50 / 20 / 10|— / 日|先验|—|
|LACDecay Breadth/TurnBreadth|0.20 / 0.30|份额|风险先验|0.15/0.25；0.25/0.35|
|LACDecay 连续日/Attrition10|2 / 0.40|日 / 份额|风险先验|3；0.30/0.50|
|B-LEX ngram/hash维度/seed|3–5 / \(2^{18}\) / 0|字符 / 维 / —|工程冻结|—|
|B-LEX 文本窗/指数衰减常数|120 / 60|日|工程先验|60/250；30/90|
|注意力核心最多/第二名门槛|2 / 90%|只 / %|先验|1/3|
|伪题材目标/最低数量|200 / 100|个|先验/工程|500；150|
|null 最近邻候选数|20|只|工程|10 / 40|
|null 马氏正则|1e-6|—|工程|—|
|匹配 SMD 上限|0.25|—|工程|0.10 / 0.20|
|P10 cross-fit folds|5|fold|工程冻结|10|
|NG06/07 分层验收题材日|100|题材日|工程|200|
|NG07 信号率差/分位相关|≤1pp / ≥0.99|—|精度闸门|更严格仅报告|
|P10 Phase-I bootstrap/分层最小分母|10,000 / 500|次 / control观测|工程|—|
|P10 年度有效日覆盖率|90|%交易日|工程闸门|95%报告|
|计算 SLO p95/p99|60 / 90|分钟|运行治理|—|
|计算截止/超时隔离比例|21:00 / 10%|时间 / 题材|运行治理|—|
|IES / IDS 阈值|0.80 / 0.80|null分位|先验|0.75 / 0.85|
|Active残差/TurnZ/CLV|0.85 / 0.5 / 0.70|—|先验|—|
|TurnZ基线|60|日|先验|—|
|容量市值/ADV分位|前40% / 前40%|题材内|先验|33% / 50%|
|容量组最小只数|3|只|先验|—|
|CCS_level门槛|0.60|null分位|先验|0.55 / 0.65|
|GCC启动带|[0.55,0.85]|null分位|先验|±0.05|
|\(CapacityBreadth^E\)|0.40|份额|先验|0.33 / 0.50|
|BROAD_TREND \(RMI^E\) 下限|-0.02|RMI|先验|−0.01 / −0.03|
|\(RMI^E\)/Δ|EMA3 / 3|span / 日|先验|EMA2/5|
|P5 角色置换次数|5,000|次|推断|—|
|EntryScore权重|.40/.25/.20/.15|—|工程|等权|
|每题材选股上限|2|只|先验|—|
|信号有效期|3|执行日|先验|—|
|主持有/最大持有|20 / 60|交易日|先验|10/40；40/80|
|Leg I 相对收益/CCS/TurnZ|-4%/.35/0|—|先验|—|
|Leg B2 IDS回撤|0.20|null分位|先验|.15/.25|
|CLIMAX低功效|1|%状态日|先验|—|
|题材失败IDS|0.30|null分位|先验|.25/.35|
|买入/卖出窗口|09:35–10:00 / 14:30–15:00|时间|制度|—|
|REJECT_GAP分位/冷启动数|.85 / 40|— / 个|先验|.80/.90|
|临时跳空阈值 主/创科/北交|.06/.09/.12|收益|先验|—|
|REJECT_GAP逆向选择非劣界|1|百分点|经济护栏|0 / 2|
|单股/题材/总仓位|10%/20%/60%|NAV|先验|—|
|最低建仓权重|2|% NAV|工程|1% / 3%|
|同时题材簇|3|簇|先验|—|
|叠加差时仓位|30|%|先验|Gate S|
|订单ADV/早盘参与率|0.5% / 5%|—|先验|—|
|研究资本|1000万|元|工程|—|
|样本episode/年度/FillRate|120/3/70%|—|工程|—|
|NON_EVALUABLE上限|30|%题材日|工程|—|
|最小推断题材簇|30|簇|工程|—|
|年化净超额/正年度/回撤倍数|5%/60%/1.25|—|工程|—|
|Gate 3 角色双 contrast MES|0.05|null分位差|经济先验|0.03 / 0.10|
|Gate 3 正向年度比例|60|%年度|工程护栏|—|
|Gate 4 StartRate/CDS/PriceShare MES|0.10 / 0.05 / 0.05|份额差|经济先验|减半/加倍报告|
|DESIGN可行性年化信号|40（关闭）/60（稳健）|episode|工程|—|
|FORWARD最小样本|60 episode AND 12月|—|工程|—|
|Run-in / mini Run-in|20 / 5|交易日|工程|—|
|重发审计日截止|D+1 12:00|时间|治理|—|
|重发不一致 Kill|连续2日 / 未闭合5日|交易日|治理|—|
|项目墙上时间|24|自然月|工程|—|
|PSI/共同支持漂移|.25 / 70%|—|工程|—|
|安全Kill成交率/回撤|30% / 15%|—|风险|—|
|绩效Kill累计超额|−5%|—|风险|—|
|fixed_slippage_bps / lambda|5 / 0.50|bps / 无量纲|工程先验|3/10；0.25/1.00|
|INPUT_SNAPSHOT/COMPUTE/ARCHIVE|20:00 / 21:00 / 21:30|时钟|治理|—|
|信号侧容量地板 MinPositionValue|2%×研究资本|元|工程|—|
|执行分钟参与率|5|%分钟成交量|制度先验|2.5% / 10%|
|g 定义窗口|09:30–09:34 VWAP|分钟线|定义|开盘价仅备援|
|开窗超时关闭|15|交易日|先验|10 / 20|
|Gate 5 β3 MES|0.005|1σ×1σ 20日毛超额|经济先验|0.003 / 0.01|
|Gate 1/2 MES_pass|0.05 / 0.10|AUC差 / 率差|经济先验|减半/加倍报告|
|Gate 6/7/8 MES_pass|0.01 / 0.01 / 0.01|20日收益差|经济先验|减半/加倍报告|
|Gate 9 MES_pass（ΔSharpe/ΔCumExcess）|0.10 / 0.02|年化Sharpe / 收益差|经济先验|减半/加倍报告|
|Gate 6 近邻池/臂内只数|5 / 2|只|工程|—|
|Gate 2/7 匹配卡尺|VALIDATION 配对距离90%分位|马氏|工程|85% / 95%|
|Gate 9 block bootstrap|21日 / 10,000次|块 / 次|推断|—|
|P10 绝对预算 率/年化|5% / 20|— / episode|经济先验|3%/10；8%/40|
|版本 α-wealth|0.05×2^{-k}（v10: k=1, α_v=0.025）|—|治理|—|
|false-death 预算|每elementary 1−0.05/(9m)|置信度|治理|—|
|统计绩效 Kill 开庭|6次 / 名义0.05/6；另设确定性风险止损|季 / 单侧|治理|—|
|PSI 分箱/窗口/最小观测|10分位 / 28日 / 100|—|治理|—|
|样本分区|§21.1 确定性算法|交易日|治理|禁止人工覆盖|

敏感性只报告稳定性，不允许选择最优参数替换主口径。

---

# 28. 研究依据、竞争解释与谨慎解释

本框架吸收但不直接照搬以下证据：

1. 共同注意力能够解释部分A股收益联动，且在机构持股较低股票中更明显；不证明机构集团作战。
2. A股新闻日收益可能混有零售注意力造成的暂时价格压力；支持将点火与可持续需求分开。
3. A股价格同时受公司信息、噪声、市场信息和注意力影响；题材框架不能解释全部行情。
4. 仅用订单大小推断资金身份可能系统性误分类；资金身份不进入核心信号。
5. 价格优先、时间优先意味着涨停开盘不等于策略订单可成交。

**竞争解释与可判读失败条件：**

|竞争解释|主要鉴别装置|
|---|---|
|机械横截面动量|P3 日期错位、Gate 7 交互|
|题材本来就热|§8.3 匹配注意力/拥挤度、P4|
|单股龙头自我认证|P1/P2、候选级 LOO|
|大市值/高流动性溢价|匹配市值/ADV、Study 4 Controls|
|策略拥挤抢跑|FORWARD 收益衰减但机制路径稳定，§21.7|
|数据/供应商制度漂移|P10、PSI、共同支持率，§21.6|

机制构造属工程假设，其有效性只能由 §17–§21 验证，不得先验视为已证实。

---

# 29. 最终理论判词

SFL-1 v12.0采用的弱因果表述：

    叙事或信息降低协调成本
    → 高显著性股票形成注意力点火
    → 其他成员是否独立扩散决定题材是否真实
    → 注意力与容量功能必须先被独立证明可分离
    → 不同容量约束使需求寻找不同载体
    → 容量需求开始扩散或接管定价
    → 若价格尚未完成扩张且真实可成交，则可能存在中段

真正需要验证的是：

\[
\text{确认后的独立扩散}
\rightarrow
\text{角色功能可分离}
\rightarrow
\text{容量需求加速}
\rightarrow
\text{可成交容量载体未来净收益}
\]

每个箭头都可单独失败；“不显著”只表示证据不足，不自动表示箭头为假。

本规格的 Canonical 纪律：

    排序变量不得用来定门槛（rank ≠ level）；
    题材级规则不得消费候选级变量（owner 必须匹配）；
    分位算子必须唯一处理并列，单调缩放不能修复离散堆积；
    H-ROLE 与 H-MIG 必须分别开庭，不得互相代理；
    冻结 Entry Cohort 负责科学归因，动态 LAC 只负责持仓风险；
    null 生成器不是输入，而是必须通过验收的测量仪器；
    null 缓存只能做数学等价加速，不得减少 controls 或改变精度；
    Track B 必须同时满足 PIT 与可执行性，历史主模式不得依赖未来训练模型；
    主终点必须与实际成交时钟对齐，处理与对照执行同构；
    FAIL 必须由反向证据支持，不显著不得冒充 FAIL；
    组合收益必须对风险暴露匹配基准验收；
    单一生产实现必须接受隔离环境独立重发审计；
    冻结物不能只有名字，Kill 与漂移规则必须有完整内容；
    任何决策只能使用其决策时点已完整可见的信息（时点级 PIT）；
    每个正式变量必须有唯一公式与缺失规则（附录 D 变量字典）；
    每个对照必须有完整生成器，"匹配"不是算法；
    PASS、FAIL、Kill 与版本三类错误率各有独立预算；
    跨章节同一正式对象必须同构（确认集合、null、控制生成器、账本语义）；
    Gate estimand 不得同时声称互斥的科学问题（如纯退出增量 vs 端到端资金）；
    题材簇在去重、null 排除、仓位与统计 cluster 中必须是同一连通/persistent 对象；
    处理组与控制组必须使用同一种角色冻结制度与同一信号日期；
    政策比较必须先外生冻结输入 tape，再谈退出增量；
    确认冻结后差分变量必须 FrozenMetricBackcast，禁止换尺拼接；
    确认谓词永久使用归档临时量尺，backcast 不得反向改写确认；
    账务合并不得暗含 measurement/null 合并，也不得迁移 ThemeRoom；
    人民币母订单、顺序冲击、最低佣金预留、T+1 开盘前结算与公司行动开盘前生效必须进入正式账本；
    Gate 对照不得提前成交却共享处理退出时钟；
    Gate8A 主 NetAlpha20 为 conditional-on-fill；Gate8B 全信号政策收益强制诊断且不得混 CI；
    no-gap 必须隔离研究账户；Impact 只通过执行价计入一次；
    trade_lot / AvgCost / tax_lot 职责分离；P10 identity key 与 supersede 唯一；
    Gate9 资本分母唯一为最大并发部署名义；稳健性 bootstrap 唯一按 persistent component 整块抽样。

---

# 附录 A：Canonical 闭包索引

本索引是本文自包含性的审计入口，不引入任何新规则：

|规范域|唯一正文位置|闭包内容|
|---|---|---|
|经济命题、识别边界、Estimand|§0–§2|可证伪机制、共同支持域、NetAlpha20、两层基准|
|发现、集合与变量 owner|§3、§5|Track S/B、§3.5 映射、连通分量簇、open∪候选日更|
|数据与 PIT|§4|字段、vol=股、分钟 bar、公司行动、DG01–DG16|
|统一算子|§6|收益、行业收益、TurnZ、CLV、Active、Pctl、EMA、Hash/seed、缺失窗口|
|角色、确认与 null|§7–§8|ALS/L、两级 IDS、伪题材生成器、等价 DAG、平衡与 NG 验收|
|容量、迁移与时钟|§9–§11|CCS rank/level、\(RMI^E\)、\(HHI^E\)、两级 GCC|
|状态与事件生命周期|§3.3、§12|冻结迁移与 LAC 动态健康分轨、固定优先序、吸收态、COOLDOWN|
|入场、执行与账本|§13–§16|人民币母订单、冲击单次计入、佣金预留、trade/tax lot、开盘前 T+1/公司行动、部分成交、预留、退出|
|机制研究与正式 Gate|§17–§20、附录 B|Gate8A/8B、CGM、persistent-component bootstrap、MES|
|样本、Freeze 与运行治理|§21–§22、附录 C|分区、Run-in、漂移、Kill、终局判词、Freeze|
|工程复现|§23–§27|目录、生产流程、伪代码、单实现独立重发审计、参数主表|

Canonical 静态检查必须证明：

1. 本文不存在对任何早期 SPEC 的规范性回引；
2. 不存在要求从本文以外补全定义的省略式规则；
3. 全部内部章节引用均指向本文现存章节；
4. 全部正式参数在 §27 或明确的 Freeze 输入登记表中出现；
5. 外部机器文件只序列化本文规则，不创造规则。

# 附录 B：Gate 科学法庭十项表

主推断算法按 §20.2。下表逐行展开 Multiplicity 与 Sequential，不以共享省略替代；样本/装置失败或区间落在灰区均 INCONCLUSIVE。

|Gate|Population / Sample|index_date（ID2 源）|Estimand \(\theta_g\)|Null|MES_pass / MES_fail|Dependence|Multiplicity|Sequential|Decision / Failure state|
|---|---|---|---|---|---|---|---|---|---|
|1 发现|Track S；共同支持域内 D−1 候选题材日|候选测量日 D|未来5日 NewJoin AUC：DISCOVERY_RESEARCH_COHORT_D − DISCOVERY_RESEARCH_NULL_D|≤0|0.05 / 0|persistent簇×月 CGM|固定顺序第1，α=α_v|HOLDOUT一次；FORWARD仅Kill|真实与 null 同日冻结|
|2 确认|Track S；确认处理 vs §17.1 匹配控制（Control Measurement Cohort）|确认日 D|Whipsaw率：控制−处理|≤0|0.10 / 0|persistent簇×月 CGM|固定顺序第2，α=α_v|仅Gate1 PASS后；HOLDOUT一次|Gate2_ControlRegistry 独立；平衡失败整样本无效|
|3 角色可分离|已确认且 \(L^E/K^E\) 均非空的 episode（episode 等权）|\(\tau_{confirm}\)|\(\theta^{ROLE}_A,\theta^{ROLE}_C\)|任一≤0|各0.05 / 0|persistent簇×月 CGM|固定顺序第3；m=2 IUT|仅Gate1–2 PASS后；HOLDOUT一次|两者均PASS才PASS；任一FAIL才FAIL；其余INCONCLUSIVE|
|4 迁移|Gate 3 PASS 域内已确认 episode（episode 等权）|\(\tau_{confirm}\)|\(\Delta StartRate_5,\Delta CDS_5,\Delta PriceShare_5\)|任一≤0|0.10/0.05/0.05；fail均0|persistent簇×月 CGM|固定顺序第4；m=3 IUT|仅Gate1–3 PASS后；HOLDOUT一次|三者均PASS才PASS；任一FAIL才FAIL；其余INCONCLUSIVE|
|5 时钟|Gate 4 PASS 域；predictor=\(\tau_{confirm}\)|\(\tau_{confirm}\)|\(\beta_3\) at \(\tau_{confirm}\)|≤0|0.005 / 0|persistent簇×月 CGM|固定顺序第5，α=α_v|仅Gate1–4 PASS后；HOLDOUT一次|顶组/单调/Slope>0子样本=guardrail|
|6 选择|Track S；Eligible≥2 且 NonSelectedEligible≥2|主信号日 D|\(R^{arm1}_{net}-R^{arm2}_{net}\)（§18 完整净执行）|≤0|1% / 0|persistent簇×月 CGM|固定顺序第6，α=α_v|仅Gate1–5 PASS后；HOLDOUT一次|毛口径仅诊断|
|7 交互|Track S；匹配≤S−1；四臂信号=S；执行=S+1；NoCapacityStartThroughS|S|\(\Delta_{INT}\) 基于 \(R^{arm}_{net}\)|≤0|1% / 0|persistent簇×月 CGM|固定顺序第7，α=α_v|仅Gate1–6 PASS后；HOLDOUT一次|处理/控制同日选择器|
|8A 执行主|Population_fill（有效期内非零成交）|理论信号日|Conditional NetAlpha20|≤0|0.01 / 0|persistent簇×月 CGM|与 θ_ADV 组成第8门 IUT（m=2）|仅Gate1–7 PASS后；HOLDOUT一次|正式主；禁与8B混CI|
|8B 政策诊断|全部理论信号|理论信号日|PolicyNetAlpha20（未成交=0）|—|无独立 MES（强制报告）|—|不进 IUT / 不单独 PASS|与8A同开庭强制输出|DIAGNOSTIC_POLICY_RETURN|
|8 逆向选择|全部理论信号；隔离账户|理论信号日|\(-\theta_{ADV}\)|上界≥0.01 为逆|−0.01；fail 下界>0.01|persistent簇×月 CGM|并入第8门 IUT|同 Gate8|禁组合干扰|
|9 退出|Track S；固定20日 tape；DeployedCapitalBase=max并发名义|tape 首笔 fill 日所属月用于诊断；主装置仍为21日 block|动态−固定 Sharpe差、累计净超额差|任一≤0|ΔSharpe 0.10、ΔCumExcess 0.02；fail均0|同步21日 block bootstrap|固定顺序第9；m=2 IUT|仅Gate1–8 PASS后；HOLDOUT一次|资本分母唯一；PASS 仍需动态 §20.5(2–8)|
|S 叠加|端到端 overlay 完整政策 vs 无叠加完整政策（允许入口/退出序列不同）|政策比较窗起始月|叠加−无叠加的Sharpe差|≤0|0.10 / 0|月块|独立 Gate S 家族 Holm FWER α_v|HOLDOUT一次；FORWARD仅Kill|不影响主终局；FAIL/INCONCLUSIVE 不进入生产|

**index_date 规则（v12）：** `ID2 = YYYY-MM(index_date)`，必须回引本表，不得另选。

**MES 说明：** 以上为预注册工程/经济最小效应，不是市场自然常数。修改即新版本。复合 Gate 使用 §18.0 IUT，核心9 Gate 使用固定顺序 gatekeeping（每 Gate α=α_v，v12 为 0.025）；Gate S 才使用 Holm。所有效果量输出双侧95% CI，并输出正式单侧 \(1-\alpha_v\) 下界与反向 false-death 预算下的 Bonferroni 同时上界（\(1-0.05/(9m)\)）。

# 附录 D：正式变量字典索引

每个进入信号、状态、退出或 Gate 的变量在本表登记唯一公式位置；实现不得使用本表以外的口径（v7 新增，A-01 Closure Pack 1）：

|变量族|唯一公式位置|缺失/并列规则位置|
|---|---|---|
|r、x、行业收益|§6.1|§6.1（行业成员<5 缺失链）|
|TurnValue/TurnZ、CLV/CLV3、Active、NewJoin|§6.2–6.5|§6.2–6.5|
|Δ、Pctl、EMA、Hash/seed|§6.6–6.9|§6.7–6.9|
|ALS 六分量、L 集合|§7.1–7.3|§7.2 各分量注|
|Breadth/NewJoinRate/TurnBreadth/SignConcord/FailureRate（含涨停计数）|§8.2|§8.2|
|IES、IDS 两级、CONF、LeaderAmountShare|§8.4–8.6、§3.5|§8.4–8.5|
|ConfirmQuality、题材簇连通分量|§5.3|§5.3|
|null 生成器、load-or-build、DISCOVERY_RESEARCH_NULL/COHORT_D、SMD、NG、DAG/SLO|§8.3|§8.3.1–8.3.9|
|FillRate、正贡献份额|§20.4|§20.4|
|CCS 双轨、Defense、Persist、过度扩张|§9.2–9.3|§9.2.3|
|DirectedDemand、LDS/CDS/PriceShare/RMI/CapacityBreadth/HHI（E 族）、题材相对收益、成交额份额|§10|§10.6|
|GCC 两级、启动带、τ 事件|§11|§11.4|
|状态机、CLIMAX、COLLAPSE、COOLDOWN、关闭码|§12|§12.10|
|EntryScore、RS、信号权、有效期|§13、§18 Gate 6|§13.3|
|g、流式参与、顺序冲击单次计入、佣金预留、部分成交、费用|§14|§14.1/14.3/14.4|
|目标权重、预留、ThemeRoom、T+1、trade/tax lot、AvgCost、公司行动|§15|§15.0/15.2–15.5|
|LAC 五变量、LACDecay|§3.3|§3.3|
|FutureAttention/FutureCapacity、θ^ROLE|§17.2|§17.2|
|AUC、Whipsaw5、ΔStartRate/ΔCDS/ΔPriceShare、GrossAlpha20、θ_ADV、ΔSharpe|§17.1–17.4、§18|§17 通用观测规则|
|NetAlpha20、基准、暴露匹配|§0.3|§0.3.2|
|PSI、FalseSignalRate、AnnualFalseEpisodes|§19、§21.6|§19|

# 附录 C：Freeze 检查表

## 定义与实现
- [ ] A 类阻断 = 0；B 类阻断 = 0
- [ ] 确认量尺永冻；FrozenMetricBackcast 不回写确认 IDS；DISCOVERY_RESEARCH_COHORT_D 唯一回引
- [ ] P10 identity key + continuation/supersede/cooldown/spawn_index 唯一
- [ ] 合并 measurement 不合并且 ThemeRoom 不迁移
- [ ] 对照自真实 τ_entry 起执行；禁止提前建仓共享退出时钟
- [ ] Impact Single-Count；CommissionReserve 预算闭包；trade_lot/AvgCost/tax_lot；T+1 与公司行动开盘前
- [ ] Gate8A/8B 分离；隔离 no-gap；Gate9 DeployedCapitalBase 唯一；bootstrap=persistent component 整块
- [ ] 附录 B index_date + 8A/8B 行；§20.5 终局分支
- [ ] 全路径/manifest/函数名/spec_version = SFL-1_v12.0
- [ ] 不存在未解析参数
- [ ] `params.yaml` 数字 lint 通过

## 数据与 null
- [ ] DG01–DG16 全过
- [ ] NG01–NG07 全过
- [ ] P10 Phase-I 基线冻结
- [ ] 共同支持覆盖达标
- [ ] null 优化 DAG 通过 naive 等价与运行 SLO

## 统计
- [ ] 附录 B 十项完整
- [ ] §20.2 CGM / seed / guardrail 冲突规则已实现
- [ ] 三态判词压力测试通过
- [ ] Monte Carlo 精度闸门通过
- [ ] HOLDOUT 未触碰校验通过
- [ ] Track S 主法庭与 Track B 辅助 court 隔离

## 执行与治理
- [ ] 账本恒等式零失败
- [ ] Run-in 20日完成，并覆盖黄金路径：开盘涨停后开板、同日最低佣金、D+1买D+2追补下午卖、除权开盘前调整、payment_date 分红、RIGHTS 不行权、合并后 ThemeRoom 不迁移、隔离 no-gap、对照不早于真实 τ_entry 建仓
- [ ] Kill Criteria 与漂移基线冻结
- [ ] 单实现独立重发审计全链一致
- [ ] 联合哈希包签署并创建 FREEZE TAG
