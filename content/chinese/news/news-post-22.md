---
title: "社区贡献 | OpenTenBase_V2.6基于麒麟源码编译安装"
date: 2024-10-29T16:31:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-22.png
author: OpenTenBase
description: ""
---
<img src=../images/news-post-22-1.png class="img-fluid" /><br/>

**前言：什么是OpenTenBase**

OpenTenBase 是一个提供写可靠性，多主节点数据同步的关系数据库集群平台。你可以将 OpenTenBase 配置一台或者多台主机上， OpenTenBase 数据存储在多台物理主机上面。数据表的存储有两种方式， 分别是 distributed 或者 replicated ，当向OpenTenBase发送查询 SQL时， OpenTenBase 会自动向数据节点发出查询语句并获取最终结果。

OpenTenBase 采用分布式集群架构（如下图）， 该架构分布式为无共享(share nothing)模式，节点之间相应独立，各自处理自己的数据，处理后的结果可能向上层汇总或在节点间流转，各处理单元之间通过网络协议进行通信，并行处理和扩展能力更好，这也意味着只需要简单的x86服务器就可以部署 OpenTenBase 数据库集群。

<img src=../images/news-post-21-2.jpg class="img-fluid" /><br/>

下面简单解读一下OpenTenBase的三大模块

• Coordinator：协调节点（简称CN）

业务访问入口，负责数据的分发和查询规划，多个节点位置对等，每个节点都提供相同的数据库视图；在功能上CN上只存储系统的全局元数据，并不存储实际的业务数据。

• Datanode：数据节点（简称DN）

每个节点还存储业务数据的分片在功能上，DN节点负责完成执行协调节点分发的执行请求。

• GTM:全局事务管理器(Global Transaction Manager)

负责管理集群事务信息，同时管理集群的全局对象，比如序列等。

本文将详细介绍如何从源码开始，完成 OpenTenBase V2.6 的编译和安装过程。

**一、安装准备及规划**

**1.1 环境要求**

在开始编译之前，请确保你的系统满足以下要求：

• 操作系统：TencentOS 2, TencentOS 3, OpenCloudOS, CentOS 7, CentOS 8, Ubuntu

• 内存：至少 4GB RAM

• 磁盘空间：至少10GB，足够的磁盘空间用于源码下载、编译和安装

**1.2 软件环境**


| 软件名称           | 软件版本   |
| ------------------ | ---------- |
| 麒麟服务端操作系统 | kylin\_v10 |
| OpenTenBase        | V2.6       |

**1.3 集群规划**

• 集群规划

下面以两台服务器上搭建1GTM主，1GTM备，2CN主（CN主之间对等，因此无需备CN），2DN主，2DN备的集群，该集群为具备容灾能力的最小配置

机器1：192.168.2.136
机器2：192.168.2.137

集群规划如下：


| 节点名称   | IP            | 数据目录                     |
| ---------- | ------------- | ---------------------------- |
| GTM master | 192.168.2.136 | /data/opentenbase/data/gtm   |
| GTM slave  | 192.168.2.137 | /data/opentenbase/data/gtm   |
| CN1        | 192.168.2.136 | /data/opentenbase/data/coord |
| CN2        | 192.168.2.137 | /data/opentenbase/data/coord |
| DN1 master | 192.168.2.136 | /data/opentenbase/data/dn001 |
| DN1 slave  | 192.168.2.137 | /data/opentenbase/data/dn001 |
| DN2 master | 192.168.2.137 | /data/opentenbase/data/dn002 |
| DN2 slave  | 192.168.2.136 | /data/opentenbase/data/dn002 |

**二、安装依赖**

根据你的操作系统，使用以下命令安装必要的依赖包：

对于基于 Red Hat 的系统（如 CentOS）：

```
dnf -y install gcc make readline-devel zlib-devel openssl-devel uuid-devel bison flex git
```

对于基于 Debian 的系统（如 Ubuntu）：

```
sudo apt-get update
sudo apt-get -y install gcc make libreadline-dev zlib1g-dev libssl-dev libossp-uuid-dev bison flex git
```

**三、创建 OpenTenBase 用户**

所有需要安装 OpenTenBase 集群的机器上都需要创建 opentenbase 用户，并设置相应的目录权限。

```
#创建 opentenbase 用户
useradd -d /data/opentenbase -s /bin/bash -m opentenbase#设置密码
passwd opentenbase
--mko0-pl,
```

**四、获取安装包**

**4.1 git获取源码**

• 创建软件目录

```
[root]
mkdir /dbsoft
```

• 使用 git 克隆 OpenTenBase 的源码仓库：

克隆源码(root用户)

```
git clone https://github.com/OpenTenBase/OpenTenBase
```

