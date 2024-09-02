---
title: "贡献案例 | 跟随社区委员会成员实现OpenTenBase高效部署"
date: 2024-08-27T14:48:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-9.png
author: OpenTenBase
description: ""
---

**安装初体验**
========

本文根据官网安装教程（https://docs.opentenbase.org/guide/01-quickstart/#\_3） 进行OpenTenBase的快速入门安装。

首先，本次安装规划使用到两台服务器，搭建1GTM主，1GTM备，2CN主，2DN主，2DN备的集群，该集群为具备容灾能力的最小配置。

机器1（CentOS 7）：10.60.143.28  
机器2（CentOS 7）：10.60.143.92

**依赖安装**
========

根据官方要求安装依赖：

`yum -y install gcc make readline-devel zlib-devel openssl-devel uuid-devel bison flex git`  

这一步和安装Oracle以及PostgreSQL的依赖非常相似。

**创建OpenTenBase所需目录和用户**
========================

`mkdir /data`  
`useradd -d data/opentenbase -s bin/bash -m opentenbase`  
`passwd xxx # set password`  

  

下载源码

https://github.com/OpenTenBase/OpenTenBase/releases/tag/v2.5.0

并传到服务器  

**编译源码**
========

我以前学习PG的时候，有一次看到我安装的和老师安装的最后功能不一样。老师说，最后源码编译。导致我现在安装PG也习惯了。

`cd /data/opentenbase/OpenTenBase`  
`rm -rf data/opentenbase/install/opentenbase_bin_v2.0`  
`chmod +x configure*`  
`./configure --prefix=/data/opentenbase/install/opentenbase_bin_v2.0 --enable-user-switch --with-openssl --with-ossp-uuid CFLAGS=-g`  
`make clean`  
`make -sj`  
`make install`  
`chmod +x contrib/pgxc_ctl/make_signature`  
`cd contrib`  
`make -sj`  
`make install`  

编译成功后可以去 data/opentenbase/install/opentenbase\_bin\_v2.0 看到编译的结果：

<img src=../images/news-post-9-01.png class="img-fluid" /><br/>

然后我们配置下用到的机器之间的互信，基本的步骤：  

`su - opentenbase`  
`ssh-keygen -t rsa`  
`ssh-copy-id -i ~/.ssh/id_rsa.pub destination-user@destination-server`  

  

**免密配置**
========

环境变量配置（所有机器都要配置）：

`[opentenbase@oracle19ocp ~]$ vim ~/.bashrc`  
`export OPENTENBASE_HOME=/data/opentenbase/install/opentenbase_bin_v2.0`  
`export PATH=OPENTENBASEHOME/bin:OPENTENBASEHOME/bin:PATH`  
`export LD_LIBRARY_PATH=OPENTENBASEHOME/lib:OPENTENBASEHOME/lib:{LD_LIBRARY_PATH}`  
`export LC_ALL=C`  

  

准备搭建集群的配置文件：

`[opentenbase@oracle19ocp ~]$ mkdir /data/opentenbase/pgxc_ctl`  
`[opentenbase@oracle19ocp ~]$ cd /data/opentenbase/pgxc_ctl`  
`[opentenbase@oracle19ocp pgxc_ctl]$ vim pgxc_ctl.conf`  

  

配置文件基本都是官方给出的模板信息，我们这里只是把IP改为自己的实验机器。

`#!/bin/bash`  
`Double Node Config`  
`IP_1=10.60.143.28`  
`IP_2=10.60.143.92`  
`pgxcInstallDir=/data/opentenbase/install/opentenbase_bin_v2.0`  
`pgxcOwner=opentenbase`  
`defaultDatabase=postgres`  
`pgxcUser=pgxcOwnertmpDir=/tmplocalTmpDir=pgxcOwnertmpDir=/tmplocalTmpDir=tmpDir`  
`configBackup=n`  
`configBackupHost=pgxc-linker`  
`configBackupDir=$HOME/pgxc`  
`configBackupFile=pgxc_ctl.bak`  

  

上文中，我们在10.60.143.28编译好一份二级制包，现在要把它同步到集群所有服务器，这个可以使用pgxc\_ctl工具，执行deploy all命令来完成。

<img src=../images/news-post-9-02.png class="img-fluid" /><br/>

执行init all命令，完成集群初始化命令：

<img src=../images/news-post-9-03.png class="img-fluid" /><br/>

执行成功最终会输出：

<img src=../images/news-post-9-04.png class="img-fluid" /><br/>

**查看集群状态**
==========

<img src=../images/news-post-9-05.png class="img-fluid" /><br/>

访问OpenTenBase和访问单机PostgreSQL基本无差别，我们可以通过任意一个CN访问数据库集群：

<img src=../images/news-post-9-06.png class="img-fluid" /><br/>

OpenTenBase使用datanode group来增加节点的管理灵活度，要求有一个default group才能使用，因此需要预先创建；一般情况下，会将节点的所有datanode节点加入到default group里 另外一方面，OpenTenBase的数据分布为了增加灵活度，加了中间逻辑层来维护数据记录到物理节点的映射，我们叫sharding，所以需要预先创建sharding，命令如下：

<img src=../images/news-post-9-07.png class="img-fluid" /><br/>

**访问**
======

至此，就可以跟使用单机数据库一样来访问数据库集群了，创建数据库，用户，创建表，增删查改等操作：

<img src=../images/news-post-9-08.png class="img-fluid" /><br/>

启停
==

通过pgxc\_ctl工具的 stop all 命令来停止集群，stop all 后面可以加上参数 -m fast 或者是 -m immediate 来决定如何停止各个节点。

<img src=../images/news-post-9-09.png class="img-fluid" /><br/>

通过pgxc\_ctl工具的start all命令来启动集群  
start all

<img src=../images/news-post-9-10.png class="img-fluid" /><br/>

<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

*   AtomGit
    
    https://atomgit.com/opentenbase/OpenTenBase
    
*   GitHub
    
    https://github.com/OpenTenBase/OpenTenBase