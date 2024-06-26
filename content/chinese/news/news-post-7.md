---
title: "30万奖金花落谁家？OpenTenBase开源核心贡献挑战赛精彩收官"
date: 2024-04-23T15:19:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-7.png
author: OpenTenBase
description: ""
---

4月17日，开放原子开源大赛南京站“OpenAtom OpenTenBase开源核心贡献挑战赛”线下决赛路演和评奖顺利完成。

<img src=../images/news-post-7-01.png class="img-fluid" /><br/>

“OpenTenBase开源核心贡献挑战赛”是开放原子开源大赛基础软件赛道备受瞩目的一环。

OpenTenBase开源核心贡献挑战赛分为开源贡献排名赛和核心贡献挑战赛两个部分，吸引了来自各大高校、科研机构和企业的多达544支队伍的积极参与，充分展现了开源数据库OpenTenBase在社区中广泛的号召力。

核心贡献挑战赛提供了分布式数据库执行效率优化、生态组件分布式改造、商业数据库语法兼容、商业数据库兼容视图以及数据库SQL信息动态展示等高质量赛题，这些题目既紧贴现实需求，又有一定的创新性和挑战性。参赛者中既有才华横溢的在校学生，也有经验丰富的一线技术专家，他们各展所长，充分展现了卓越的专业素养，并诞生了大量令人瞩目的优秀作品。经过数月的激烈较量，最终有11支作品脱颖而出，成功晋级至激动人心的决赛路演环节。

为确保评审的公正与专业，大赛特邀了西北工业大学国家示范性软件学院教授、博士生导师武君胜，腾讯数据库研发负责人伍鑫，腾讯云数据库产品总监王云龙，腾讯云数据库高级开发工程师陈再妮，以及腾讯开源联盟主席单致豪等5位专家担任决赛评委，这些专家在数据库开发和开源等领域均有丰富的经验和深厚的技术积累，他们认真负责的态度和专业的点评也为大赛增添了光彩。

在决赛的舞台上，各参赛团队信心满满，从容展示了其作品的创新亮点和技术优势。五位评委分别从赛题难度、代码完成度、易用性、安全性、价值和应用等维度对作品进行了全方位的打分和评点，并针对每个作品提出了宝贵的意见和建议。经过评委们的认真评审，最终决出了以下奖项：

<img src=../images/news-post-7-02.png class="img-fluid" /><br/>

在此也向获奖团队表示祝贺，他们将分享由大赛提供的30万奖金。

我们再来回顾一下本次挑战赛中很有代表性的优秀作品。

来自天翼云的国云数智团队带来了对OpenTenBase的深度优化，其作品包含RDA (Remote Data Access)和DDS(Distributed Dependency Spread)两个部分。RDA为OpenTenBase提供了高效的节点间异步实时数据通信框架，可以解决OpenTenBase基于PG原生通信架构存在的一系列问题。DDS是⼀种分布式死锁检测算法，它基于RDA实现，适⽤于多节点的分布式数据库系统，能够在分布式死锁出现时由内核⾃动检测并解除死锁。该作品也因其深度和前瞻性荣获本次挑战赛的一等奖。

<img src=../images/news-post-7-03.png class="img-fluid" /><br/>

中软国际磐石数据库团队在OpenTenBase中实现了ALL_OBJECTS/ALL_TABLES/ALL_PROCEDURES/ALL_TAB_COLUMNS 4张视图。在分析了Oracle 19C中相应的视图后，团队结合对OpenTenBase中相同对象及属性的兼容性分析，编写了对应的视图创建SQL，并将其追加至内核代码中，最终实现了在OpenTenBase pg_catalog schema下创建公共兼容视图，从而显著提升了OpenTenBase与Oracle的兼容性，同时能极大地方便开发人员、DBA管理与使用数据库对象，降低数据库切换后的学习成本。该作品在本次挑战赛中荣获二等奖。

“友谊第一”团队在OpenTenBase中实现了pgxc_ora_sysview和pgxc_dbms_metadata两个插件，分别实现了Oracle 19C的4张系统视图兼容和DBMS_METADATA兼容属性，并提供了丰富的测试用例和详细的说明手册。该作品也在本次挑战赛中荣获了二等奖。

<img src=../images/news-post-7-04.png class="img-fluid" /><br/>

来自杭州云猿生、网易云音乐、用友、西安电子科技大学、晋中学院、中国科学技术大学、华北理工大学的8个团队荣获了本次挑战赛的三等奖。

<img src=../images/news-post-7-05.png class="img-fluid" /><br/>

除此之外，还有来自长虹佳华、北京商越网络、长沙飞思科技、拱北海关、上海依图科技、西安百变网络、安徽标信查、北京趣拿科技、杭州数溪科技、合肥大学、阜阳师范大学、北京航空航天大学、苏州大学、晋中学院的多个团队或个人获得了开源贡献奖。

参与评审的几位专家老师对参赛作品的水准给予了高度评价。本次挑战赛的题目与数据库迁移和落地过程中的现实需求紧密契合，具有一定的门槛和难度。许多团队对数据库技术有着深入的了解，并进行了充分的调研，能在较短的时间内完成作品的开发和测试工作，实属不易。

通过OpenTenBase开源核心贡献挑战赛这个平台，参赛团队不仅提升了自身的技术实力，也结识了志同道合的伙伴。相信随着越来越多的开源贡献者和用户的加入，OpenTenBase数据库的生态会越来越繁荣！

<img src=../images/news-post-7-06.png class="img-fluid" /><br/>

期待更多的朋友加入我们，共同打造一个更加强大和活跃的OpenTenBase开发者社区！


OpenTenBase开源地址：
https://github.com/OpenTenBase

欢迎添加蓓蓓（微信号：OpenTenBase）加入OpenTenBase交流群，获取最新资讯及公告！

<img src=../images/news-post-7-07.png class="img-fluid" /><br/>