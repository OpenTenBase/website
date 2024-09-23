---
title: "社区贡献 | OpenTenBase集群部署初探"
date: 2024-08-30T12:48:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-12.png
author: OpenTenBase
description: ""
---


**贡献者介绍**
---------

胖头鱼的鱼缸（尹海文）

*   OpenTenBase ACE  
    
*   Oracle ACE Pro: Database（Oracle与MySQL）
    
*   PostgreSQL ACE Partner
    
*   10年数据库行业经验，现主要从事数据库服务工作
    
*   拥有OCM 11g/12c/19c、MySQL 8.0 OCP、Exadata、CDP等认证
    

**集群规划**
-------

按照下面的规划进行搭建，本次使用的操作系统是CentOS 7.9：

<img src=../images/news-post-12-2.png class="img-fluid" /><br/>

模块交互关系如下：  

<img src=../images/news-post-12-3.png class="img-fluid" /><br/>

**部署集群**
----

### **1\.** **操作系统配置**

配置HOSTS\*

```
cat >> etc/hosts <<EOF  
10.10.10.101 opt01  
10.10.10.102 opt02  
10.10.10.103 opt03  
10.10.10.104 opt04  
10.10.10.105 opt05  
EOF  
```  

关闭防火墙

```
systemctl stop firewalld.service   
systemctl disable firewalld.service  
```  

关闭SELinux

```
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' etc/selinux/config  
setenforce 0  
```  

创建用户

```
mkdir data  
useradd -d data/opentenbase -s bin/bash -m opentenbase  
passwd opentenbase  
```  

配置ssh互信

```
su - opentenbase  
ssh-keygen -t rsa  
ssh-copy-id -i ~/.ssh/id_rsa.pub opt01  
ssh-copy-id -i ~/.ssh/id_rsa.pub opt02  
ssh-copy-id -i ~/.ssh/id_rsa.pub opt03  
ssh-copy-id -i ~/.ssh/id_rsa.pub opt04  
ssh-copy-id -i ~/.ssh/id_rsa.pub opt05  
``` 

安装依赖软件

```
rm -rf etc/yum.repos.d/CentOS-*  
cat > etc/yum.repos.d/redhat.repo <<EOF  
[CentOS]  
name=CentOS  
baseurl=https://mirrors.aliyun.com/centos-vault/centos/7/os/x86_64  
enabled=1  
gpgcheck=0  
EOF  
  
yum -y install gcc make readline-devel zlib-devel openssl-devel uuid-devel bison flex git  
``` 

### **2\.** **数据库软件下载与安装**

仅需在一个节点执行即可

```
su - opentenbase  
git clone https://github.com/OpenTenBase/OpenTenBase  

在下面地址下载最新安装包

https://github.com/OpenTenBase/OpenTenBase/tags  

tar -xvf OpenTenBase-2.6.0.tar.gz  
  
cd data/opentenbase/OpenTenBase-2.6.0/  
chmod +x configure*  
./configure --prefix=/data/opentenbase/install/opentenbase_bin_v2.0 --enable-user-switch --with-openssl --with-ossp-uuid CFLAGS=-g  
make clean  
make -sj  
make install  
chmod +x contrib/pgxc_ctl/make_signature  
cd contrib  
make -sj  
make install  
  
配置环境变量（每个节点都需要执行）  

cat >> ~/.bashrc <<EOF  
export OPENTENBASE_HOME=/data/opentenbase/install/opentenbase_bin_v2.0  
export PATH=\$OPENTENBASE_HOME/bin:\$PATH  
export LD_LIBRARY_PATH=\$OPENTENBASE_HOME/lib:\$LD_LIBRARY_PATH  
export LC_ALL=C  
EOF  
```  
### **3\.** **配置集群**

