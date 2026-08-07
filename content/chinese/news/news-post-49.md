---
title: "第八届中国PostgreSQL数据库生态大会圆满收官，OpenTenBase摘得开源影响力大奖"
date: 2025-11-30T20:16:00+08:00
#image_webp: images/news/news-post-49.webp
image: images/news/news-post-49.png
author: OpenTenBase
description: ""
---

2025年11月29日，第八届中国PostgreSQL数据库生态大会在杭州西溪灵隐智选假日酒店圆满落幕。本届大会由中国开源软件联盟PostgreSQL分会主办，以“开源无界，探索无限可能”为主题，汇聚了来自全国各地的行业专家、技术开发者、DBA、企业用户及开源爱好者，通过主旨演讲、专题研讨等形式，打造了一场兼具技术深度与社区温度的年度盛会。

会上，腾讯云数据库研发总监、OpenAtom OpenTenBase社区技术委员会副主席范孝剑发表主旨演讲，深入解析了PostgreSQL在开源生态与商业价值融合中的发展路径，并分享了腾讯云在PostgreSQL产品演进中的最佳实践。他指出，PostgreSQL始终在架构与生态两个维度持续拓展新的可能性：在架构层面，从集中式、云原生存算分离到分布式架构不断演进，以应对数据规模增长带来的挑战；在生态层面，则从原生PG生态逐步扩展至HTAP混合负载、AI融合等多元场景。面对AI时代带来的新机遇，PostgreSQL正凭借其灵活的扩展能力与开放生态，不断夯实其作为企业级数据库的核心竞争力。

<img src=../images/news-post-49-1.png class="img-fluid" /><br/>

面对企业去O迁移的核心痛点，凭借极致的Oracle语法兼容能力，为用户提供了一条高效、平滑的迁移路径。具体而言，TDSQL PG/OpenTenBase通过在语法层、元数据层和视图层对Oracle与PostgreSQL的差异进行隔离——包括SQL语法、PL/SQL能力、系统视图及高级包等——同时在底层复用统一的执行引擎框架（涵盖事务管理、行列混合存储、MPP分布式架构、计算引擎与查询优化器），实现了“一套引擎、双模兼容”的技术突破。用户可在同一集群中灵活创建PostgreSQL或Oracle模式的数据库，极大降低了迁移成本与运维复杂度。

<img src=../images/news-post-49-2.png class="img-fluid" /><br/>

在性能优化方面，TDSQL PG/OpenTenBase的查询优化器能力全面升级。通过pushpred谓词下推、OR转UNION拆分、代价模型评估等优化器增强技术，复杂查询性能提升达五倍以上。结合AI大模型进行索引推荐与执行计划优化，为去O迁移提供了强有力的技术支撑。

<img src=../images/news-post-49-3.png class="img-fluid" /><br/>

面向HTAP场景，TDSQL PG/OpenTenBase构建了以协调节点（CN）为核心的分布式执行框架，结合自研的DOP并行调度机制与Push-based分片交互协议，进一步优化执行效率。其基于时间戳的事务设计使高并发OLTP性能提升超50%。同时，通过软硬件协同优化，TDSQL PG成功登顶国际权威基准测试TPC-DS榜单，彰显其极致性能实力。

<img src=../images/news-post-49-4.png class="img-fluid" /><br/>

此外，为响应AI时代对多模态数据分析的需求，OpenTenBase打造了统一的多模态数据底座，支持结构化、半结构化与非结构化数据的融合存储与管理。用户可直接通过标准SQL调用内置AI函数，实现文本生成、图像分析、情感识别等场景的库内关联分析，真正打通数据与智能的“最后一公里”。

<img src=../images/news-post-49-5.png class="img-fluid" /><br/>

在大会颁奖环节，OpenTenBase凭借其在开源社区建设、技术创新与产业落地方面的卓越贡献，荣获 “2025年度开源影响力奖”。OpenTenBase社区秘书长单致豪代表社区上台领奖。

<img src=../images/news-post-49-6.png class="img-fluid" /><br/>

多位OpenTenBase ACE专家也在现场展开深入交流，探讨了OpenTenBase社区未来的发展方向。

<img src=../images/news-post-49-7.png class="img-fluid" /><br/>

“开源无界，探索无限可能” 不仅是本届大会的主题，更是OpenTenBase持续前行的方向。此次斩获 “2025年度开源影响力奖”，既是行业对其技术创新力与产业落地价值的认可，也是对开源生态共建的激励。未来，OpenTenBase将继续携手社区开发者、行业伙伴和广大用户，深耕PostgreSQL技术生态，强化在金融、政务、电信、AI等关键领域的应用支撑能力；同时积极参与全球开源协作，推动数据库与人工智能的深度融合，让开源技术真正成为产业数智化转型的 “加速器”。
