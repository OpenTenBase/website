---
title: "从智能优化器到数据库 Agent：OpenTenBase 城市行杭州站圆满举办"
date: 2026-08-31T10:00:00+08:00
author: OpenTenBase
description: "8 月 29 日下午，开放原子技术沙龙·OpenTenBase 城市行杭州站顺利举办，7 场技术分享与 1 场 Workshop，从智能优化器到数据库 Agent 生产落地，吸引上百位数据库开发者、DBA、架构师与 AI 应用开发者参与。"
---

8 月 29 日下午，开放原子技术沙龙·OpenTenBase 城市行杭州站顺利举办。本次活动由 OpenTenBase 社区与腾讯云联合主办，以“从智能优化器到数据库 Agent 生产落地”为主题，共安排 7 场技术分享与 1 场 Workshop，集中探讨 AI 查询优化、OpenTenBase 分布式 SQL 调优、Agent 记忆、AI 辅助数据库内核开发及数据库 Agent 生产治理等议题，吸引上百位数据库开发者、DBA、架构师和 AI 应用开发者参与。

<div class="text-center">
  <img src="../images/event-post-11-01.jpg" class="img-fluid" alt="OpenTenBase 城市行杭州站活动现场" />
</div>

### 一、社区演进：智能体时代的开源路线图

腾讯开源 TOC 主席、OpenTenBase 社区秘书长单致豪以《AI 与智能体时代 OpenTenBase 开源社区演进》开场。他从社区战略视角出发，分享了 OpenTenBase 在智能体时代的技术布局、生态建设与开源治理思路：在 AI 与智能体浪潮下，开源社区如何持续演进、连接开发者与产业实践，并为开发者和企业参与者勾勒出清晰的社区路线图。

<div class="text-center">
  <img src="../images/event-post-11-02.jpg" class="img-fluid" alt="单致豪分享：AI 与智能体时代 OpenTenBase 开源社区演进" />
</div>

### 二、TXSQL 自学习优化器：把专家调优经验沉淀进 AI

腾讯数据库研发工程师李天峰带来《TXSQL 自学习优化器——数据库智能查询优化落地实践》。他指出，数据库性能调优长期依赖资深专家经验，一条慢查询往往耗费大量人力反复排查：同一条 SQL 在不同数据分布下最优执行计划可能完全相反，而传统优化器存在基数估计不准确、代价模型不准确、搜索空间不完备、错误计划不感知等根本性缺陷，多表连接的计划空间更是呈阶乘级增长。

面对这些难题，TXSQL 给出的答案是“基于 AI 的自学习优化器”：通过完整的计划干预与计划重现能力，覆盖 39 种执行算子、十余种改写规则；通过智能计划管理（SPM）彻底解决大模型幻觉影响，以计划基线加持续演进提供兜底机制，实现性能“只增不减”；并以“真实负载数据、MCTS 扩展、SFT 对齐、RL 强化”的数据飞轮持续驱动腾讯混元优化器大模型迭代。

落地成效方面：AI 优化器已上线数千个实例，线上 SQL 总耗时降低 90% 以上；在 SaaS、金融等业务场景中，跑批任务从超过 6 小时缩短到 3.5 小时内，实现开箱即用的“千人千面”最优性能。

<div class="table-responsive">
<table class="table table-bordered">
<thead>
<tr><th>线上真实负载指标</th><th>SQL 总执行时间</th><th>CPU</th><th>慢日志</th></tr>
</thead>
<tbody>
<tr><td>优化效果</td><td>↓ 63.8%</td><td>↓ 90.8%</td><td>↓ 23.8%</td></tr>
</tbody>
</table>
</div>

<div class="text-center">
  <img src="../images/event-post-11-03.jpg" class="img-fluid" alt="TXSQL 自学习优化器分享现场" />
</div>

### 三、我的 AI 屠龙刀：放大你，而不是替代你

