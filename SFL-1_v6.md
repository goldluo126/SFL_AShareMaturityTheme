# SFL-1 v6.0 正式交易规格书

## 题材发现—确认分离 × 角色迁移 × 容量时钟 × 可实现交易

**英文代号：** Theme Discovery–Confirmation, Role Migration and Capacity Clock Framework
**简称：** SFL-TDCRMC
**版本：** v6.0 Canonical
**状态：** `FREEZE-CANDIDATE`；规范文本已自包含。仅在 §4 数据闸门、§8.3 生成器验收（NG01–NG07）、§17 机制研究闸门、§20.4-bis 可行性核查、§21.1 分区清单物化与附录 C 全部通过后方可冻结
**适用市场：** 中国A股，日频信号、分钟级执行、仅做多
**信号时钟：** D日收盘后计算，D+1执行
**研究目标：** 放弃预测底部；确认题材共同需求已经发生，识别边际需求从注意力载体向容量载体传播的早期阶段，赚取容量定价尚未完成的中段收益
**非承诺：** 本文是可回测、可复现、可证伪的研究与交易规格，不代表该策略已经被实证证明有效

## Canonical 与自包含声明

1. **唯一规范源：** 本文件是 SFL-1 v6.0 的唯一规范性来源。实现、回测、验收、影子运行和正式判词不得引用任何早期版本、审查报告、变更日志或口头解释补全规则。
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

> 在一个于D日前已经被定义的候选题材集合中，少数高显著性股票首先完成注意力点火；如果剔除这些股票和待买股票后，其他成员仍出现独立扩散，并且边际需求开始由低容量注意力载体向高容量核心载体迁移，而容量核心尚未完成主要价格扩张，则在按 §14.2 执行规则于信号有效期内可成交的条件下，D+1（或有效期内重试日）买入容量核心可能获得扣除真实成本后的条件超额收益。

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

- \(\tau_{entry}\)：实际成交日（D+1，或 §13.3 有效期内的重试成交日），成交价为该日 09:35—10:00 VWAP（§14.1）；
- 持有计数：\(\tau_{entry}\) 计为持有第 1 日；
- 入口验证阶段（§16.1）：第 20 个持有交易日收盘生成退出信号，第 21 个交易日 14:30—15:00 VWAP 卖出（顺延规则 §14.3）；
- \(R^{net}_i\) = 卖出净所得 / 买入净成本 − 1（全部费用与冲击按 §14.4）；
- 未在有效期内成交的信号不进入 NetAlpha20 样本；其占比与假想收益按 §20.6 一等读数强制报告（Gate 8 逆向选择对照防止该筛选掩盖坏规则）。

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
2. 假想买入：与真实信号同一 \(\tau_{entry}\)、同一 09:35—10:00 VWAP 窗口；
3. 机械拒绝规则逐股同构适用：停牌、09:30 开盘价等于涨停价、09:35—10:00 无成交、全部成交在涨停价且无法证明可成交（§14.2 前四条）；
4. REJECT_GAP：对照股使用与配对真实信号**相同状态**、**对照股自身所属交易板块**的同一张阈值表（§14.2）判定；
5. 被拒对照股从该篮子剔除；篮子收益 = 成交成员等权；成交成员为零的篮子从平均中剔除；
6. 卖出：与真实信号同一卖出日、同一 14:30—15:00 VWAP 窗口，顺延规则同构（§14.3）；成本规则同构（§14.4）；
7. \(C^{valid}\) = 成交成员非零的篮子集合；\(|C^{valid}| < 100\) → 该 episode 的 episode 层主终点记 `NON_EVALUABLE`，登记、不进入主样本、计入 §20.4 样本充分性核查。

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

在已确认题材内，容量时钟较高且仍在上升的股票，其未来20日净收益高于容量匹配对照。

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

**B-LEX：历史正式主模式。** 对每个公司，将截至 D−1 21:30 可用文本归一化后做字符 3–5 gram hashing：

- 输入必须是 ETL 以冻结 `parser_version` 生成并保存哈希的 `text/plain` UTF-8 字节；出现 HTML 标签或解码失败即隔离。依次执行 Unicode 15.0 NFKC、ASCII 字母小写化、Unicode General Category 以 P/C/Z 开头的 code point 替换为单空格、连续空白折叠、首尾去空白；不做简繁转换、分词或停用词删除；
- 在每个连续 CJK/ASCII字母/数字 token 两端加入 `^`、`$`，只在 token 内生成长度3、4、5的 Unicode code-point n-gram；不跨 token；
- document 唯一键为 `(company_id, source_url, published_at, content_hash)`；同公司相同 content_hash 只保留 available_at 最早的一条；
- 固定维度 \(M=2^{18}\)；对 n-gram 的 UTF-8 bytes 使用 MurmurHash3 x86_32、seed=0。令无符号32位结果为 h，index=`h mod M`，sign=高位 bit31 为0时 +1、为1时 −1；
- 文档桶 k 的 signed count 为 \(z_{d,k}=\sum sign(ngram)\)；\(TF_{d,k}=sign(z_{d,k})\log(1+|z_{d,k}|)\)；
- 截至 D−1 的去重文档数为 \(N_D\)，\(df_{k,D}=\sum_d\mathbf1(z_{d,k}\ne0)\)，\(IDF_{k,D}=\log((1+N_D)/(1+df_{k,D}))+1\)；
- 文档向量 \(v_{d,k}=TF_{d,k}IDF_{k,D}\) 后做 L2 归一化；零向量视为无有效文本；
- 公司向量为过去120日文本 TF-IDF 向量按 \(e^{-age/60}\) 衰减加权后 L2 归一化；
- \(SemanticSim\) 为公司向量 cosine similarity；无有效文本时记不可配对，不填0；
- 每个 D 的处理顺序固定为：先纳入 `available_at <= D-1 21:30` 的新增去重文档并更新 \(N_D,df,IDF\)，再以同一 \(IDF_D\) 重算受影响的120日公司向量；
- IDF 的文档频次与公司向量按日增量更新，缓存规则见 §8.3.8–§8.3.9。

B-LEX 不依赖预训练语义模型，允许覆盖具有 PIT 文本快照的完整历史，是 Track B 正式历史检验的默认口径。

**B-EMB：可选 measurement regime。** 使用登记 embedding 时，必须满足：

- `training_cutoff < regime_start`；
- 模型、分词器、pooling、最大长度和权重哈希在整个 regime 内固定；
- 只编码 `available_at <= D-1 21:30` 的文本；
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

Track B单独报告，不与Track S混合计算主判词。Track S 是主轨，可以独立通过并生产；Track B 样本不足时输出 `TRACK_B_INSUFFICIENT`，**不阻断 Track S**。只有 Track B 在自身共同支持域内通过 Gate 1–9，才允许从下一新 SPEC/regime 起与 Track S 合并；不得在当前评估样本中即时合并。

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

episode 合并时：