**4.2 下载源码包**

登录git，下载最新的v2.6.0版本

https://github.com/OpenTenBase/OpenTenBase/tags

**五、编译源码**

**5.1 配置安装环境变量**

所有节点都要操作

```
mkdir -p /data/opentenbase/{install,dbsoft}
chown -R opentenbase:opentenbase /data/opentenbase
```

**5.2 编译安装**

• 将源码包移动到源码目录

```
#复制安装包
cp /dbsoft/OpenTenBase-2.6.0.tar.gz /data/opentenbase/dbsoft

#解压
tar -zxvf OpenTenBase-2.6.0.tar.gz
```

• 进入源码目录并进行编译：

```
#进入源码目录
/data/opentenbase/dbsoft/OpenTenBase-2.6.0

#赋予配置脚本执行权限
chmod +x configure*

#配置编译选项
./configure --prefix=/data/opentenbase/install/opentenbase_bin_v2.6 --enable-user-switch --with-openssl --with-ossp-uuid CFLAGS=-g

#编译安装软件
make clean
make -sj  4
make install

#编译 contrib 目录下的工具
cd contrib
make -sj 4
make install
```

**六、集群初始化**

**6.1 禁用 SELinux 和 防火墙 (可选)**

```
--关闭selinux
vi /etc/selinux/config
# disable SELinux, change SELINUX=enforcing to SELINUX=disabled--

关闭防火墙
systemctl disable firewalld
systemctl stop firewalld
```

**6.2 配置 SSH**

互信为了方便后续操作，建议配置opentenbase机器间的 SSH 互信：

```
[opentenbase]
#切换opentenbase用户
su - opentenbase

# 192.168.2.136生成 SSH 密钥对
ssh-keygen -t rsa

# 将公钥复制到其他节点
ssh-copy-id -i ~/.ssh/id_rsa.pub opentenbase@192.168.2.136
ssh-copy-id -i ~/.ssh/id_rsa.pub opentenbase@192.168.2.137
ssh-copy-id -i ~/.ssh/id_rsa.pub opentenbase@192.168.2.138
```

**6.3 配置opentenbase环境变量**

集群所有机器都需要配置

```
[opentenbase]
$ vim ~/.bashrc
export OPENTENBASE_HOME=/data/opentenbase/install/opentenbase_bin_v2.6
export PATH=$OPENTENBASE_HOME/bin:OPENTENBASEHOME/bin:$PATH
export LD_LIBRARY_PATH=$OPENTENBASE_HOME/lib:OPENTENBASEHOME/lib:${LD_LIBRARY_PATH}
export LC_ALL=C
```

**6.4 配置root环境变量**

集群所有机器都需要配置,把opentenbase用户的$PATH环境变量添加到 etc/environment

```
[root]
cat /etc/environment
PATH=/data/opentenbase/install/opentenbase_bin_v2.6/bin:/data/opentenbase/.local/bin:/data/opentenbase/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin
```

**6.5 初始化pgxc\_ctl.conf文件**

```
[opentenbase]
mkdir /data/opentenbase/pgxc_ctl
cd /data/opentenbase/pgxc_ctl
vim pgxc_ctl.conf
```

如下，是结合上文描述的IP，端口，数据库目录，二进制目录等规划来写的pgxc\_ctl.conf文件。

pgxc\_ctl.conf配置如下：