```
mkdir data/opentenbase/pgxc_ctl  
cd data/opentenbase/pgxc_ctl  
  
#根据官方文档配置文件进行调整：  
vim pgxc_ctl.conf  
#!/bin/bash  

Haiwen Yin's OpenTenBase Cluster Configuration  
  
IP_1=10.10.10.101  
IP_2=10.10.10.102  
IP_3=10.10.10.103  
IP_4=10.10.10.104  
IP_5=10.10.10.105  
  
pgxcInstallDir=/data/opentenbase/install/opentenbase_bin_v2.0  
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
  
coordNames=(cn01 cn02)  
coordPorts=(30004 30004)  
poolerPorts=(31110 31110)  
coordPgHbaEntries=(0.0.0.0/0)  
coordMasterServers=($IP_1 $IP_2)  
coordMasterDirs=($coordMasterDir $coordMasterDir)  
coordMaxWALsernder=2  
coordMaxWALSenders=($coordMaxWALsernder $coordMaxWALsernder)  
coordSlave=n  
coordSlaveSync=n  
coordArchLogDirs=($coordArchLogDir $coordArchLogDir)  
  
coordExtraConfig=coordExtraConfig  
cat > $coordExtraConfig <<EOF  
#================================================  

Added to all the coordinator postgresql.conf  

Original: $coordExtraConfig  
  
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
  
|local|all|all||trust|
|host|all|all|0.0.0.0/0|trust|
|host|replication|all|0.0.0.0/0|trust|
|host|all|all|::1/128|trust|
|host|replication|all|::1/128|trust|
  
  
EOF  
  
  
coordSpecificExtraPgHba=(none none)  
coordAdditionalSlaves=n	  
cad1_Sync=n  
  
#---- Datanodes ---------------------  
dn1MstrDir=/data/opentenbase/data/dn001  
dn2MstrDir=/data/opentenbase/data/dn002  
dn3MstrDir=/data/opentenbase/data/dn003  
dn1SlvDir=/data/opentenbase/data/dn001  
dn2SlvDir=/data/opentenbase/data/dn002  
dn3SlvDir=/data/opentenbase/data/dn003  
dn1ALDir=/data/opentenbase/data/datanode_archlog  
dn2ALDir=/data/opentenbase/data/datanode_archlog  
dn3ALDir=/data/opentenbase/data/datanode_archlog  
  
primaryDatanode=dn01  
datanodeNames=(dn01 dn02 dn03)  
datanodePorts=(40004 40004 40004)  
datanodePoolerPorts=(41110 41110 41110)  
datanodePgHbaEntries=(0.0.0.0/0)  
datanodeMasterServers=($IP_3 $IP_4 $IP_5)  
datanodeMasterDirs=($dn1MstrDir $dn2MstrDir $dn3MstrDir)  
dnWALSndr=4  
datanodeMaxWALSenders=($dnWALSndr $dnWALSndr $dnWALSndr)  
  
datanodeSlave=y  
datanodeSlaveServers=($IP_4 $IP_5 $IP_3)  
datanodeSlavePorts=(50004 50004 50004)  
datanodeSlavePoolerPorts=(51110 51110 51110)  
datanodeSlaveSync=n  
datanodeSlaveDirs=($dn1SlvDir $dn2SlvDir $dn3SlvDir)  
datanodeArchLogDirs=($dn1ALDir/dn001 $dn2ALDir/dn002 $dn3ALDir/dn003)  
  
datanodeExtraConfig=datanodeExtraConfig  
cat > $datanodeExtraConfig <<EOF  
#================================================  

Added to all the coordinator postgresql.conf  

Original: $datanodeExtraConfig  
  
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
  
datanodeSpecificExtraConfig=(none none none)  
datanodeExtraPgHba=datanodeExtraPgHba  
cat > $datanodeExtraPgHba <<EOF  
  
|local|all|all||trust|
|host|all|all|0.0.0.0/0|trust|
|host|replication|all|0.0.0.0/0|trust|
|host|all|all|::1/128|trust|
|host|replication|all|::1/128|trust|
  
EOF  
  
  
datanodeSpecificExtraPgHba=(none none none)  
  
datanodeAdditionalSlaves=n  
walArchive=n  
```  

### **4.** **分发软件**

```
pgxc_ctl  
> deploy all  
``` 

<img src=../images/news-post-12-4.png class="img-fluid" /><br/>

### **5\.** **启动数据库**

```
pgxc_ctl  
> init all  
``` 

### **6\.** **查看集群状态**

```
pgxc_ctl  
> monitor all  
``` 

<img src=../images/news-post-12-5.png class="img-fluid" /><br/>

**访问数据库**
---------

```sql
psql -h 10.10.10.101 -p 30004 -d postgres -U opentenbase
select * from pgxc_node;
```

<img src=../images/news-post-12-6.png class="img-fluid" /><br/>

**总结**
------

整体来说OpenTenBase的安装过程还是比较快捷的。

但是有几点需要记录一下：

1.  使用RHEL8.10安装有点问题，不知道是操作还是其他的原因，回头再整一次
    
2.  uuid-devel没有在标准iso和epel中，安装有点点麻烦

<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

*   AtomGit
    
    https://atomgit.com/opentenbase/OpenTenBase
    
*   GitHub
    
    https://github.com/OpenTenBase/OpenTenBase