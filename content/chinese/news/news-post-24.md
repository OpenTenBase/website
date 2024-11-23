---
title: "社区贡献 | 使用OpenTenBase命令行工具pgxc_ctl添加DN节点"
date: 2024-11-18T16:31:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-24.png
author: OpenTenBase
description: ""
---
<img src=../images/news-post-24-1.png class="img-fluid" /><br/>

pgxc_ctl 是 OpenTenBase 提供的一个命令行工具，用于管理和操作集群。添加 DN（Data Node）节点是扩展集群存储和处理能力的常见操作。

以下是使用 pgxc_ctl 添加 DN 节点的详细步骤。

一、前提条件
------------

1. 已经安装并配置好 OpenTenBase 集群：确保你已经有一个运行中的 OpenTenBase 集群。
2. 已编译并安装好 OpenTenBase 二进制文件：确保 pgxc_ctl 工具可用。
3. 新的 DN 节点已经准备：确保新的 DN 节点已经安装了必要的依赖，并且可以访问现有的集群节点。

二、当前运行环境如下
--------------------

```
PGXC monitor all
Running: gtm master
Running: gtm slave
Running: coordinator master cn001
Running: coordinator master cn002
Running: datanode master dn001
Running: datanode slave dn001
Running: datanode master dn002
```

集群信息如下：


三、添加DN节点
--------------

目标：在新的节点192.168.2.138添加dn003节点

注：PGXC的操作均在管控节点192.168.2.136上操作

### 3.1. 准备新 DN 节点

* 在新的 DN 节点上，创建必要的目录结构并设置权限：

  # 创建数据目录mkdir -p /data/opentenbase/data/dn003  mkdir -p /data/opentenbase/installchown -R opentenbase:opentenbase /data/opentenbase  # 创建 opentenbase 用户useradd -d /data/opentenbase -s /bin/bash -m opentenbase   # 设置密码passwd opentenbase

### 3.2. 配置opentenbase环境变量

[opentenbase]$ vim ~/.bashrcexport OPENTENBASE_HOME=/data/opentenbase/install/opentenbase_bin_v2.6export PATH=$OPENTENBASE_HOME/bin:$PATHexport LD_LIBRARY_PATH=$OPENTENBASE_HOME/lib:${LD_LIBRARY_PATH}export LC_ALL=C
### 3.3. 分发二进制文件

将 OpenTenBase 的二进制文件分发到新的 DN 节点。

* 管控节点192.168.2.136分发软件包

  [opentenbase]pgxc_ctl -c /data/opentenbase/pgxc_ctl/pgxc_ctl.conf deploy 192.168.2.138

### 3.4. 配置 SSH 互信

确保现有的集群节点可以无密码访问新的 DN 节点：

# 生成 SSH 密钥对（如果还没有）ssh-keygen -t rsa   # 将公钥复制到新的 DN 节点ssh-copy-id -i ~/.ssh/id_rsa.pub opentenbase@192.168.2.138
### 3.5 添加dn003节点

pgxc_ctl -c /data/opentenbase/pgxc_ctl/pgxc_ctl.conf PGXCadd datanode master dn003 192.168.2.138 40004 20012 /data/opentenbase/data/dn003 none none none none
### 3.6 节点操作补充(不用执行)

1）初始化新的 DN 节点

[opentenbase]pgxc_ctl -c /data/opentenbase/pgxc_ctl/pgxc_ctl.conf PGXCinit datanode dn003
2）启动新的 DN 节点

使用 pgxc_ctl 工具启动新的 DN 节点：

[opentenbase]pgxc_ctl -c /data/opentenbase/pgxc_ctl/pgxc_ctl.conf PGXCstart datanode dn003
3）移除DN节点

使用 pgxc_ctl 工具移除新的 DN 节点：

[opentenbase]pgxc_ctl -c /data/opentenbase/pgxc_ctl/pgxc_ctl.conf PGXCremove datanode master dn003  clean
四. 验证
--------

新增的 DN 节点连接到任意一个协调节点（CN），验证新的 DN 节点是否已经成功加入集群：

[opentenbase@db1 ~]$ psql -h 192.168.2.136 -p 30004 -d postgres -U opentenbasepsql (PostgreSQL 10.0 OpenTenBase V2)Type "help" for help.postgres=# select * from pgxc_node; node_name | node_type | node_port |   node_host   | nodeis_primary | nodeis_preferred |   node_id   |  node_cluster_name  -----------+-----------+-----------+---------------+----------------+------------------+-------------+--------------------- gtm       | G         |     50001 | 192.168.2.136 | t              | f                |   428125959 | opentenbase_cluster cn001     | C         |     30004 | 192.168.2.136 | f              | f                |  -264077367 | opentenbase_cluster cn002     | C         |     30004 | 192.168.2.137 | f              | f                |  -674870440 | opentenbase_cluster dn001     | D         |     40004 | 192.168.2.136 | t              | t                |  2142761564 | opentenbase_cluster dn002     | D         |     40004 | 192.168.2.137 | f              | f                |   -17499968 | opentenbase_cluster dn003     | D         |     40004 | 192.168.2.138 | f              | f                | -1956435056 | opentenbase_cluster(6 rows)
五、总结
--------

通过以上步骤，你可以成功地使用 pgxc_ctl 工具向 OpenTenBase 集群中添加一个新的 DN 节点。主要步骤包括准备新节点、分发二进制文件、添加节点以及验证新增节点是否成功加入集群。

<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

* AtomGit

  https://atomgit.com/opentenbase/OpenTenBase
* GitHub

  https://github.com/OpenTenBase/OpenTenBase