```
#!/bin/bash
# Double Node Config

#主要调整IP地址即可
IP_1=192.168.2.136
IP_2=192.168.2.137

pgxcInstallDir=/data/opentenbase/install/opentenbase_bin_v2.6
pgxcOwner=opentenbase
defaultDatabase=postgres
pgxcUser=$pgxcOwner
tmpDir=/tmp
localTmpDir=$tmpDir
configBackup=n
configBackupHost=pgxc-linker
configBackupDir=$HOME/pgxc
configBackupFile=pgxc_ctl.bak


#---- GTM ----------
gtmName=gtm
gtmMasterServer=$IP_1
gtmMasterPort=50001
gtmMasterDir=/data/opentenbase/data/gtm
gtmExtraConfig=none
gtmMasterSpecificExtraConfig=none
gtmSlave=y
gtmSlaveServer=$IP_2
gtmSlavePort=50001
gtmSlaveDir=/data/opentenbase/data/gtm
gtmSlaveSpecificExtraConfig=none

#---- Coordinators -------
coordMasterDir=/data/opentenbase/data/coord
coordArchLogDir=/data/opentenbase/data/coord_archlog

coordNames=(cn001 cn002 )
coordPorts=(30004 30004 )
poolerPorts=(31110 31110 )
coordPgHbaEntries=(0.0.0.0/0)
coordMasterServers=($IP_1 IP1$IP_2)
coordMasterDirs=($coordMasterDir coordMasterDir$coordMasterDir)
coordMaxWALsernder=2
coordMaxWALSenders=($coordMaxWALsernder coordMaxWALsernder$coordMaxWALsernder )
coordSlave=n
coordSlaveSync=n
coordArchLogDirs=($coordArchLogDir coordArchLogDir$coordArchLogDir)

coordExtraConfig=coordExtraConfig
cat > $coordExtraConfig <eof $coordExtraPgHba <eof
#================================================
# Added to all the coordinator postgresql.conf
# Original: $coordExtraConfig

include_if_exists = '/data/opentenbase/global/global_opentenbase.conf'
 
wal_level = replica
wal_keep_segments = 256 
max_wal_senders = 4
archive_mode = on 
archive_timeout = 1800 
archive_command = 'echo 0' 
log_truncate_on_rotation = on 
log_filename = 'postgresql-%M.log' 
log_rotation_age = 4h 
log_rotation_size = 100MB
hot_standby = on 
wal_sender_timeout = 30min 
wal_receiver_timeout = 30min 
shared_buffers = 1024MB 
max_pool_size = 2000
log_statement = 'ddl'
log_destination = 'csvlog'
logging_collector = on
log_directory = 'pg_log'
listen_addresses = '*'
max_connections = 2000
 
EOF

coordSpecificExtraConfig=(none none)
coordExtraPgHba=coordExtraPgHba
cat > $coordExtraPgHba <<EOF
 
local   all             all                                     trust
host    all             all             0.0.0.0/0               trust
host    replication     all             0.0.0.0/0               trust
host    all             all             ::1/128                 trust
host    replication     all             ::1/128                 trust
 
 
EOF
 
 
coordSpecificExtraPgHba=(none none)
coordAdditionalSlaves=n 
cad1_Sync=n
 
#---- Datanodes ---------------------
dn1MstrDir=/data/opentenbase/data/dn001
dn2MstrDir=/data/opentenbase/data/dn002
dn1SlvDir=/data/opentenbase/data/dn001
dn2SlvDir=/data/opentenbase/data/dn002
dn1ALDir=/data/opentenbase/data/datanode_archlog
dn2ALDir=/data/opentenbase/data/datanode_archlog
 
primaryDatanode=dn001
datanodeNames=(dn001 dn002)
datanodePorts=(40004 40004)
datanodePoolerPorts=(41110 41110)
datanodePgHbaEntries=(0.0.0.0/0)
datanodeMasterServers=($IP_1 $IP_2)
datanodeMasterDirs=($dn1MstrDir $dn2MstrDir)
dnWALSndr=4
datanodeMaxWALSenders=($dnWALSndr $dnWALSndr)
 
datanodeSlave=y
datanodeSlaveServers=($IP_2 $IP_1)
datanodeSlavePorts=(50004 54004)
datanodeSlavePoolerPorts=(51110 51110)
datanodeSlaveSync=n
datanodeSlaveDirs=($dn1SlvDir $dn2SlvDir)
datanodeArchLogDirs=($dn1ALDir/dn001 $dn2ALDir/dn002)
 
datanodeExtraConfig=datanodeExtraConfig
cat > $datanodeExtraConfig <<EOF
#================================================
# Added to all the coordinator postgresql.conf
# Original: $datanodeExtraConfig
 
include_if_exists = '/data/opentenbase/global/global_opentenbase.conf'
listen_addresses = '*' 
wal_level = replica 
wal_keep_segments = 256 
max_wal_senders = 4
archive_mode = on 
archive_timeout = 1800 
archive_command = 'echo 0' 
log_directory = 'pg_log' 
logging_collector = on 
log_truncate_on_rotation = on 
log_filename = 'postgresql-%M.log' 
log_rotation_age = 4h 
log_rotation_size = 100MB
hot_standby = on 
wal_sender_timeout = 30min 
wal_receiver_timeout = 30min 
shared_buffers = 1024MB 
max_connections = 4000 
max_pool_size = 4000
log_statement = 'ddl'
log_destination = 'csvlog'
wal_buffers = 1GB
 
EOF
 
datanodeSpecificExtraConfig=(none none)
datanodeExtraPgHba=datanodeExtraPgHba
cat > $datanodeExtraPgHba <<EOF
 
local   all             all                                     trust
host    all             all             0.0.0.0/0               trust
host    replication     all             0.0.0.0/0               trust
host    all             all             ::1/128                 trust
host    replication     all             ::1/128                 trust
 
 
EOF
 
 
datanodeSpecificExtraPgHba=(none none)
 
datanodeAdditionalSlaves=n
walArchive=n
```

