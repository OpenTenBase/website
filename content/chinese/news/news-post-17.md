---
title: "支持空间数据库插件PostGIS，OpenTenBase V2.6.0 正式发布"
date: 2024-09-10T17:38:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-17.png
author: OpenTenBase
description: ""
---


在现已发布的OpenTenBase v2.6.0 版本中，我们引入了多个重要的新特性和改进，旨在提升用户体验和系统性能。以下是本次更新的详细内容。

**OpenTenBase v2.6.0 新特性介绍**

**1\. 功能增强**

**1.1 支持空间数据库插件PostGIS**

新增了对PostGIS插件的支持，用户现在可以在OpenTenBase中使用PostGIS插件进行空间数据的存储和查询。

使用场景：适用于需要处理地理空间数据的应用，如地图服务、地理信息系统（GIS）等。

**1.2 支持RPM包部署**

增加了对RPM包的支持，用户可以更方便地在基于RPM包管理的Linux发行版上部署OpenTenBase。

使用场景：适用于企业级应用的快速部署和版本管理。

**1.3 新增插件slowquery**

引入了slowquery插件，可以记录执行时间超过设定阈值的SQL查询，帮助用户识别性能瓶颈和优化性能。

使用场景：需要进行SQL性能调优。

**2\. 已知Bug修复**

**2.1 操作系统支持问题修复**

修复了多个操作系统上的兼容性问题，提高了系统的稳定性和兼容性。

使用场景：适用于多操作系统环境下的应用部署。

**2.2 make html编译问题修复**

修复了由于/doc/src/sgml/func.sgml引起的编译问题，确保文档编译的顺利进行。

使用场景：适用于需要生成HTML文档的用户。

**3\. 周边生态**

**3.1 容器化部署指南及文档**

新增了容器化部署案例的指南及文档，用户可以更轻松地在容器环境中部署OpenTenBase。

使用场景：适用于Docker和Kubernetes环境。

**3.2 KubeBlock插件支持及文档**

添加了对KubeBlock插件的支持，并提供了相应的文档。

使用场景：在Kubernetes集群中更高效地部署和管理OpenTenBase。

**3.3 Grafana/Prometheus监控指南及文档**

新增了基于Grafana/Prometheus的监控部署案例指南及文档，用户可以通过Grafana和Prometheus对系统进行全面的监控和告警。

使用场景：适用于需要实时监控和告警的企业级应用。

**OpenTenBase v2.6.0 新特性的价值**

这些新特性和改进主要面向以下几类用户和场景：

● 地理信息系统开发者：通过支持PostGIS插件，OpenTenBase为GIS开发者提供了强大的空间数据处理能力，解决了过去需要额外配置空间数据库的痛点。

● 企业级用户：RPM包支持和操作系统兼容性问题的修复，简化了在企业环境中的部署和维护工作。

● 性能调优工程师：slowquery插件帮助用户识别和优化SQL查询的性能瓶颈，提高系统的响应速度和稳定性。

● 容器化和集群管理用户：新增的容器化部署指南、KubeBlock插件支持以及Grafana/Prometheus监控指南，为用户提供了全面的文档和工具支持，简化了在容器和集群环境中的部署和管理流程。

**OpenTenBase v2.6.0 安装部署步骤**

**1. 前置准备**

安装依赖包：用户根据包管理工具的不同选用不同的指令

`yum -y install gcc make readline-devel zlib-devel openssl-devel uuid-devel bison flex  
`  

  

or

`aptinstall -y gcc make libreadline-dev zlib1g-dev libssl-dev libossp-uuid-dev bison flex  
`  

  

**2. 使用dnf工具安装OpenTenBase**

当前OpenTenBase v2.6.0已经支持rpm包部署，用户既可以直接使用rpm安装可用的OpenTenBase版本，也可以直接从OpenCloudOS的yum源中直接安装

`dnf install OpenTenBase*  
`  

  

安装完毕，检查 OpenTenBase 版本

```
[opentenbase@node1~]$ pg_ctl -V
pg_ctl (PostgreSQL)10.0OpenTenBaseV2

[opentenbase@node1~]$ pgxc_ctl -V
/usr/bin/bash Pgxc_ctl10.0OpenTenBaseV2
```

