---
title: "社区贡献 | OpenTenBase架构介绍与部署步骤"
date: 2024-09-11T18:22:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-18.png
author: OpenTenBase
description: ""
---

本文分两部分，第一部分简单介绍OpenTenBase技术架构，第二部分则对OpenTenBase部署操作中所涉及的命令与步骤进行记录。
-------------------------------------------------------------------

### **什么是OpenTenBase**

OpenTenBase 是一个提供写可靠性，多主节点数据同步的关系数据库集群平台。你可以将 OpenTenBase 配置一台或者多台主机上， OpenTenBase 数据存储在多台物理主机上面。数据表的存储有两种方式， 分别是 distributed 或者 replicated ，当向OpenTenBase发送查询 SQL时， OpenTenBase 会自动向数据节点发出查询语句并获取最终结果。

OpenTenBase 采用分布式集群架构（如下图）， 该架构分布式为无共享(share nothing)模式，节点之间相应独立，各自处理自己的数据，处理后的结果可能向上层汇总或在节点间流转，各处理单元之间通过网络协议进行通信，并行处理和扩展能力更好，这也意味着只需要简单的x86服务器就可以部署 OpenTenBase 数据库集群。

<img src=../images/news-post-18-1.jpg class="img-fluid" /><br/>

### **OpenTenBase集群环境的搭建**

#### 两台云服务器10.99.1.1、10.99.1.2;GTM两台(主、备),CN节点两个,DN节点四个(两主、两备)

10.99.1.1:GTM主(端口50001)、CN1(端口30004)、DN1主(40004)、DN1备(端口50004)

10.99.1.2:GTM备(端口50001)、CN2(端口30004)、DN2主(40004)、DN2备(端口50004)

##### 执行命令10.99.1.1上

`安装依赖 yum -y install gcc make readline-devel zlib-devel openssl-devel uuid-devel bison flex git`  
`创建用户 mkdir data`  
`指定home目录 useradd -d /data/opentenbase -s /bin/bash -m opentenbase`  
`修改密码 passwd opentenbase`  
`获取源码 选取其中一台主机,在root用户下,切换用户 su - opentenbase`  
 `git clone https://github.com/OpenTenBase/OpenTenBase 只需要在次节点拉取即可,其他节点可以忽略此步骤`  
`设置环境变量 export SOURCECODE_PATH=/data/opentenbase/OpenTenBase`  
 `export INSTALL_PATH=/data/opentenbase/install`  
`源码编译 cd ${SOURCECODE_PATH}`  
 `rm -rf ${INSTALL_PATH}/opentenbase_bin_v2.0`  
 `chmod +x configure*`  
`开始    ./configure --prefix=${INSTALL_PATH}/opentenbase_bin_v2.0  --enable-user-switch --with-openssl  --with-ossp-uuid CFLAGS=-g`  
 `make clean`  
 `make -sj`  
 `make install`  
 `chmod +x contrib/pgxc_ctl/make_signature`  
 `cd contrib`  
 `make -sj`  
 `make install`  
`禁用 SELinux 和 防火墙 systemctl stop firewalld`  
`机器间的ssh互信配置 ssh-keygen`  
 `ssh-copy-id opentenbase@10.99.1.2`  
`环境变量配置 cd /data/opentenbase`  
 `vim .bashrc`  
 `粘贴    export OPENTENBASE_HOME=/data/opentenbase/install/opentenbase_bin_v2.0`  
 `export PATH=$OPENTENBASE_HOME/bin:$PATH`  
 `export LD_LIBRARY_PATH=$OPENTENBASE_HOME/lib:${LD_LIBRARY_PATH}`  
 `export LC_ALL=C`  
 `生效    source .bashrc`  
`初始化pgxc_ctl.conf文件 mkdir /data/opentenbase/pgxc_ctl`  
 `cd /data/opentenbase/pgxc_ctl`  
 `双节点配置文件:https://docs.opentenbase.org/guide/pgxc_ctl_double.conf`  
 `单节点配置文件:https://docs.opentenbase.org/guide/pgxc_ctl_single.conf`  
 `选择单节点`  
 `wget https://docs.opentenbase.org/guide/pgxc_ctl_double.conf`  
 `mv pgxc_ctl_double.conf pgxc_ctl.conf`  
 `vim pgxc_ctl.conf`  
 `把IP_1、IP_2换成10.99.1.1、10.99.1.2`  
`安装  pgxc_ctl`  
`PGXC deploy all`  
`PGXC init all`  
`查看节点运行情况`  
`PGXC monitor all`  
`登录数据库`  
`psql -h 10.99.1.1 -p 40004 -U opentenbase -d postgres`  
`DN节点加入到默认组中 create default node group default_group  with (dn001,dn002);`  
 `create sharding group to group default_group;`  

##### 执行命令10.99.1.2上

`安装依赖 yum -y install gcc make readline-devel zlib-devel openssl-devel uuid-devel bison flex git`  
`创建用户 mkdir /data`  
`指定home目录 useradd -d /data/opentenbase -s /bin/bash -m opentenbase`  
`修改密码 passwd opentenbase`  
`环境变量配置 su - opentenbase`  
 `cd /data/opentenbase`  
 `vim .bashrc`  
 `粘贴    export OPENTENBASE_HOME=/data/opentenbase/install/opentenbase_bin_v2.0`  
 `export PATH=$OPENTENBASE_HOME/bin:$PATH`  
 `export LD_LIBRARY_PATH=$OPENTENBASE_HOME/lib:${LD_LIBRARY_PATH}`  
 `export LC_ALL=C`  
 `生效    source .bashrc`  

<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

*   AtomGit
    
    https://atomgit.com/opentenbase/OpenTenBase
    
*   GitHub
    
    https://github.com/OpenTenBase/OpenTenBase