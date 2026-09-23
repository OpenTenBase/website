---
title: "DATA × Memory，驱动 Agent 持续进化——OpenTenBase 城市行成都站圆满举办"
date: 2026-09-21T18:05:00+08:00
author: OpenTenBase
description: "9 月 19 日下午，开放原子技术沙龙 · OpenTenBase 城市行走进成都。"
---

9 月 19 日下午，开放原子技术沙龙 · OpenTenBase 城市行走进成都。活动以「DATA × Memory：驱动 Agent 持续进化」为主题，由 OpenTenBase 社区、腾讯云联合主办，将社区嘉年华、开源技术分享、AI 智能体工作坊与开源黑客松汇聚在一个下午，吸引上百位开发者与高校技术爱好者来到现场。

<div class="text-center">
  <img src="../images/event-post-14-01.jpg" class="img-fluid" alt="OpenTenBase 城市行成都站活动现场" />
</div>

### 01 社区嘉年华：15 个展位连成一条开源长廊

活动伊始，WorkBuddy、腾讯云开发者社区、腾讯云架构师技术同盟、云贝教育、Bethune X、Datawhale、DBX、IFClub、川序、蜀鸿会、云技术、现代·开原社团、科成开源、腾讯开源、腾讯混元等 15 个社区及生态伙伴展位现场亮相，把会场连成一条热闹的开源嘉年华长廊：各展位社区伙伴面对面答疑交流，介绍社区工作，吸引新的朋友加入；参会者沿展位路线逛展打卡、集章兑换 OpenTenBase 与 WorkBuddy 限定周边，展位前始终人气不减。

<div class="text-center">
  <img src="../images/event-post-14-02.jpg" class="img-fluid" alt="社区嘉年华：15 个社区及生态伙伴展位现场交流" />
</div>

### 02 开场致辞：AI 正进入长程带状态时代

腾讯云数据库副总经理罗云在开场致辞中表示，AI 正从无状态的一次性服务演进为 7×24 小时运行的长程带状态服务，这一范式转移由「模型」与「上下文」共同驱动，而支撑 context 的是存量数据基础设施与 Agent 原生的非结构化记忆两套体系。他指出，OpenTenBase 脱胎于腾讯云 TDSQL 的企业级分布式数据库内核，由腾讯云捐赠至开放原子开源基金会孵化，希望与社区共建分布式企业级数据库。腾讯云数据库团队近期开源的 TencentDB Agent Memory 项目在七十余天内获得近三万个 star，成为腾讯增长最快的开源项目。他期待本次沙龙能促进业界在存量基础设施演进与增量记忆数据存储上的交流碰撞。

<div class="text-center">
  <img src="../images/event-post-14-03.jpg" class="img-fluid" alt="腾讯云数据库副总经理罗云开场致辞" />
</div>

### 03 技术分享：从内核原理到 Agent 记忆

随后三场分享依次展开，三位讲师围绕数据库带来了从底层原理到动手实践的完整路径。

**张坤《从 SQL 到执行计划：解析 OpenTenBase 分布式查询》**

腾讯 TDSQL-PG / OpenTenBase 内核研发工程师张坤从一条校园成绩查询出发，在本机真实运行的 2 CN + 2 DN 集群（OpenTenBase v5.0）上拆解计划的生成过程：读懂一份分布式计划，只需回答「怎么算、在哪算、数据怎么走」。他现场演示了同一条 SQL 在三种数据摆放方式下的差别——两表按相同分片键时数据就近相遇，换一种分片方式则引入节点间重分布，小表改为复制表后跨节点搬运随之消失，而三者查询结果完全一致。借助 EXPLAIN VERBOSE，他还说明了 Remote 节点即进程边界，并留下三个回去就能跑的实验：换分片键、改复制表、先问 AI 再验证。

<div class="text-center">
  <img src="../images/event-post-14-04.jpg" class="img-fluid" alt="张坤分享：从 SQL 到执行计划，解析 OpenTenBase 分布式查询" />
</div>

**冯光普《OpenTenBase 内核 TXSQL 性能优化实践》**

腾讯云数据库 MySQL 专家冯光普分享了 TXSQL 内核的性能优化实践。TXSQL 是腾讯捐赠到 OpenTenBase 社区的开源数据库内核，也是统一支撑 CDB、TDSQL-C、TDSQL 等产品的 MySQL 分支，已在腾讯云上承载 100PB+ 存储规模、超 40 万实例，并向上游社区回馈百余项 bugfix。围绕真实业务痛点，他重点讲解了两项优化：面向直播带货、春节红包等秒杀场景的热点更新技术，通过自动识别热点行并排队处理，把单账户并发交易能力从每秒 150 笔提升到千笔级；并行 DDL 采用三阶段全并行建索引，5 亿条数据的索引构建从 15 分钟缩短到 40 秒，加速比最高达官方 MySQL 的 5 倍。此外还有查询缓存、逻辑复制 Hash Scan 优化与物理复制并行回放等特性，勾勒出从真实问题反哺社区的内核优化路径。

<div class="text-center">
  <img src="../images/event-post-14-05.jpg" class="img-fluid" alt="冯光普分享：OpenTenBase 内核 TXSQL 性能优化实践" />
</div>

**谭琬潼《从个人记忆到团队资产：Agent 记忆的进化与开源之路》**

