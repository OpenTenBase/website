---
title: "OpenTenBase社区贡献案例：如何通过修改代码增强现有SQL语法"
date: 2024-08-29T09:34:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-11.png
author: OpenTenBase
description: ""
---

**贡献者介绍**
---------

李人瑞，本科毕业于中国矿业大学数据科学与大数据技术专业，现就读于苏州大学，攻读大数据技术与工程专业的研究生学位。

在参与2023年12月于无锡举行的OpenAtom Techathon极限24小时编程马拉松活动中，李人瑞第一次接触了OpenTenBase，学习了相关的基础知识。后续，他作为小队队长参加了开放原子开源大赛OpenTenBase开源核心贡献挑战赛，对OpenTenBase进行Oracle语法适配，并获得了开源贡献奖。

**案例背景**
--------

OpenTenBase是腾讯云数据库团队在PostgreSQL基础上研发的企业级分布式HTAP开源数据库，具有高扩展性、商业数据库语法兼容、分布式HTAP引擎、多级容灾和多维度资源隔离等能力，成功应用在金融、政府、电信、医疗、航天等行业的核心业务系统。 

本案例主要关注Oracle语法中的select xxx from 主表 partition（子表）。通过对OpenTenBase中的语法表进行修改，实现了案例功能，并编写了相应的回归测试内容以验证修改的有效性。

**技术实现**
--------

**1\. 系统部署**
------------

在这个案例中，使用VMware Workstation创建了两台虚拟机，系统为CentOS 7（1511）。考虑到CentOS 7目前已停止维护，后续对此案例的实现可以使用OpenTenBase推荐的操作系统，如TencentOS 2, TencentOS 3, OpenCloudOS, Ubuntu。虚拟机的硬件设备设置如图一。

<img src=../images/news-post-11-1.png class="img-fluid" /><br/>

图 1 虚拟机的硬件设备设置

**2\.** **OpenTenBase部署** 
--------------------------

OpenTenBase的部署可以参考官方文档https://docs.opentenbase.org/guide/01-quickstart/。下面，通过源码编译的方式，在两台虚拟机上部署OpenTenBase。

首先，在虚拟机上安装相关依赖。

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_88025b6e-65aa-11ef-862e-fa163eb4f6be.png)

图2 相关依赖安装 

在OpenTenBase集群中的所有系统上创建opentenbase用户，之后的有关opentenbase的步骤都要切换到opentenbase用户下再进行操作。

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_88176de2-65aa-11ef-862e-fa163eb4f6be.png)

之后，下载源码并进行编译安装（这里使用GitHub，也可以使用AtomGit和Gitee网址）

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_882573a6-65aa-11ef-862e-fa163eb4f6be.png)

    1.cd ${SOURCECODE_PATH}  

上述代码中的两个参数如下所示

    ${SOURCECODE_PATH}=/data/opentenbase/OpenTenBase

下面以两台服务器上搭建1GTM主，1GTM备，2CN主（CN主之间对等，因此无需备CN），2DN主，2DN备的集群，该集群为具备容灾能力的最小配置。

OpenTenBase1

192.168.17.128

OpenTenBase2

192.168.17.129

集群规划如下：

节点

机器IP

目录

GTM master 

192.168.17.128

/data/opentenbase/data/gtm

GTM slave

192.168.17.129

/data/opentenbase/data/gtm

CN1

192.168.17.128

/data/opentenbase/data/coord

CN2

192.168.17.129

/data/opentenbase/data/coord

DN1 master

192.168.17.128

/data/opentenbase/data/dn001

DN1 slave

192.168.17.129

/data/opentenbase/data/dn001

DN2 master

192.168.17.129

/data/opentenbase/data/dn002

DN2 slave

192.168.17.128

/data/opentenbase/data/dn002

禁用SELinux和防火墙：

    1.vi etc/selinux/config   

执行vi /etc/selinux/config后，将文件中SELINUX=enforcing修改为SELINUX=disabled，如下图所示

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_882f5466-65aa-11ef-862e-fa163eb4f6be.png)

禁用防火墙流程如图所示。

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_884d5fd8-65aa-11ef-862e-fa163eb4f6be.png)

配置机器间的ssh互信：

    1.su opentenbase  

对于两台机器组成的集群，第三行代码（ssh-copy-id）在每台机器上要执行两次，即对本机和另一台机器进行ssh配置，详细内容见图4。