- 科学研究样本在合并日前一日右删失，两个原 episode 不产生合并后的 H-ROLE/H-MIG 观测；
- 生产运行集合取 \(B^E\) 并集；同股角色冲突时采用 \(\tau_{confirm}\) 更早 episode 的角色，仍并列时采用 episode_id 字典序较小者；
- 合并后集合只服务持仓状态和退出，不生成新的主信号。

### Live Active Core \(A_{u,t}\)

持仓期间每日更新的活跃核心。构造规则：

1. **种子：** \(A_{u,\tau_{confirm}} = E_{u,\tau_{confirm}}\)；
2. **每日纳入（对全市场股票 j）：** 同时满足
   - 以当日更新前成员 \(A_{u,t-1}\) 固定回看 t−20:t−1，j 与该成员集合每日等权行业残差收益序列的相关系数 \(\ge 0.50\)；
   - 过去10日内至少1日 \(Active_{j}=1\)（定义见 §6.4）；
   - 非ST、非停牌、非退市整理；
3. **每日移出：** 连续10个交易日不满足纳入条件(a)与(b)的股票；
4. **身份登记：** 每只成员记录 `joined_at` / `left_at` / `join_reason`，LAC 每日快照哈希归档。

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
|\(IDS^{\theta}_{u,D}\)（题材级）|题材|\(B\setminus L\)，不剔候选|状态机、Whipsaw5、Leg B2/X、§12 全部门槛|
|\(IDS^{-i}_{u,D}\)（候选级）|候选股|\(B\setminus(L\cup\{i\})\)|该候选入场资格、\(CONF^{-i}\)、P1|
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

信号只能读取：

\[
available\_at \le D\ 21{:}30\ \text{Asia/Shanghai}
\]

核心信号不依赖D日两融数据。龙虎榜 D 日 20:00 后发布的数据，available_at 记为 D 日 20:30，只允许进入 §17.5 的事后机制研究，与 D+1 决策在工程上物理隔离（独立库表、独立任务、信号任务无读取权限）。

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
  1. 语义相似度计算只可使用 \(published\_at \le D-1\) 且 \(available\_at \le D-1\ 21{:}30\) 的文本快照；
  2. B-LEX 的 hashing 维度、算法、seed、字符 n-gram 范围、IDF 截止日与状态哈希必须归档；未来文本置空后历史向量哈希不变；
  3. B-EMB 的 training cutoff 必须登记，且严格早于该 measurement regime 起点；不要求早于 B-LEX 历史样本起点；
  4. 每条文本须有可验证的公开来源与时间戳；无法验证 published_at 的语料不得进入 \(SemanticSim\)；
  5. B-EMB 模型权重/分词器/配置哈希变更即新 regime；禁止跨 regime 无条件池化；
  6. Track B 可评估 episode 少于120或不足3年时输出 `TRACK_B_INSUFFICIENT`，不以放宽 PIT 纪律换取样本。
- **DG14（行业与股本 PIT）：** 行业分类映射与自由流通股本/市值均按日（至少按公告生效日）快照保存；分类改版（如申万 2014/2021）与股本修订不得回填历史；行业残差 \(x_{i,t}\) 与 TurnValue 只可使用 t 日当时快照。历史快照不可恢复的区间，按 DG11 截断。
- **DG15（基准事实源）：** 组合层基准"全A等权指数"为自建：成分 = 当日满足 §5.1 前四条资格（A股普通股票、上市满120交易日、非ST/退市整理、当日正常交易）的全部股票，等权、每日再平衡、含退市股票至其最后交易日；构造代码与逐日成分哈希入冻结物。次要对照（沪深300、中证1000）须使用 PIT 成分。

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

容量约束：

\[
OrderValue_i
\le
\min(
0.5\%\times ADV20_i,
5\%\times VolumeValue^{09:30-10:00}_{i,D+1}
)
\]

如果目标资本未知，研究阶段按标准化1000万元组合计算，并同时报告容量曲线。

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

## 5.3 高重叠题材

同日题材Jaccard：

\[
J(A,B)=\frac{|A\cap B|}{|A\cup B|}
\]

若 \(J\ge0.60\)，归为同一题材簇。

- 入场只保留确认质量最高的代表题材；
- 相同股票不重复持仓；
- 统计推断以题材簇×episode为聚类单元。

## 5.4 episode 合并与分裂

### 合并

两个进行中 episode（\(u_1\) 的 \(\tau_{confirm}\) 更早）满足：

\[
J(A_{u_1,t}, A_{u_2,t}) \ge 0.60
\quad\text{连续2个交易日}
\]

则 \(u_2\) 合并入 \(u_1\)：

- 保留 \(u_1\) 的 episode_id 与全部事件时间；
- 成员集合取并集，Entry Cohort 各自冻结存档；
- \(u_2\) 的持仓并入 \(u_1\) 账务，其状态机与退出腿统一由 \(u_1\) 接管；
- \(u_2\) 登记 `MERGED_INTO = u_1`，统计推断按合并后单 episode 计。

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

行业映射按 DG14 的当日 PIT 快照。不使用包含候选股自身的题材指数计算"纯度"。

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
- REJECT_GAP 的经验分位 Q0.85（§14.2：\(Q_{0.85}\) 取满足 \(Pctl(g;S)\ge 0.85\) 的最小样本值）。

**离散变量并列处理：** Persist∈{0,1,2,3} 等离散变量直接按上式 midrank 处理并列，**不做任何缩放**。单调正变换不改变经验分位，因此不得以缩放替代并列规则。

被评对象 \(x\) 自身不进入参照集 \(S\)（自排除），除非该处显式声明池化定义。

## 6.8 EMA 初始化与最短观测

- 递归式：\(EMA_t = \alpha X_t + (1-\alpha)EMA_{t-1}\)，\(\alpha = 2/(span+1)\)（span=3 时 \(\alpha=0.5\)）；
- 初始化：\(EMA_{t_0} = X_{t_0}\)，\(t_0\) 为该变量序列在本 episode 观察窗内的首个可用日；
- 最短观测：\(\Delta_3 EMA3\) 需要不少于 4 个可用观测；不足时该变化量记为**不可判定**，凡以其为条件的判定一律按"条件不满足"处理（保守方向），并登记 `INSUFFICIENT_WINDOW`。

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

特征全部转化为题材内分位（Pctl 按 §6.7）：

- 点火时序；
- 当前连板高度；
- 5日累计行业残差收益；
- 3日成交额占题材份额；
- 3日正向残差收益贡献；
- 过去10日题材回调日的相对防守。