腾讯云架构师同盟成员、前阿里高级数据库专家德哥（周正中）带来极具个人风格的《我的 AI 屠龙刀》。他首先厘清了 AI 与人类的比较优势：AI 擅长海量记忆、24×7 不间断工作、并行处理、模式识别；人类则在真正创造、价值判断、审美品味、复杂沟通、情感共鸣与目标设定上不可替代。

核心洞察：“AI 屠龙刀不等于替代你，而是放大你”——把不擅长、效率低的部分交给 AI，让人专注于创造、判断与品味。

他提出好兵器的三大衡量标准：懂你、会进化、能交付，并拆解了屠龙刀的八大组成——智能体（Agents）、模型协议（MCP）、技能包（Skill）、工具集（Tools）、记忆模块（Memory）、运行环境（Runtime）、权限系统（Permission）与代码地图（Codegraph）。在实战层面，德哥展示了用自动化工作流“找 Bug、修 Bug、实现 Issue”（如 Claude 加 Codegraph 处理 PostgreSQL Issue）、论文·新闻·播客·公众号创作、股市分析·财报解读等场景，并提出“越用越好：沉淀垂直场景资产”的进化闭环，最终以“一个 Docker 镜像搞定一切”收尾。

<div class="text-center">
  <img src="../images/event-post-11-04.jpg" class="img-fluid" alt="德哥（周正中）分享：我的 AI 屠龙刀" />
</div>

### 四、内生智能数据库：直面访存延迟这个核心瓶颈

易景科技首席研究员、北京大学企业导师吕海波带来前瞻研究《内生智能数据库前瞻技术研究》。他从 hk2（为 coding 而生的智能体）切入，随后沿“纸面算力下的未来之路”与“内存的破局之道”两条主线展开：从 CPU 到 GPU 再到 LPU 等新型处理器，“推理计算”的底层范式正在深刻变化；而无论 DDR5、GDDR 还是 HBM，内存核心频率始终在 500～1000MHz 之间，访存延迟才是 AI 时代算力的最核心瓶颈，破局之道在于冗余计算单元与隐藏访存延迟，最终回归数据库在 AI 时代的危机与机遇。

<div class="text-center">
  <img src="../images/event-post-11-05.jpg" class="img-fluid" alt="吕海波分享：内生智能数据库前瞻技术研究" />
</div>

### 五、分布式 SQL 调优：先看数据在哪，再看 SQL 怎么跑

杭州云贝数据库技术有限公司总经理、腾讯云 TVP、PostgreSQL ACE 总监郭一军带来《打破分布式 SQL 性能瓶颈——OpenTenBase 优化器与 SQL 调优实战》。这位 25 年数据库老兵开宗明义：分布式时代，SQL 性能瓶颈已从单机 CPU/IO 转向网络通信、数据重分布与分布式计划生成。

“分布式 SQL 调优的第一性问题，是数据在哪、要搬几次，其次才是索引和参数。”

他系统拆解了 OpenTenBase 的 CN/DN/GTM 架构与查询执行链路、RBO 与 CBO 的融合机制、代价因子的计算方式（从全表扫描、索引扫描 0.29 的启动成本到 LIMIT 的代价截断）、五种扫描方式与三种 Join 算法，以及 SQL Shipping 与 PLAN Shipping 的取舍。随后以六大真实生产案例给出可复用打法：

**六大真实生产案例**

<div class="table-responsive">
<table class="table table-bordered">
<thead>
<tr><th>案例</th><th>说明</th></tr>
</thead>
<tbody>
<tr><td>案例一</td><td>复合索引：查询从 1624ms 降到 1.04ms，提升约 1566 倍</td></tr>
<tr><td>案例二</td><td>带上分片键：避免不带分片键查询被广播到全部 DN</td></tr>
<tr><td>案例三</td><td>分片键与 Join 列对齐：消除数据重分布，651 秒到 20 毫秒，约 3.2 万倍</td></tr>
<tr><td>案例四</td><td>小表改复制表：Join 本地化，8.1 秒到 86 毫秒，约 94 倍</td></tr>
<tr><td>案例五</td><td>DISTINCT 改写为 GROUP BY：支持局部聚合下推，27 秒到 11 秒</td></tr>
<tr><td>案例六</td><td>会话级调大 work_mem：消除磁盘排序，Sort Method 从 Disk 到 Memory</td></tr>
</tbody>
</table>
</div>