之后，在集群中的所有机器上进行配置环境变量：

    1.[opentenbase@localhost ~]$ vim ~/.bashrc  

 ![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_8861abe6-65aa-11ef-862e-fa163eb4f6be.png)  

图3 配置环境变量 

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_88774af0-65aa-11ef-862e-fa163eb4f6be.png)

图4 配置ssh互信

之后，初始化pgxc\_ctl.conf文件：

    1.[opentenbase@localhost ~]$ mkdir /data/opentenbase/pgxc_ctl  

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_888fac80-65aa-11ef-862e-fa163eb4f6be.png)

图5 初始化pgxc\_ctl.conf文件

pgxc\_ctl.conf文件可以复制官网快速入门文档中的内容，主要修改其中的IP地址，如图5红色部分所示。 

在一个节点配置好配置文件后，需要预先将二进制包部署到所有节点所在的机器上，这个可以使用pgxc\_ctl工具，执行deploy all命令来完成。

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_88a037bc-65aa-11ef-862e-fa163eb4f6be.png)

图6 使用pgxc\_ctl进行文件分发

不用退出pgxc\_ctl，输入并执行init all，进行集群的初始化。初始化之后，可以使用monitor all检查集群的运行情况。如图7所示，所有节点都正常运行。

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_88afc880-65aa-11ef-862e-fa163eb4f6be.png)

图7 集群运行情况

之后，可以用访问PostgreSQL的方式，对OpenTenBase集群进行访问。 

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_88c34cde-65aa-11ef-862e-fa163eb4f6be.png)

图8 访问OpenTenBase集群

**3\.** **Oracle语法适配** 
-----------------------

主要对语法表进行修改，以实现该案例中的语法适配。

其中，语法表gram.y位于OpenTenBase源文件的src/backend/目录下。

如图9所示，添加了红色框内的内容。通过对gram.y的内容进行分析，OpenTenBase之前已经有一部分关于partition的代码，所以直接添加了相关的语法，以达到目标。 

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_88d6ec26-65aa-11ef-862e-fa163eb4f6be.png)

图9 修改gram.y

为了验证修改的有效性，需要编写相应的回归测试内容。

其中，回归测试的相关内容位于OpenTenBase源文件的src/backend/目录下。

首先，创建用于回归测试的sql文件，文件路径为：src/test/regress/sql/select\_partition.sql，内容为

    1.--  

将上述代码与其理想输出写入文件中，这个文件的文件路径为：src/test/regress/expected/select\_partition.out。注意，sql文件与out文件的文件名是一样的。 

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_8904e090-65aa-11ef-862e-fa163eb4f6be.png)

图10 out文件的内容

之后，将该测试内容添加进测试计划中，测试计划的文件路径为：

src/test/regress/parallel\_schedule 

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_891e6678-65aa-11ef-862e-fa163eb4f6be.png)

图11 添加测试计划

修改之后，需要对OpenTenBase重新进行编译安装，并进行回归测试。

    1.cd ${SOURCECODE_PATH}  

修改后的回归测试输出如图12，可见添加的测试内容成功运行，且没有引入其他的错误。 

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240829_8937f2dc-65aa-11ef-862e-fa163eb4f6be.png)

图12 回归测试输出

修改之后，需要对编译后的OpenTenBase重新进行集群搭建，可以参考官方文档与2\. OpenTenBase部署部分。

完成以上部分，我们就实现了OpenTenBase对select xxx from 主表 partition（子表）这一语法的适配功能。

经验总结 
-----

本案例通过修改语法表的内容，实现了OpenTenBase对Oracle语法的适配。本次案例有以下几点经验可供借鉴：

1.可以查看语法表中已实现的内容，简单实现语法扩展。

2.修改内容后，要添加对应的回归测试内容，验证修改的有效性，且保证修改不引入更多的bug。

3.对于OpenTenBase源码修改任务，需要深入学习数据库知识，如编译器知识，语法知识等。

总的来说，本案例对OpenTenBase进行了简单的修改，如果需要适配更加复杂的内容，需要继续学习相应的数据库底层原理与知识。 

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

*   AtomGit
    
    https://atomgit.com/opentenbase/OpenTenBase
    
*   GitHub
    
    https://github.com/OpenTenBase/OpenTenBase