\[
ALS_{i,D}
=
Mean(
Pctl(EarlyIgnition),
Pctl(BoardHeight),
Pctl(CumX5),
Pctl(AmountShare3),
Pctl(PositiveReturnShare3),
Pctl(Defense)
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

### 题材级集合

\[
N^{\theta}_{u,D}
=
B_{u,D-1}\setminus L_{u,D}
\]

用于题材级变量（\(IDS^{\theta}\)），供状态机、Whipsaw5 与退出腿消费。

### 候选级 Leave-One-Out 集合

对于候选容量股票 \(i\)：

\[
N^{-i}_{u,D}
=
B_{u,D-1}
\setminus
(L_{u,D}\cup\{i\})
\]

该候选的入场确认只能使用该集合，防止候选股和龙头自我认证。

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

## 8.3 匹配伪题材生成器

### 8.3.1 排除集与候选池

对真实题材 \(u\)、日期 \(D\)：

\[
X_{u,D}
=
B_{u,D-1}
\cup
\bigcup_{v:\,J(v,u)\ge0.60} B_{v,D-1}
\cup
\bigcup_{\text{进行中 episode } e} E_{e}
\cup
\text{当日组合持仓股}
\]

候选池 \(P_{u,D}\) = §5.1 合格股票 \(\setminus X_{u,D}\)。伪题材成员不得与真实题材、同簇题材、任何进行中 episode 或持仓共享，防止 null 池被处理效应污染（donor–recipient 共享题材）。

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

### 8.3.3 生成算法（确定性）

对每个 \(c\in\{1,\dots,200\}\)：

1. 以 \(Seed=Hash(spec\_version,theme\_id,D,c)\) 初始化 PCG64 随机流；
2. 真实成员按（申万一级行业 × 交易板块）联合分层；
3. 层内按股票代码升序逐成员匹配：对真实成员 \(m\)，在 \(P_{u,D}\) 中同层、且未被本伪题材已用的股票中，按 7 维马氏距离（log自由流通市值、log ADV20、前20日累计收益、前20日日收益波动率、前20日TurnValue中位数、前20日涨停次数、前20日Active日数；协方差为当日全体合格股票的样本协方差，对角加 \(10^{-6}\) 正则）取最近 20 只，随机流等概率抽 1；
4. 同层候选不足 20 只时取全部；同层候选为空时放宽为同交易板块任意行业中最近 20 只（该次匹配登记 `RELAXED_MATCH`，比例入报告）；仍为空 → 该伪题材构造失败；
5. 同一伪题材内成员不放回；不同伪题材之间允许成员重复（跨篮子依赖在 §20.2 声明并由题材簇聚类吸收）。

### 8.3.4 可行性阶梯与失败处置

设成功构造的伪题材数为 \(n_c\)：

- \(n_c = 200\)：正常；
- \(100 \le n_c < 200\)：继续，登记 `PARTIAL_NULL(n_c)`；
- \(n_c < 100\)：该题材日记 `NON_EVALUABLE(NULL_INFEASIBLE)`：不产生任何 level 变量、不产生信号、状态机当日维持前一日状态，事件登记。

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

均值和方差均按上述权重计算。分母为0且均值相同则 SMD=0；分母为0且均值不同则 \(|SMD|=\infty\)。

类别 balance features 为每个实际出现的“申万一级行业”与“交易板块”独立 dummy。对类别 a：

\[
SMD_a
=
\frac{p^R_a-p^C_a}
{\sqrt{(p^R_a(1-p^R_a)+p^C_a(1-p^C_a))/2}}
\]

零分母规则同连续特征。成员数量必须与真实题材完全相等，不使用 SMD。对连续特征另报告 q10/q50/q90 差除以真实组 IQR 的诊断，但不进入主闸门。

任一连续或类别 \(|SMD|>0.25\)，或成员数不等 → 该题材日记 `NON_EVALUABLE(BALANCE_FAIL)`：不产生任何 level 变量、不产生信号、状态机当日维持前一日状态，并登记首个失败维度。全部 SMD 与诊断分位入 Gate 报告。

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

1. **StockDayFeatureStore（全市场每日一次）：** 计算并按 `(trade_date, stock_code, raw_snapshot_hash, feature_code_hash)` 缓存 r、x、TurnValue、TurnZ、CLV、Active、NewJoin、Rel 基础项和 §9.2 原始特征。滚动窗用 ring buffer 增量更新；结果必须与全窗口重算满足 §26.3 容差。
2. **TextFeatureStore（每条新文本一次）：** B-LEX 只增量更新 hash term frequency、expanding document frequency/IDF 与受影响公司120日衰减向量；B-EMB 只编码新增文本。每日公司向量必须与从该 regime 起全部 PIT 文本重算满足 §26.3 容差。
3. **DonorIndex（每日每层一次）：** 对行业×交易板块构建标准化向量、协方差和确定性近邻索引；同日所有真实题材共享。排除集在查询后施加，禁止为了共享索引而跳过 §8.3.1 排除。
4. **NullMembership（每真实题材日一次）：** 生成最多200个伪题材的 `recipient_member → donor_member` 对齐矩阵；IES、IDS、CCS、GCC、Gate 对照和 P10 必须共享这一冻结矩阵，不得各自重新抽样。
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

IES 为题材级变量（owner=题材）。

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
- 满足组合订单容量；
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

每个 \(K^E\) 成员的原始特征先对匹配伪题材标准化：取该真实题材日 200 个匹配伪题材（§8.3）在其伪确认日冻结的容量角色作为 null 池，按 §6.7 的 midrank Pctl 计算经验分位：

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

- \(\tau_{confirm}\)：首次独立确认日（题材级：IES 与 \(IDS^{\theta}\) 达标）；
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

多状态条件同日并存时由优先序唯一裁决；该题材日为 `NON_EVALUABLE`（§8.3.4/8.3.5）时，状态维持前一日值并登记。

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

- 自清算完成次一交易日起计数 10 个交易日；
- 冷却期内该题材及其同簇题材（\(J\ge0.60\)）**不得确认新 episode**、不得产生任何入场信号；
- 冷却期满：episode 关闭归档，题材回 DORMANT，新 episode 可从 DISCOVERED 重新评估。

本节是 episode 关闭与重新点火的唯一合法路径。

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

**分位基准：** \(Pctl_{cross}\)（按 §6.7）为同日全部处于 DISCOVERED 及以后状态题材的横截面分位；题材级变量（\(RMI^E\)、\(GCCSlope^{\theta}\)）每题材日只贡献一个观测，取题材间分位后映射到其候选股，避免同题材候选数改变横截面权重。\(CCS\_rank\) 为组内排序成分（§9.2.2），\(CONF\) 本身已是 level 尺度。

这些权重是冻结前登记的工程权重，不通过网格优化。敏感性仅比较等权版本，不选择较优者替换主口径。

每题材选择最多2只。

并列顺序：

1. CCS_rank高；
2. ADV20高；
3. 股票代码升序。

## 13.3 同一episode规则与信号有效期

- 首次入场后不加仓；
- 平仓后不在同一episode重新入场；
- **信号有效期：** D 日信号在 D+1 执行；若因 §14.2 拒绝、§15 组合拒绝（REJECT_PORTFOLIO_FULL / REJECT_MIN_WEIGHT）或未成交，允许在 D+2、D+3 重试，每个重试日开盘前重新校验：题材状态仍为 EARLY_CAPACITY/HEALTHY_TRANSFER、个股未触发过度扩张否决、CONF 仍达标。D+3 收盘仍未成交，信号作废；
- **入场权消耗：** 每只候选股在一个 episode 生命周期内只有一次主信号入场权；信号成交或作废（含有效期耗尽）均消耗该权利。**组合拒绝不提前消耗入场权**（有效期内仍可重试；有效期尽则作废并消耗）。episode 入场归因以信号日 D 计，实际成交日记为 \(\tau_{entry}\)，二者差异逐笔登记并纳入执行分析；
- 未成交不改写状态史：episode 状态史的首次进入日以状态机实际首次进入日计，与成交无关；
- episode 结束按 §12.8–12.9 闭环（COLLAPSE 清算 + COOLDOWN 期满，或 Leg C 到期平仓后进入 COOLDOWN 同规则）；
- 新episode可重新评估（受 §12.9 冷却约束）。

---

# 14. 执行规则

## 14.1 主执行价格

正式交易与正式回测均使用：

    实际执行日 09:35—10:00 VWAP

首个执行日为 D+1；重试执行日为 D+2、D+3。

原因：

- 避免将开盘集合竞价中无法确定的排队成交全部视为可得；
- 避免零延迟使用9:30瞬时价格；
- 能用分钟线复现。

## 14.2 买入拒绝

满足任一项则不成交：

- 09:30—10:00期间停牌；
- 09:35—10:00无成交；
- 09:30开盘价等于涨停价；
- 09:35—10:00全部成交均在涨停价且无法证明订单可成交；
- 预计订单超过第5.1节容量限制；
- **REJECT_GAP：** 信号条件跳空超过阈值，定义为

\[
g_{i,t}
=
\frac{VWAP^{09:35-10:00}_{i,t}}{Close_{i,t-1}}-1
\]

拒绝规则：

    g > Q0.85(g | 历史同状态信号, 同交易板块)

其中 Q0.85 按 §6.7 唯一定义。阈值取自历史同状态（EARLY_CAPACITY / HEALTHY_TRANSFER 分别统计）、同交易板块信号的条件跳空经验分布，随样本滚动更新并逐日归档版本。

**冷启动兜底：** 同状态同板块累计信号数 < 40 时，使用临时阈值：主板 0.06，创业板/科创板 0.09，北交所 0.12（固定冷启动先验，入 §27 参数表，样本达标后自动切换为经验分位，切换事件登记）。

**不顺延追买：** 因 REJECT_GAP 未成交的信号按 §13.3 在 D+2、D+3 重试，重试日重新计算 g 与拒绝判定；不允许以提高限价的方式追买。

记录为：

    UNFILLED_LIMIT
    UNFILLED_SUSPENSION
    REJECT_GAP
    REJECT_CAPACITY
    REJECT_PORTFOLIO_FULL
    REJECT_MIN_WEIGHT

开盘涨停后盘中开板，不允许回填为开盘价成交。

## 14.3 卖出执行

D日收盘后形成退出信号：

    D+1 14:30—15:00 VWAP卖出

若：

- 一字跌停；
- 停牌；
- 14:30—15:00无可成交量；

逐日顺延至首个可执行窗口。

顺延期间继续计入净值和风险。

## 14.4 费用与冲击

所有法定与经纪费用从按日期版本化的 `fee_schedule.yaml` 读取。该文件属于 PIT 输入事实，其唯一 schema 为：

|字段|类型|含义|
|---|---|---|
|effective_from / effective_to|交易日（闭区间）|费率生效区间，不得重叠|
|board|枚举|MAIN / CHINEXT / STAR / BSE / ALL|
|side|枚举|BUY / SELL / BOTH|
|commission_rate|非负小数|经纪佣金率|
|minimum_commission_cny|非负人民币元|每笔母订单最低佣金|
|stamp_tax_rate|非负小数|印花税率|
|transfer_fee_rate|非负小数|过户费率|
|exchange_fee_rate|非负小数|交易所及监管费率合计|
|source_url / published_at / available_at|字符串/时间戳|官方或经纪合同事实源与 PIT 时钟|
|source_hash|SHA-256|事实源内容哈希|

对每笔母订单（同股票、同方向、同执行窗口的分钟成交合并）：

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

\(\sigma_{20}\) 使用日收益率小数尺度；乘以 \(10^4\) 后容量冲击单位为 bps。买入成交价乘以 \(1+ImpactBps/10^4\)，卖出成交价乘以 \(1-ImpactBps/10^4\)。敏感性固定报告 `fixed_slippage_bps ∈ {3,10}` 与 `lambda ∈ {0.25,1.00}`，不得择优替换主值。

必须分别输出：

- 佣金；
- 印花税；
- 交易所及过户费用；
- 固定滑点；
- 容量冲击；
- 未成交机会损失；
- 总净成本。

---

# 15. 组合构建与交易账本

## 15.0 组合硬约束

- 每只股票初始上限：10%；
- 每题材簇上限：20%；
- 同时持有题材簇上限：3；
- 总风险仓位上限：60%；
- 无合格信号时持有现金；
- 不强制满仓；
- 同一股票属于多个题材时只持有一份；
- 组合已满时后续信号进入有效期重试，不跨有效期排队。

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

- D+1 因执行拒绝或组合拒绝未成交后，为该信号预留其当日 TargetValue 至下个执行窗口；
- 每日收盘后按最新 NAV 与硬约束重算预留，取原预留与新 TargetValue 较小者；
- 预留资金计现金、不计风险仓位，但不得分配给其他信号；
- 信号成交、作废、状态复核失败或有效期耗尽时立即释放预留；
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

    CREATED → RESERVED → SUBMITTED → FILLED / REJECTED / EXPIRED / CANCELLED

每次转移保存时间、原因、前后现金、预留、仓位与槽位快照哈希。

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
    下一交易日14:30—15:00 VWAP执行

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

### Leg H：健康交接保护

若注意力核心断板或走弱，但同时满足：

    IDS^θ 下降不超过0.10
    RMI^E > 0
    CapacityBreadth^E >= 0.40
    GCCSlope^θ >= 0

不减仓。

该规则防止将龙头向容量载体的健康迁移误判为退潮。Leg H 只覆盖“龙头走弱”，不得覆盖 COLLAPSE 状态——状态机已进入 COLLAPSE 时 Leg X 无条件优先。

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

Leg H是保护条件，不是卖出腿。

---

# 17. 必须先完成的机制研究

在正式组合回测前，先做事件研究。未经机制闸门，不得直接跑端到端策略并解释市场故事。

## 17.1 Study 1：发现—确认分离研究

### 样本

每个D−1已经存在、且满足 §8.3 共同支持的题材候选。

### 处理组

D日满足候选级独立确认。

### 控制组

同日匹配，但未满足确认的题材。

### 主指标

- 未来5日非龙头NewJoin AUC；
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

- **Gate 1 / H-DISC：** 真实 D−1 候选题材的未来5日 NewJoin AUC − 匹配伪题材 AUC 按 Gate 1 MES/CI 裁决；
- **Gate 2 / H-CONF：** 已确认处理组与同日未确认匹配题材的 Whipsaw 率差（控制−处理）按 Gate 2 MES/CI 裁决，MES=10个百分点；
- 未来5日 \(IDS^\theta\) AUC 为 Gate 1/2 的强制次要机制读数，不替代上述唯一主 estimand；
- LOO后仍成立；
- Track S和Track B分别报告。

## 17.2 Study 2：角色可分离性研究（H-ROLE）

本研究独立回答：注意力角色 \(L\) 与容量角色 \(K\) 是否具有不同的未来市场功能，而不是同一涨幅排序的两个名称。为避免定义自证，所有主 outcome 只使用 D+1:D+5 数据，不使用构造 D 日角色的变量值。

### 个股未来功能分数

对 \(i\in L^E\cup K^E\)，每个分量均对 §8.3 匹配伪题材中同资格股票取 §6.7 midrank 分位：

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
- `MorningExecutableValueShare5`：D+1:D+5 每日 09:35–10:00 非涨停可成交金额占该股票 ADV20 的比例之和。

### 两个共同主 estimand

\[
\theta^{ROLE}_{A}
=
Mean_{i\in L^E}(FutureAttention_i)
-
Mean_{i\in K^E}(FutureAttention_i)
\]

\[
\theta^{ROLE}_{C}
=
Mean_{i\in K^E}(FutureCapacity_i)
-
Mean_{i\in L^E}(FutureCapacity_i)
\]

### 判读

- Gate 3 PASS：两个 estimand 的单侧95% CI下界均 > 0.05，复合 p 值按 §18.0 IUT 定义；
- Gate 3 FAIL：任一 estimand 的 Bonferroni 单侧97.5%上界 < 0；
- 其余为 INCONCLUSIVE；
- P5 角色随机化后两个 contrast 应回到0附近；
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
- 容量组未来20日净异常收益高于容量匹配随机组；
- 结果在剔除最大市值股和最高成交股后仍存在。

## 17.4 Study 4：容量时钟事件研究

仅在已确认题材中，将候选容量股按 \(GCC^{\theta}\) 和CCS_level分组。

### 主检验

\[
Return20
=
\alpha
+\beta_1 CCS\_level
+\beta_2 GCC^{\theta}
+\beta_3 CCS\_level\times GCC^{\theta}
+\gamma'Controls
+\epsilon
\]

**Controls 冻结清单：**

    log(FreeFloatMV_D)
    log(ADV20_D)
    CumReturn20_{D-20:D-1}
    Volatility20_{D-20:D-1}
    TurnValueMedian20_{D-20:D-1}
    BoardFixedEffects
    CalendarMonthFixedEffects

禁止增删控制项；任何修改即新 SPEC。全部连续控制项按 VALIDATION 区均值/标准差标准化，参数冻结后同尺度应用 HOLDOUT/FORWARD。

核心是：

\[
\beta_3>0
\]

### 通过条件

- CCS_level×\(GCC^{\theta}\) 交互项按 Gate 5 MES/CI 规则通过；
- 顶组净收益高于容量匹配随机组；
- 四分位收益具有预期单调性；
- 候选股自身从候选级 \(GCC^{-i}\) 中剔除后的入场结果仍成立。

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

- **固定顺序：** Gate 1→2→…→9；Gate g 仅在全部前序 Gate PASS 后开庭。固定顺序 gatekeeping 在每 Gate \(\alpha=0.05\) 下控制核心家族强 FWER≤5%；
- **单一 estimand Gate：** \(p_g\) 为检验 \(H_0:\theta_g\le MES^{pass}_g\) 的单侧 p 值；
- **复合 Gate：** 对 m 个共同主 estimand 分别得到 \(p_{g,k}\)，使用 intersection-union test：\(p_g=\max_k p_{g,k}\)。Gate 3 的 m=2；Gate 4 的 m=3；Gate 8 的 m=2；Gate 9 的 m=2；
- **PASS：** \(p_g<0.05\)，等价地所有共同主 estimand 的单侧95%下界均高于各自 \(MES^{pass}\)；
- **FAIL：** 对反向命题 \(\theta_{g,k}<MES^{fail}_{g,k}\)，使用 Bonferroni 同时上界（单侧置信度 \(1-0.05/m\)）；任一共同主 estimand 的该上界低于其 \(MES^{fail}\)；
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

容量需求份额和容量广度在确认后相对匹配 null 出现独立、稳定的上升路径；\(\Delta StartRate_5\) 为共同主读数之一。按附录 B 三态裁决。

FAIL：

    FAIL_ROLE_MIGRATION

## Gate 5：容量时钟有效

\(GCC^{\theta}\) 和CCS_level交互能够预测未来20日净收益；必须同时报告启动带 [0.55, 0.85] 的敏感性（带边界 ±0.05 平移）。按附录 B 三态裁决。

FAIL：

    FAIL_CAPACITY_CLOCK

## Gate 6：容量载体选择特异性

同一题材、同一入场日比较：

- **生产 EntryScore 前 2**；
- 容量匹配随机2只；
- 全部可交易容量股等权；
- 题材内RS前2。

主统计：

\[
Alpha20_{EntryScore}
=
R^{net}_{EntryScore}
-
R^{net}_{MatchedRandom}
\]

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

\[
\Delta_{INT}
=
(A-B)-(C-D)
\]

按附录 B 三态裁决。FAIL：

    FAIL_FILTER_INTERACTION

## Gate 8：交易可实现

加入分钟执行、涨跌停、未成交和全部成本后，主终点按附录 B 三态裁决。

**强制逆向选择对照：** 被 REJECT_GAP 拒绝的信号，假想以拒绝日 09:35—10:00 VWAP 成交，计算其 NetAlpha20 并与实际成交组比较：

- 定义 \(\theta_{ADV}=NetAlpha20_{rejected}-NetAlpha20_{filled}\)，非劣界为1个百分点；
- guard PASS：\(\theta_{ADV}\) 的单侧95%上界 < 0.01；
- guard FAIL：\(\theta_{ADV}\) 的单侧95%下界 > 0.01，记 `FAIL_ADVERSE_SELECTION`；
- 其余记 `INCONCLUSIVE_ADVERSE_SELECTION`，REJECT_GAP 不得进入生产（保守禁用），Gate 8 可继续以“无 REJECT_GAP”规则重新形成预注册生产候选；
- Gate 8 的两个共同主 estimand 为执行 NetAlpha20 与 \(-\theta_{ADV}\)，按 §18.0 IUT 合成；
- 该对照在 VALIDATION 与 HOLDOUT 各报告一次。

FAIL：

    FAIL_EXECUTION

## Gate 9：退出增量

动态退出相对固定20日持有改善风险调整后净收益。**两政策使用完全相同的入场信号与入场成交序列，只允许退出规则不同；**否则不构成退出增量识别。走廊测量为强制输入。按附录 B 三态裁决。

FAIL：

    FAIL_EXIT_INCREMENT

退出失败不推翻入口，可退回固定持有或另立退出项目。

## Gate S：市场状态叠加层

评估 §15.4 三个预注册叠加规则：叠加开/关的 NetAlpha20、最大回撤、夏普对比，Holm 校正后三态裁决；PASS 方可进入生产；FAIL 或 INCONCLUSIVE 均不进入。Gate S 不影响入口与退出判词。

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

在每个题材的 \(L^E\cup K^E\) 内保持 \(|L^E|\) 与 \(|K^E|\) 不变，使用 §8.3 seed 规则进行5,000次无放回标签置换；每次重算 \(\theta^{ROLE}_A,\theta^{ROLE}_C\)。真实双 contrast 必须优于随机标签，用于 Gate 3 的匹配集合内随机化检验。不得以角色构造资格重新筛选置换标签。

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

将 §8.3 匹配伪题材作为“假真实题材”，运行从发现后确认、角色识别、容量时钟到主信号产生的完整链条（不交易真实资金），估计：

- 每日/每年假 IGNITION 数；
- 每日/每年假 CONFIRMED_EXPANSION 数；
- 每日/每年假主信号数；
- 各板块、牛熊状态、题材规模层的假点火率；
- 假信号的 GCC/IDS/CCS_level 分布。

### 假信号率唯一定义

对 control c：

- `P10_EVALUABLE(c,D)=1` 当其交叉拟合 reference controls 不少于100、通过同构 SMD 闸门且完整链无缺失；
- `P10_FAKE_SIGNAL(c,D)=1` 当且仅当该假真实题材在 D 日完整链产生至少一只主信号；同题材多股仍计1。

\[
FalseSignalRate(W)
=
\frac{\sum_{D\in W}\sum_c P10\_FAKE\_SIGNAL(c,D)}
{\sum_{D\in W}\sum_c P10\_EVALUABLE(c,D)}
\]

分母为0的窗口记 `P10_INSUFFICIENT`，不填0。年化假 episode 数：

\[
AnnualFalseEpisodes_y
=
\sum_{D\in y}
FalseSignalRate(\{D\})\times RealEvaluableThemes_D
\]

其中 `RealEvaluableThemes_D` 为 D 日通过 §8.3 可行性和平衡闸门的真实题材数。板块分层按题材成员数最多的交易板块（并列取板块代码升序），规模层按 VALIDATION 冻结的成员数 tertile。

Phase-I 99% 预测上界：在 VALIDATION 日级 `(fake_count,evaluable_count)` 上按自然月做10,000次有放回 block bootstrap；每次计算全部连续28交易日 pooled ratio 的最大值，取这些最大值的99%分位。总体与每个分层（累计 evaluable≥500）分别冻结；FORWARD 任一有效层超过其上界即 `NULL_RATE_DRIFT`。

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

## 20.2 主推断装置与依赖结构

**正式主装置：** 题材簇 × 自然月双向聚类稳健标准误，由其生成 Gate 主效果 CI 与 p 值。原因：同时覆盖同簇重复题材与同月市场共同冲击。

**强制稳健性：**

- 5,000次题材簇 episode 块bootstrap；
- 匹配集合内随机化检验；
- 按信号日历周聚类替代自然月的敏感性。

三者冲突时以主装置裁决，但冲突必须标 `INFERENCE_CONFLICT` 并在首页披露；若主装置 PASS 而两项稳健性均方向相反，降级为 INCONCLUSIVE。

同一题材多只股票不能当作独立样本。合并后的 episode 按单单元计。伪题材间允许 donor 复用导致基准相关，该依赖由真实题材簇聚类单元吸收，不把200个篮子当独立样本。

**Monte Carlo 精度闸门：** bootstrap/随机化检验使用不少于5,000次；若 Gate 固定顺序阈值0.05（或 Gate S Holm 阈值）两侧的 Monte Carlo 95% 二项置信区间跨越决策阈值，自动增加到50,000次；仍跨越则 INCONCLUSIVE。

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

上述9个正式 Gate 使用 §18.0 固定顺序 gatekeeping；每个复合 Gate 使用 IUT，核心家族强 FWER≤5%。不得在前序未 PASS 时计算或解释下游正式 p 值。

Gate S 在核心 Gate 完成后独立评估三个 overlay 候选，使用 Holm FWER 5%；Study 5 为 EXPLORATORY，不进入正式家族。

探索结果必须标记：

    EXPLORATORY

不得在当前评估样本中升级为主规则。

## 20.4 样本充分性

满足任一项则判定：

    INSUFFICIENT_EVIDENCE

条件：

- 可评估独立episode少于120；
- 样本少于3个自然年度；
- 单一年度贡献超过总净收益50%；
- 单一题材簇贡献超过总净收益20%；
- 实际可成交订单少于理论订单70%；
- `NON_EVALUABLE` 题材日超过所有合格题材日30%；
- 任一正式 Gate CI 因有效题材簇少于30个而不可估计。

## 20.4-bis DESIGN 期信号频率可行性核查

此核查只回答“按当前信号定义是否有可能在墙上时间内获得足够样本”，不得使用收益方向调阈值。

- 在 DESIGN 全区运行完整信号链，报告年化独立 episode 数及其 Poisson 95% 区间；
- 若年化点估计 < 40 或区间上界 < 60，当前设计记 `INFEASIBLE_SIGNAL_RATE`，不得进入 VALIDATION；可修订信号定义，但必须创建新 SPEC 版本；
- 若 40–60，记 `LOW_SIGNAL_RATE`，允许进入 VALIDATION，但必须在项目墙上时间上限内证明可满足 §20.4；
- 核查只使用信号数量，不读取任何未来收益、方向或 Gate 效果。

## 20.5 经济门槛

固定20日入口政策取得 `PASS_ENTRY_ONLY` 资格同时要求：

1. episode 层 NetAlpha20 按附录 B Gate 8 的 PASS 规则通过；
2. 固定20日完整组合对**暴露匹配基准**的年化净超额不低于5%；
3. 至少60%的自然年度暴露匹配超额为正；
4. 最大回撤不超过同风险暴露基准回撤1.25倍；
5. 成本翻倍后累计净超额仍为正；
6. 剔除最佳5%episode后累计净超额仍为正；
7. 实际成交率不低于70%；
8. §15.3 账本恒等式零失败。

最终 `PASS` 还要求 Gate 9 PASS，且动态退出完整组合独立重复满足上列第2–8项；否则终局按 §22。以上是工程验收门槛，不是市场自然常数；修改即创建新SPEC版本。

## 20.6 一等读数清单

以下读数与 NetAlpha20 同列主报告首页：

- 走廊测量全表；
- 执行拒绝分解；
- REJECT_GAP 逆向选择对照；
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

1. `USABLE_START`：DG01–DG15 全部可满足且具备最长前视无关回溯窗（250 个交易日）的首个交易日；
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
- 真实主信号率、成交率、各拒绝率；
- Track S/B 占比、题材成员数分布；
- LACBreadth、LACTurnBreadth、LACRelReturn5、LACAttrition10 与 LACDecay 触发率；
- DAG 节点耗时、cache hit、COMPUTE_TIMEOUT 率与重发审计耗时。

### 漂移触发

任一项触发 `QUARANTINE`：

1. 28日滚动 P10 假信号率 > Phase-I 同层99%预测上界；
2. 任一核心变量分布相对 Phase-I 的 PSI > 0.25，连续5日；
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
- 连续10个交易日成交率 < 30%；
- 实际组合回撤 > 15%（相对 FORWARD 初始 NAV）。

处置：立即停止新入场、按冻结退出规则清算；判词 `KILL_SAFETY`。该判词是风险治理决策，不宣称科学机制为假。

### 绩效 Kill（仅在满足 §21.5 后季度评估）

- FORWARD episode 层 NetAlpha20 的单侧95% CI上界 < 0；或
- 暴露匹配累计净超额 < −5%，且过去连续两个季度均为负；或
- 成本翻倍压力下累计净超额 < −10%。

处置：`KILL_ECONOMIC_FAIL`，停止新入场并清算。若机制指标正常而收益失效，登记 `ALPHA_CROWDING_CANDIDATE`；若机制指标同步失效，登记 `MECHANISM_DECAY_CANDIDATE`。二者均不得在同一 regime 内调参复活。

### 仪器漂移

按 §21.6 先 QUARANTINE；不可恢复时 `KILL_MEASUREMENT_REGIME`。不将仪器失败误写为策略科学 FAIL。

---

# 22. 终局判词

    INVALID_DATA
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
    KILL_MEASUREMENT_REGIME

报告标记（不覆盖上述唯一终局）：

    TRACK_B_INSUFFICIENT
    LOW_POWER_CLIMAX
    INCONCLUSIVE_EXIT
    OVERLAY_PASS / OVERLAY_REJECTED / OVERLAY_INCONCLUSIVE

逐层终止：

    Gate 0失败 → INVALID_DATA
    任一账本闸门失败 → LEDGER_INVALID
    Gate 1–8 任一 INCONCLUSIVE / 样本装置失败 → INSUFFICIENT_EVIDENCE
    Gate 1–8 任一有统计支持的 FAIL → 对应 FAIL_*
    Gate 1–8 全部 PASS，但固定20日入口政策未通过 §20.5 → FAIL_ECONOMIC_GUARDRAIL
    Gate 1–8 与固定20日入口政策通过，Gate 9 PASS → PASS
    Gate 1–8 与固定20日入口政策通过，Gate 9 FAIL → PASS_ENTRY_ONLY，并附 FAIL_EXIT_INCREMENT
    Gate 1–8 与固定20日入口政策通过，Gate 9 INCONCLUSIVE → PASS_ENTRY_ONLY，并附 INCONCLUSIVE_EXIT
    Gate S 只产生 OVERLAY_* 报告标记；仅 OVERLAY_PASS 可启用叠加层，不改变主策略终局

不能用下游失败否定尚未单独失败的上游理论。**FAIL 是有反向证据的判词，不是“不够显著”的别名。**

---

# 23. 工程目录

    /sfl1_v6/
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

    D−1 21:30
      └─ 冻结D日可观察的Base Universe

    D 15:00以后
      ├─ 落地日线、分钟线、交易状态
      ├─ 增量更新 StockDayFeatureStore 与 DonorIndex
      ├─ 更新持仓 episode 的 LAC 与五个动态健康变量
      ├─ 更新 episode 合并/分裂
      ├─ 识别注意力角色
      ├─ 构造一次 NullMembership 与 PseudoThemeStats
      ├─ 计算题材级 IDS^θ、GCC^θ
      ├─ 用精确 sufficient-statistics LOO 计算 IDS^-i、GCC^-i
      ├─ 计算 RMI^E、CCS_rank/CCS_level、EntryScore
      ├─ 更新唯一状态机（含 LACDecay 独立 COLLAPSE 分支）
      ├─ 生成信号；更新有效重试信号
      ├─ 计算退出腿
      └─ 运行 DG/NG、SLO、漂移与账本检查

    D 21:00
      └─ 未完成题材 COMPUTE_TIMEOUT；超10%则全日 QUARANTINE

    D 21:30
      └─ 信号、预留、退出订单、SHA256只增不改

    D+1 09:35—10:00
      ├─ 新/重试订单按统一排序分配
      ├─ 执行买入并记录拒绝
      └─ 更新现金、预留与仓位

    D+1 14:30—15:00
      └─ 执行前一日退出订单

    D 20:30以后（物理隔离）
      └─ 龙虎榜仅入 diagnostics

    D+1 12:00以前（独立审计环境）
      └─ 从只读原始快照重发 D 日全链并比较逐层哈希

---

# 25. 核心伪代码

    for D in trading_days:
        stock_features = update_stock_day_feature_store(D)
        donor_index = build_daily_donor_index(stock_features, D)
        themes = load_base_universe_asof(D - 1)

        # 持仓后动态传感器先更新，供当日 COLLAPSE 独立分支使用
        update_live_active_core(open_episodes, D)
        update_episode_structure(open_episodes, D)
        lac_health = compute_lac_health_and_decay(open_episodes, D)

        # 题材级阶段：每题材每日恰好一次
        for theme in deduplicate_theme_clusters(themes, D):
            base = eligible_base_members(theme, D)
            if not pass_theme_gate(base, D):
                continue

            provisional_leaders = identify_attention_roles(base, D)
            provisional_capacity = identify_capacity_candidates(
                base - provisional_leaders, D
            )

            nulls = build_matched_nulls_v6(
                theme, D, n=200, donor_index=donor_index,
                cache_key=canonical_cache_key(theme, D),
            )
            if len(nulls) < 100 or not pass_balance_gate(nulls):
                hold_state_and_log_non_evaluable(theme, D)
                continue

            pseudo_stats = aggregate_pseudo_theme_stats_once(nulls, D)
            ids_theme = independent_diffusion(
                members=base - provisional_leaders,
                nulls=pseudo_stats, owner="theme",
            )

            ies = ignition_evidence(
                theme, provisional_leaders, pseudo_stats
            )
            role_cohort = load_or_freeze_role_cohort_on_first_confirmation(
                theme=theme, date=D, base=base,
                leaders=provisional_leaders,
                capacity=provisional_capacity,
                ies=ies, ids_theme=ids_theme,
            )
            if role_cohort.status == "NO_ROLE_COHORT":
                update_discovery_confirmation_state_only(theme, ies, ids_theme)
                continue
            if not role_cohort.is_frozen:
                continue

            capacity_trade = current_trade_eligible_subset(
                role_cohort.K_E, D
            )
            ccs_levels = carrier_scores_level(
                role_cohort.K_E, pseudo_stats
            )
            gcc_theme = median(ccs_levels.values())
            rmi = role_migration_index(
                role_cohort.B_E, role_cohort.L_E, role_cohort.K_E, D
            )

            state = lifecycle_state_once_per_theme(
                theme, ies=ies,
                ids_theme=ids_theme, gcc_theme=gcc_theme, rmi=rmi,
                lac_decay=lac_health.get(theme).decay_if_held,
                priority=PARAMS.state_priority,
            )

            if first_entry_window(theme, state):
                for stock in capacity_trade:
                    ids_loo = exact_loo_from_sufficient_stats(
                        pseudo_stats, removed_recipient=stock,
                    )
                    gcc_loo = exact_delete_one_median(ccs_levels, stock)
                    if pass_candidate_entry(
                        stock, ids_loo, gcc_loo, ccs_levels[stock], D
                    ):
                        emit_signal(valid_until=D + 3)

        emit_exit_orders(open_episodes, D)
        reserve_and_allocate_cash(signals, retries, PARAMS)
        assert_ledger_identities()
        enforce_compute_slo_or_quarantine(D)
        run_drift_checks()

    fills = execute_vwap(orders)
    update_reservations_and_positions(fills)
    assert_ledger_identities()
    verdict = three_state_gate_evaluation()

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
|样本episode/年度/成交率|120/3/70%|—|工程|—|
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

SFL-1 v6.0采用的弱因果表述：

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
    冻结物不能只有名字，Kill 与漂移规则必须有完整内容。

---

# 附录 A：Canonical 闭包索引

本索引是本文自包含性的审计入口，不引入任何新规则：

|规范域|唯一正文位置|闭包内容|
|---|---|---|
|经济命题、识别边界、Estimand|§0–§2|可证伪机制、共同支持域、NetAlpha20、两层基准|
|发现、集合与变量 owner|§3、§5|Track S/B、B/E/A 三集合、题材/候选/组合归属|
|数据与 PIT|§4|字段、可用时钟、DG01–DG15、失败判词|
|统一算子|§6|收益、TurnZ、CLV、Active、Pctl、EMA、缺失窗口|
|角色、确认与 null|§7–§8|ALS/L、两级 IDS、伪题材生成器、等价 DAG、平衡与 NG 验收|
|容量、迁移与时钟|§9–§11|CCS rank/level、\(RMI^E\)、\(HHI^E\)、两级 GCC|
|状态与事件生命周期|§3.3、§12|冻结迁移与 LAC 动态健康分轨、固定优先序、吸收态、COOLDOWN|
|入场、执行与账本|§13–§16|信号权、VWAP、拒绝、成本、权重、现金预留、退出|
|机制研究与正式 Gate|§17–§20、附录 B|研究设计、安慰剂、三态法庭、MES、主推断|
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

|Gate|Population / Sample|Estimand \(\theta_g\)|Null|MES_pass / MES_fail|Dependence|Multiplicity|Sequential|Decision / Failure state|
|---|---|---|---|---|---|---|---|---|
|1 发现|共同支持域内 D−1 候选题材日|未来5日 NewJoin AUC：真实−null|≤0|0.05 / 0|题材簇×月|固定顺序第1，α=.05|HOLDOUT一次；FORWARD仅Kill|§18.0；不足→INCONCLUSIVE|
|2 确认|可评估确认/未确认匹配题材|Whipsaw率：控制−处理|≤0|0.10 / 0|题材簇×月|固定顺序第2，α=.05|仅Gate1 PASS后；HOLDOUT一次|§18.0；不足→INCONCLUSIVE|
|3 角色可分离|已确认且 \(L^E/K^E\) 均非空的 episode|\(\theta^{ROLE}_A,\theta^{ROLE}_C\)|任一≤0|各0.05 / 0|题材簇×月|固定顺序第3；m=2 IUT|仅Gate1–2 PASS后；HOLDOUT一次|两者均PASS才PASS；任一FAIL才FAIL；其余INCONCLUSIVE|
|4 迁移|Gate 3 PASS 域内已确认 episode|\(\Delta StartRate_5,\Delta CDS_5,\Delta PriceShare_5\)|任一≤0|0.10/0.05/0.05；fail均0|题材簇×月|固定顺序第4；m=3 IUT|仅Gate1–3 PASS后；HOLDOUT一次|三者均PASS才PASS；任一FAIL才FAIL；其余INCONCLUSIVE|
|5 时钟|Gate 4 PASS 域内容量候选|标准化后交互项 \(\beta_3\)|≤0|0.05 / 0|题材簇×月|固定顺序第5，α=.05|仅Gate1–4 PASS后；HOLDOUT一次|§18.0；不足→INCONCLUSIVE|
|6 选择|同题材同日可交易容量股|EntryScore前2−匹配随机 NetAlpha20|≤0|1% / 0|题材簇×月|固定顺序第6，α=.05|仅Gate1–5 PASS后；HOLDOUT一次|§18.0；不足→INCONCLUSIVE|
|7 交互|2×2共同支持样本|\(\Delta_{INT}\)|≤0|1% / 0|题材簇×月|固定顺序第7，α=.05|仅Gate1–6 PASS后；HOLDOUT一次|§18.0；不足→INCONCLUSIVE|
|8 执行|有效期内实际成交信号|NetAlpha20 与 \(-\theta_{ADV}\)|任一不达标|0.01/−0.01；fail 0/−0.01|题材簇×月|固定顺序第8；m=2 IUT|仅Gate1–7 PASS后；HOLDOUT一次|两者均PASS才PASS；逆向选择按 §18 Gate 8；其余INCONCLUSIVE|
|9 退出|同一入场成交序列|动态−固定 Sharpe差、净收益差|任一≤0|0.10/0；fail均0|题材簇×月|固定顺序第9；m=2 IUT|仅Gate1–8 PASS后；HOLDOUT一次|两者均PASS才PASS；任一FAIL才FAIL；其余INCONCLUSIVE|
|S 叠加|同一入口/退出序列|叠加−无叠加的Sharpe差|≤0|0.10 / 0|月块|独立 Gate S 家族 Holm FWER 5%|HOLDOUT一次；FORWARD仅Kill|按 §18.0；FAIL/INCONCLUSIVE 均不进入生产|

**MES 说明：** 以上为预注册工程/经济最小效应，不是市场自然常数。修改即新版本。复合 Gate 使用 §18.0 IUT，核心9 Gate 使用固定顺序 gatekeeping；Gate S 才使用 Holm。所有效果量输出双侧95% CI，并输出正式单侧下界与反向 Bonferroni 同时上界。

# 附录 C：Freeze 检查表

## 定义与实现
- [ ] A 类阻断 = 0；B 类阻断 = 0
- [ ] Pctl/EMA/状态/owner/账本算法唯一
- [ ] 不存在未解析参数
- [ ] `params.yaml` 数字 lint 通过

## 数据与 null
- [ ] DG01–DG15 全过
- [ ] NG01–NG07 全过
- [ ] P10 Phase-I 基线冻结
- [ ] 共同支持覆盖达标
- [ ] null 优化 DAG 通过 naive 等价与运行 SLO

## 统计
- [ ] 附录 B 十项完整
- [ ] 三态判词压力测试通过
- [ ] Monte Carlo 精度闸门通过
- [ ] HOLDOUT 未触碰校验通过

## 执行与治理
- [ ] 账本恒等式零失败
- [ ] Run-in 20日完成
- [ ] Kill Criteria 与漂移基线冻结
- [ ] 单实现独立重发审计全链一致
- [ ] 联合哈希包签署并创建 FREEZE TAG