腾讯云数据库产品经理、TencentDB Agent Memory 产品负责人谭琬潼回答了一个核心问题：为什么 Agent 越来越能干，却总是像刚入职？她指出，能力被保留了，但状态被清空了——上一次任务形成的项目背景、判断与约束，没有成为下一次任务的输入。个人记忆解决「连续」，团队记忆解决「继承」：她将团队记忆拆解为 Chat Memory、LLM-Wiki、Code Graph、Skill 四类核心资产，分别回答「以前发生过什么、项目现在知道什么、代码改动影响哪里、团队通常怎样做」，并提出资产化、原子化、相关性优先、治理内生、持续闭环五大设计原则。架构上由 Gateway、Context Proxy 与 Memory Core 分层协作，把权限隔离、质量控制、生命周期、可信度与可追溯审计做进资产全流程，让团队经验做到「干完有沉淀、换人不重来、开局就读档」。目前个人记忆服务已在 GitHub 开源（TencentDB-Agent-Memory），欢迎社区围绕记忆检索算法、资产治理策略与真实场景反馈共建。

<div class="text-center">
  <img src="../images/event-post-14-06.jpg" class="img-fluid" alt="谭琬潼分享：从个人记忆到团队资产，Agent 记忆的进化与开源之路" />
</div>

### 04 实操工作坊：从第一次 PR 到开源数据库助手

下午的工作坊转向动手实践。

**周正中（德哥）《利用 WorkBuddy 完成你的第一次 PR》**

OpenTenBase 社区 ACE、PG 中文社区创始人之一周正中（德哥）把开源贡献流程拆解为准备、实操、交付三阶段共 14 个步骤：从注册 GitHub、签署 CLA、在 Issue 下留言认领，到 Fork 仓库、创建特性分支、编写测试并完成编译回归，再到规范提交 Commit、根据 Maintainer 意见迭代修正直至合入主干。其中编码与自纠环节，他现场打开 WorkBuddy 实操演示：新建任务、选择代码开发、安装 codegraph 扩展，让 AI 在仓库上下文里完成编码与自查——这也是建议全场参会者下午在黑客松里上手的工具。他还梳理了社区 Issue 的六大门类——文档与体验优化、构建与环境兼容、运维工具与可观测、测试用例与质量保障、周边生态与协议兼容、内核健壮性与 Bug 修复，并给出一份按耗时与新手友好度评级的核心 Issue 清单，让不同基础的参与者都能找到适合自己的第一刀。

<div class="text-center">
  <img src="../images/event-post-14-07.jpg" class="img-fluid" alt="周正中（德哥）分享：利用 WorkBuddy 完成你的第一次 PR" />
</div>

**张瑞《基于 WorkBuddy 构建开源数据库助手》**

腾讯云开发者社区技术产品运营、WorkBuddy 产品布道师、OpenTenBase 核心贡献者张瑞则演示了如何用 WorkBuddy 把「读网」变成「资产」。他沿收集、整理、筛选入库、复用成资产四个环节展开：在 WorkBuddy 中通过资讯专家、资讯类 Skill 与自定义采集 Skill 多路收集，统一汇入收件箱；针对视频、论文、开源项目等不同内容类型提取不同要点，同时保留原文与出处；再以质量评分与人工判断决定入库，沉淀为可随时回查的资料库。他现场展示了一次采集 Skill 的迭代过程：把关注方向写进 WorkBuddy 记忆后，单次采集相关度从 3.6/8 提升到 7.1/8，一手来源从 0 提升到 8/8；同时提醒「规则越多，采集未必越准」，每一版规则变更都要重新验证。AI 技术动态跟踪、Agent 框架选型、OpenTenBase 专家包三个案例，串起从信息消费到知识资产、再到开源贡献的完整路径。

<div class="text-center">
  <img src="../images/event-post-14-08.jpg" class="img-fluid" alt="张瑞分享：基于 WorkBuddy 构建开源数据库助手" />
</div>

### 05 开源黑客松：90 分钟从想法到可验证成果

随后的 90 分钟开源黑客松把现场气氛推向顶点。哨声一响，参会者立刻带着电脑组队上阵：现场敲定选题、认领 Issue、配置环境，把当天学到的 WorkBuddy 用法与 OpenTenBase 贡献流程直接投入实战，在倒计时中完成从想法到可验证成果的冲刺。

不少团队是第一次接触开源贡献，从环境配置、Commit 规范到 PR 提交，现场多位专家导师全程巡回指导，俯身到各个小组中排查问题、打磨方案，讨论声与键盘声此起彼伏。

<div class="text-center">
  <img src="../images/event-post-14-09.jpg" class="img-fluid" alt="开源黑客松现场" />
</div>

90 分钟倒计时结束，多个团队交出了自己的成果：有的完成了第一次代码或文档贡献，有的借助 WorkBuddy 围绕 OpenTenBase、TencentDB Agent Memory 开发了自己的应用。

<div class="text-center">
  <img src="../images/event-post-14-10.jpg" class="img-fluid" alt="黑客松团队展示成果" />
</div>

### 06 颁奖时刻：八支团队脱颖而出

经过初审、成果展示与评委打分，最终八支团队脱颖而出获得表彰，其中三支团队获开源优秀奖、五支团队获开源先锋奖。颁奖环节把现场气氛再度点燃：获奖团队依次登台，接过奖金与证书，台下掌声与欢呼声此起彼伏，为这个下午画上了最热烈的句号。

<div class="text-center">
  <img src="../images/event-post-14-11.jpg" class="img-fluid" alt="黑客松获奖团队合影" />
</div>

从一条 SQL 的执行计划，到数据库内核的极致优化，再到让 Agent 记住团队经验、带着更多人完成第一次 PR——成都站用一个下午验证了「从理解技术到动手共创」的完整路径。

10 月 16 日，OpenTenBase 城市行将来到上海，欢迎提前扫码报名锁定席位，下次见！

<div class="text-center">
  <img src="../images/event-post-14-12.jpg" class="img-fluid" alt="OpenTenBase 城市行上海站活动海报" />
</div>

**OpenAtom OpenTenBase 社区**

开源分布式数据库 · 欢迎 Star 与共建