最后他总结了“先看数据在哪，再看 SQL 怎么跑”的五步法与六条军规，为分布式场景提供一套经生产验证的方法论。

<div class="text-center">
  <img src="../images/event-post-11-06.jpg" class="img-fluid" alt="郭一军分享：打破分布式 SQL 性能瓶颈" />
</div>

### 六、Agent 记忆开源：从个人记忆到团队资产

腾讯云数据库产品经理谭琬潼分享《从个人记忆到团队资产：Agent 记忆的进化与开源之路》。她指出“Agent 总像刚入职”的痛点：传统方案或暴力截断上下文，或将历史无脑堆入向量库，代价是高昂的推理成本与失控的幻觉率。其团队的设计思路是“个人记忆解决连续，团队记忆解决知识、技能与决策的继承”，将团队沉淀为四类核心资产——Chat Memory、LLM-Wiki、Code Graph 与 Skill，并以统一资产语义、五个设计原则贯穿整体架构：Proxy 编排上下文、Core 管理资产事实、任务闭环与治理三部分协同工作。

谭琬潼介绍，TencentDB Agent Memory 以 MIT 协议完全开源，零外部 API 依赖、本地即可部署、坚持框架中立：符号化短期记忆最高降低 61% 的 Token 消耗、任务通过率相对提升 51%；L0-L3 分层长期记忆把碎片对话层层提炼为用户画像，PersonaMem 准确率从 48% 提升到 76%。演讲中还分享了该项目的团队记忆能力，并公开开源 Roadmap 与社区共建路径（github.com/TencentCloud/TencentDB-Agent-Memory），欢迎更多开发者参与记忆抽取与检索算法、Code Graph 与 Skill 连接器、评测集与文档等方向的共建。

<div class="text-center">
  <img src="../images/event-post-11-07.jpg" class="img-fluid" alt="谭琬潼分享：Agent 记忆的进化与开源之路" />
</div>

### 七、WorkBuddy 实战：pg_hint_plan 缓存重构提升 38% 性能

腾讯云 DBA、PostgreSQL ACE 杨向博分享了《WorkBuddy 助力：pg_hint_plan 性能优化实战——hint_table 缓存重构性能提升 38%》。他剖析了原生 hint_table 的三个弊端：每条 SQL 都要通过 SPI 发起一次查询、内部查询污染 pg_stat_statements 的 QPS 口径、缓存无法跨 backend 复用，高并发下成为明显瓶颈；而 PostgreSQL 的 SysCache 只服务系统表、relcache 失效也不生效，现成缓存机制用不了。

为此他提出两种方案：方案 A（v1.9.1）在每个 backend 内构建私有 HTAB 缓存，以 DSM 原子版本号做跨 backend 失效判断；方案 B（v1.9.2）进一步升级为 DSM 共享 dshash，配合 DSA 存储变长 hint、CAS single-flight 门闩解决冷启动并发。压测显示：在 64 并发下两种方案 TPS 分别提升 36.1% 与 38.2%，延迟降低 28.4%，单后端私有内存削减约 80%。他还总结了压测方法论的四个反直觉发现——VmRSS 测不了共享内存、小样本会骗人、空闲机器测不出差距、冷启动才暴露 Bug——强调“数据可信度比数字好看更重要”。