这种方法安装之后的OpenTenBase的二进制文件位于/usr/bin/目录之下。

**3. 创建用户，设定 SSH 秘钥，并创建数据目录**

```
useradd opentenbase
echo'opentenbase:opentenbase'| chpasswd
su -l opentenbase -c "mkdir home/opentenbase/.ssh -p"
su -l opentenbase -c "ssh-keygen -t rsa -f home/opentenbase/.ssh/id_rsa -N ''"
mkdir -p data/opentenbase
chown -R opentenbase:opentenbase data/opentenbase
```

  

\*上述操作在每一个节点上都要进行

**4. 配置节点间的ssh免密登陆**

要求节点之间可以使用ssh免输密码认证登陆，将每个节点的~/.ssh/id\_rsa.pub 复制到其他节点的~/.ssh/authorized\_keys中。

**5. 使用pgxc\_ctl构建集群**

接下来我们介绍用pgxc\_ctl方便构建OpenTenBase集群的方法。

我们采用1gtm1c2d的示例进行演示，更多的细节可以参照 https://www.opentenbase.org/

我们假定两个物理机的ip地址分别为192.168.56.101和192.168.56.102。

各个节点的配置方案如下

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20240920_63647f48-7712-11ef-926c-fa163eb4f6be.png)

\*接下来的操作在单个节点192.168.56.101上即可

● 创建pgxc\_ctl\_cluster.conf文件，保存到/home/opentenbase/pgxc\_ctl\_cluster.conf