**6.6 分发二进制包**

在一个节点上配置好配置文件后，使用 pgxc\_ctl 工具将二进制包部署到所有节点：

```
[opentenbase@db1 pgxc_ctl]$ pgxc_ctl -c /data/opentenbase/pgxc_ctl/pgxc_ctl.conf
/usr/bin/bash
Installing pgxc_ctl_bash script as /home/opentenbase/pgxc_ctl/pgxc_ctl_bash.
Installing pgxc_ctl_bash script as /home/opentenbase/pgxc_ctl/pgxc_ctl_bash.
Reading configuration using /home/opentenbase/pgxc_ctl/pgxc_ctl_bash --home /home/opentenbase/pgxc_ctl --configuration /data/opentenbase/pgxc_ctl/pgxc_ctl.conf
Finished reading configuration.
******** PGXC_CTL START ***************

Current directory: /home/opentenbase/pgxc_ctl
PGXC 【这里输入指令】 deploy all
```

分发日志

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c2a16308-9694-11ef-a88b-fa163eb4f6be.png)

注：以下指令均在pgxc\_ctl终端命令行中

**6.7 初始化集群**

使用 pgxc\_ctl 工具初始化集群：

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c2b33196-9694-11ef-a88b-fa163eb4f6be.png)

初始化日志

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c2cae0a2-9694-11ef-a88b-fa163eb4f6be.png)

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c2eb1bb0-9694-11ef-a88b-fa163eb4f6be.png)

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c30c2328-9694-11ef-a88b-fa163eb4f6be.png)

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c317e6ea-9694-11ef-a88b-fa163eb4f6be.png)

**6.8 查看集群状态**

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c32ce31a-9694-11ef-a88b-fa163eb4f6be.png)

日志如下

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c337340a-9694-11ef-a88b-fa163eb4f6be.png)

**6.9 安装错误处理**

1）日志查看

一般init集群出错，终端会打印出错误日志，通过查看错误原因，更改配置即可，或者可以通过/data/opentenbase/pgxc\_ctl/pgxc\_log路径下的错误日志查看错误，排查配置文件的错误
![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c3433d9a-9694-11ef-a88b-fa163eb4f6be.png)

2）清理集群重新安装

通过运行 pgxc\_ctl 工具，执行clean all命令删除已经初始化的文件，修改pgxc\_ctl.conf文件，重新执行init all命令重新发起初始化。

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c34ff15c-9694-11ef-a88b-fa163eb4f6be.png)

3) 常见问题

• 执行inti all时，提示pg\_ctl命令找不到

这样的问题，通常是环境变量的问题，通过root用户配置/etc/environment 解决

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c35d8c72-9694-11ef-a88b-fa163eb4f6be.png)

• 执行inti all时，提示无法创建目录

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c36865b6-9694-11ef-a88b-fa163eb4f6be.png)

这样的问题，通常是分发节点上，没有创建对应的目录

解决方案：检查所有节点的目录配置，如果没有创建，创建即可。

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c3771a70-9694-11ef-a88b-fa163eb4f6be.png)

**七、集群访问**

访问OpenTenBase集群和访问单机的PostgreSQL基本上无差别，我们可以通过任意一个CN访问数据库集群：例如通过连接CN节点select pgxc\_node表即可查看集群的拓扑结构（当前的配置下备机不会展示在pgxc\_node中），在Linux命令行下通过psql访问的具体示例如下：

**7.1 登录cn主节点**

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c381e694-9694-11ef-a88b-fa163eb4f6be.png)

**7.2 使用数据库前需要创建default group以及sharding表**

OpenTenBase使用datanode group来增加节点的管理灵活度，要求有一个default group才能使用，因此需要预先创建；一般情况下，会将节点的所有datanode节点加入到default group里 另外一方面，OpenTenBase的数据分布为了增加灵活度，加了中间逻辑层来维护数据记录到物理节点的映射，我们叫sharding，所以需要预先创建sharding，命令如下：

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c390cfd8-9694-11ef-a88b-fa163eb4f6be.png)

**7.3 创建数据库，用户，创建表，增删查改等操作**

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c3a8126a-9694-11ef-a88b-fa163eb4f6be.png)

**八、集群启停**

**8.1 停止集群**

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241030_c3b4d9d2-9694-11ef-a88b-fa163eb4f6be.png)

停止日志


**8.2 启动集群**


**九、结语**

通过以上步骤，你已经成功地从源码编译并安装了 OpenTenBase V2.6。现在你可以开始使用 OpenTenBase 来管理你的分布式数据库集群。

<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

* AtomGit

  https://atomgit.com/opentenbase
* GitHub

  https://github.com/OpenTenBase