“AI 是杠杆，会用 AI 的人替代不会用的人”——这场分享同时是 AI 编程工具赋能数据库内核优化的真实落地案例：杨向博把需求拆成 8 条严格约束交给 WorkBuddy，完成“需求实现、自动测试验证迭代、高质量交付”的完整链路。用好 AI 的前提，是行业透彻认知与出色抽象能力。

<div class="text-center">
  <img src="../images/event-post-11-08.jpg" class="img-fluid" alt="杨向博分享：WorkBuddy 助力 pg_hint_plan 性能优化" />
</div>

### 八、数据库 Agent 五道关：从 Demo 走向生产

开源数据库工具 DBX 作者童天宇以《AI 会写 SQL 了，然后呢？——数据库 Agent 从 Demo 到生产的五道关》为当天的技术分享收官。他的核心判断是：大模型已经能快速生成 SQL，但“会写 SQL”并不等于“能够安全操作生产数据库”。当 Agent 开始选择连接、理解表结构、生成不同数据库方言并尝试执行时，真正的工程问题才刚刚开始。

结合开源数据库工具 DBX 的实践，数据库 Agent 落地生产必须通过五道关：上下文、兼容性、权限、风险、执行，并配以“上生产前的五个检查问题”，把五道关收束成一套可用于方案设计和上线评估的检查框架。

五道关逐一拆解：上下文（一个消失的 Schema 会怎样误导 Agent）、兼容性（方言和能力应该由系统约束）、权限（最终权限是四层上限的交集）、风险（有 WHERE 不代表有范围）、执行（MCP 提供接口，治理决定边界）。

<div class="text-center">
  <img src="../images/event-post-11-09.jpg" class="img-fluid" alt="童天宇分享：数据库 Agent 从 Demo 到生产" />
</div>

### 九、Workshop：与开源项目面对面

技术分享结束后，活动进入 Workshop 环节。李天峰、谭琬潼、童天宇三位专家分别结合项目 GitHub 仓库，向现场开发者介绍了 TXSQL、TencentDB Agent Memory 与 DBX 三个开源项目的最新情况与社区参与方式。三位专家还就 Issue 提报、PR 协作流程等社区参与细节进行了展示，鼓励大家以贡献者身份加入开源社区共建。

### OpenTenBase ACE 专家授牌仪式

活动最后举行了 OpenTenBase ACE 专家授牌仪式。OpenTenBase ACE 专家委员会主席薛晓刚为五位新任 ACE 专家授牌，他们是程奕豪、马顺华、张镇、洪杰、周正中（德哥）。五位专家均在数据库领域深耕多年、经验丰富，今后将在技术布道、案例实践、社区答疑与课程共建等方向持续为 OpenTenBase 社区贡献力量。至此，OpenTenBase ACE 专家队伍进一步壮大，社区的技术影响力与生态辐射范围也迈上了新台阶。

<div class="text-center">
  <img src="../images/event-post-11-10.jpg" class="img-fluid" alt="OpenTenBase ACE 专家授牌仪式" />
</div>

### 十一、结语

从查询优化器的智能化，到数据库 Agent 的生产治理；从分布式 SQL 调优，到 AI 辅助数据库内核开发，7 场分享共同呈现出一个清晰趋势：数据库与 AI 的融合，正在从能力验证走向生产工程。

OpenTenBase 城市行希望持续连接内核研发者、DBA、架构师与开源贡献者，让真实的生产经验转化为可复用的方法，并进一步沉淀为开放的社区能力。杭州站至此圆满落幕，期待下一站再见。

**OpenAtom OpenTenBase 社区**

开源分布式数据库 · 欢迎 Star 与共建

官网：https://www.opentenbase.org

AtomGit：https://opentenbase.atomgit.com

GitHub：https://github.com/OpenTenBase/OpenTenBase

TXSQL：https://github.com/OpenTenBase/TXSQL

觉得项目不错？给我们点个 Star ⭐ 吧

提交 Issue / PR，一起把国产分布式数据库做得更好

— 关注我们，获取更多技术干货 —