`#!/bin/bash  
# Cluster Configuration  
  
IP_1=192.168.56.101  
IP_2=192.168.56.102  
  
pgxcInstallDir=/opt/opentenbase  
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
gtmMasterPort=5000  
gtmMasterDir=/data/opentenbase/cluster/gtm  
gtmExtraConfig=none  
gtmMasterSpecificExtraConfig=none  
gtmSlave=y  
# 新增,否则slave配置中为null  
gtmSlaveName=gtm-slave  
gtmSlaveServer=$IP_2  
gtmSlavePort=5000  
gtmSlaveDir=/data/opentenbase/cluster/gtm  
gtmSlaveSpecificExtraConfig=none  
  
#---- Coordinators -------  
coordMasterDir=/data/opentenbase/cluster/cn  
coordArchLogDir=/data/opentenbase/cluster/cn_archlog  
  
coordNames=(cn01 cn02)  
coordPorts=(30000 30000)  
poolerPorts=(31110 31110)  
coordPgHbaEntries=(0.0.0.0/0)  
coordMasterServers=($IP_1$IP_2)  
coordMasterDirs=($coordMasterDir$coordMasterDir)  
coordMaxWALsernder=2  
coordMaxWALSenders=($coordMaxWALsernder$coordMaxWALsernder)  
coordSlave=y  
coordSlaveSync=y  
coordArchLogDirs=($coordArchLogDir$coordArchLogDir)  
  
coordExtraConfig=coordExtraConfig  
cat > $coordExtraConfig <<EOF  
#================================================  
# Added to all the coordinator postgresql.conf  
# Original: $coordExtraConfig  
  
# include_if_exists = '/data/cerdb/db/cerdata/1.0/cluster/global/global_cerdb.conf'  
  
wal_level = replica  
wal_keep_segments = 256   
max_wal_senders = 4  
archive_mode = on   
archive_timeout = 1800   
archive_command ='echo 0'  
log_truncate_on_rotation = on   
log_filename ='postgresql-%M.log'  
log_rotation_age = 4h   
log_rotation_size = 100MB  
hot_standby = on   
wal_sender_timeout = 30min   
wal_receiver_timeout = 30min   
shared_buffers = 1024MB   
#max_pool_size = 2000  
max_pool_size = 65535  
log_statement ='ddl'  
log_destination ='csvlog'  
logging_collector = on  
log_directory ='pg_log'  
#listen_addresses = '*'  
listen_addresses ='0.0.0.0'  
max_connections = 2000  
  
EOF  
  
coordSpecificExtraConfig=(none none)  
coordExtraPgHba=coordExtraPgHba  
cat > $coordExtraPgHba <<EOF  
  
local   all             all                                     trust  
host    all             all             0.0.0.0/0               trust  
host    replication     all             0.0.0.0/0               trust  
host    all             all::1/128                 trust  
host    replication     all::1/128                 trust  
  
EOF  
  
coordSpecificExtraPgHba=(none none)  
coordAdditionalSlaves=n   
cad1_Sync=n  
  
#---- Datanodes ---------------------  
dn1MstrDir=/data/opentenbase/cluster/dn01  
dn2MstrDir=/data/opentenbase/cluster/dn02  
  
dn1SlvDir=/data/opentenbase/cluster/dn01-slave  
dn2SlvDir=/data/opentenbase/cluster/dn02-slave  
  
dn1ALDir=/data/opentenbase/cluster/dn_archlog  
dn2ALDir=/data/opentenbase/cluster/dn_archlog  
  
primaryDatanode=dn01  
datanodeNames=(dn01 dn02 )  
datanodePorts=(40000 40000 )  
datanodePoolerPorts=(41110 41110 )  
datanodePgHbaEntries=(0.0.0.0/0)  
datanodeMasterServers=($IP_1$IP_2)  
datanodeMasterDirs=($dn1MstrDir$dn2MstrDir)  
dnWALSndr=4  
datanodeMaxWALSenders=($dnWALSndr$dnWALSndr)  
  
datanodeSlave=y  
datanodeSlaveServers=($IP_2$IP_1)  
datanodeSlavePorts=(50000 50000 )  
datanodeSlavePoolerPorts=(51110 51110 )  
datanodeSlaveSync=y  
datanodeSlaveDirs=($dn1SlvDir$dn2SlvDir)  
datanodeArchLogDirs=($dn1ALDir/dn01$dn2ALDir/dn02)  
  
datanodeExtraConfig=datanodeExtraConfig  
cat > $datanodeExtraConfig <<EOF  
#================================================  
# Added to all the coordinator postgresql.conf  
# Original: $datanodeExtraConfig  
  
# include_if_exists = '/data/cerdb/db/cerdata/1.0/cluster/global/global_cerdb.conf'  
#listen_addresses = '*'   
listen_addresses ='0.0.0.0'  
wal_level = replica   
wal_keep_segments = 256   
max_wal_senders = 4  
archive_mode = on   
archive_timeout = 1800   
archive_command ='echo 0'  
log_directory ='pg_log'  
logging_collector = on   
log_truncate_on_rotation = on   
log_filename ='postgresql-%M.log'  
log_rotation_age = 4h   
log_rotation_size = 100MB  
hot_standby = on   
wal_sender_timeout = 30min   
wal_receiver_timeout = 30min   
shared_buffers = 1024MB   
max_connections = 4000   
#max_pool_size = 4000  
max_pool_size = 65535  
log_statement ='ddl'  
log_destination ='csvlog'  
wal_buffers = 1GB  
  
EOF  
  
datanodeSpecificExtraConfig=(none none )  
datanodeExtraPgHba=datanodeExtraPgHba  
cat > $datanodeExtraPgHba <<EOF  
  
local   all             all                                     trust  
host    all             all             0.0.0.0/0               trust  
host    replication     all             0.0.0.0/0               trust  
host    all             all::1/128                 trust  
host    replication     all::1/128                 trust  
  
EOF  
  
datanodeSpecificExtraPgHba=(none none )  
  
datanodeAdditionalSlaves=n  
walArchive=n  
  
`  

在节点192.168.56.101上运行

```
pgxc_ctl -c home/opentenbase/pgxc_ctl_cluster.conf init all  
pgxc_ctl -c home/opentenbase/pgxc_ctl_cluster.conf start all
```

测试数据库连接

```
psql -h 192.168.56.101 -p 30000 -d postgres -U opentenbase  
select * from pgxc_node;
```
  
<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

*   AtomGit
    
    https://atomgit.com/opentenbase/OpenTenBase
    
*   GitHub
    
    https://github.com/OpenTenBase/OpenTenBase