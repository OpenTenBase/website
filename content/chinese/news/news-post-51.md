---
title: "OpenAtom OpenTenBase 5.0 安装部署并集成 opentenbase_ai 插件与腾讯混元等大模型"
date: 2025-12-05T10:49:00+08:00
#image_webp: images/news/news-post-51.webp
image: images/news/news-post-51.png
author: OpenTenBase
description: ""
---

张瑞、颜渝韩

我们作为 OpenAtom OpenTenBase 校园大使，结合校园场景下的技术实践需求，完成了在 openEuler 22.03-LTS-SP4 系统上部署 OpenTenBase 5.0 并集成 AI 能力的全流程验证，现形成文档贡献。OpenTenBase 作为聚焦高可靠性与横向扩展的分布式关系型数据库，其 “GTM+CN+DN” 的架构设计清晰易懂，仅需普通 x86 设备即可搭建集群，既能满足校园内技术学习、实验验证的需求，也能支撑小型项目的稳定运行，非常契合高校技术实践场景。

在实际操作中，我们从源码编译入手，逐步完成单机集群的初始化与启动—— 通过明确 GTM（全局事务管理）、CN（协调节点）、DN（数据节点）的角色分工与端口配置，成功解决了组件间通信的基础问题；随后重点验证了 opentenbase_ai 插件的集成流程，发现该插件通过依赖 pgsql-http 扩展，能轻松实现 SQL 与 AI 模型的对接，尤其对腾讯混元、DeepSeek 等大模型的兼容性极佳，无需复杂代码开发即可调用 AI 能力，大幅降低了校园开发者的使用门槛。

在功能测试环节，我们针对校园场景设计了多个实用案例：如用 AI 生成数据库技术宣传语辅助校园推广、通过情感分析处理师生对系统的反馈意见、基于文本摘要提炼技术文档核心内容等，均能通过简单 SQL 语句高效实现，充分体现了 OpenTenBase“数据库 + AI” 融合的便捷性。

## 前言：认识 OpenTenBase 与 opentenbase_ai 插件

OpenTenBase 是一款聚焦高可靠性与多主节点数据同步的分布式关系型数据库集群系统，采用“无共享（Shared Nothing）” 架构设计——数据分片存储于多个物理节点，协调节点（CN）对外提供统一数据库视图，既能实现并行处理，又支持横向扩展，仅需普通 x86 服务器即可搭建集群。

其核心组件分为三类：

- 协调节点（Coordinator，简称 CN）：业务访问入口，负责 SQL 解析、查询规划与数据分发，仅存储全局元数据，不存储业务数据；多个 CN 位置对等，提供一致的数据库视图。
- 数据节点（Datanode，简称 DN）：存储业务数据分片，执行 CN 分发的查询 / 写入请求，支持主备架构保障数据可靠性。
- 全局事务管理器（GTM）：管理集群全局事务信息（如事务 ID）与全局对象（如序列），确保分布式事务的一致性。

而opentenbase_ai 插件是 OpenTenBase（基于 PostgreSQL 内核）的 AI 能力扩展，支持直接在 SQL 中调用大语言模型（LLM），实现文本生成、情感分析、摘要提取、向量生成、图像分析等功能，且兼容腾讯混元、阿里通义千问、DeepSeek 等主流大模型，为当前软硬件生态下的 “数据库 + AI” 智能应用提供底层支撑。

## 一、环境规划与准备

### 1.1 部署模式与节点规划

本文以单机测试环境为例（生产环境需多节点高可用部署），将 CN、DN（主备）、GTM 所有组件部署在同一台机器，通过不同端口区分。

| 组件角色 | 数据存储路径 | 默认端口（可自定义） | 说明 |
| --- | --- | --- | --- |
| GTM | `/home/opentenbase/data/gtm` | 6666 | 全局事务管理 |
| 协调节点（CN） | `/home/opentenbase/data/coord` | 5432 | 业务访问入口 |
| DN（主节点） | `/home/opentenbase/data/dn_m` | 5433 | 主数据节点，存储业务数据 |
| DN（备节点） | `/home/opentenbase/data/dn_s` | 5434 | 备数据节点，同步主节点数据 |

### 1.2 部署环境

| 类别 | 要求 |
| --- | --- |
| 操作系统 | openEuler 22.03-LTS-SP4（x86_64） |
| 内存 | 至少 4GB RAM（推荐 8GB+） |
| 磁盘空间 | 至少 20GB 可用空间（含源码、编译产物、数据） |
| 网络 | 单机环境无需跨节点通信，确保本地端口未被占用 |

### 1.3 安装系统依赖

openEuler 系统同时支持 yum 与dnf 包管理工具，dnf 作为yum 的升级版本，在依赖解析、安装效率上更具优势，因此推荐使用dnf 安装编译与运行所需依赖：

```bash
# 更新系统
sudo dnf update -y

# 安装基础工具与开发依赖
sudo dnf install -y \
  gcc \
  gcc-c++ \
  make \
  cmake \
  readline-devel \
  zlib-devel \
  openssl-devel \
  uuid-devel \
  bison \
  flex \
  git \
  libcurl-devel \
  libxml2-devel \
  libxslt-devel \
  perl-IPC-Run \
  perl-Test-Simple \
  tcl-devel \
  python3-devel \
  rpm-build \
  pkgconfig \
  krb5-devel \
  openldap-devel \
  vim
```

**下载zstd源码**

```bash
# 下载zstd源码
cd /tmp
wget https://github.com/facebook/zstd/releases/download/v1.5.2/zstd-1.5.2.tar.gz
tar -xzf zstd-1.5.2.tar.gz
cd zstd-1.5.2

# 编译安装
make
sudo make install PREFIX=/usr/local

# 更新库路径
sudo ldconfig

# 设置环境变量
export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"
```

**下载lz4源码**

```bash
# 下载lz4源码
cd /tmp
wget https://github.com/lz4/lz4/archive/v1.9.4.tar.gz
tar -xzf v1.9.4.tar.gz
cd lz4-1.9.4

# 编译安装
make
sudo make install PREFIX=/usr/local

# 更新库路径
sudo ldconfig

# 设置环境变量
export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"
export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"
```

**检查包管理状态**

```bash
# 首先检查当前的包管理器状态
sudo dnf clean all
```

**安装libxml2-devel及其依赖包**

```bash
# 安装libxml2-devel及其依赖包
sudo dnf install -y libxml2-devel libxml2 cmake-filesystem xz-devel zlib-devel pkgconfig

# 验证安装
rpm -qa | grep libxml2

# 检查xml2-config命令是否可用
which xml2-config

# 检查pkg-config是否能找到libxml-2.0
pkg-config --exists libxml-2.0 && echo "libxml2 found" || echo "libxml2 NOT found"

# 查看libxml2的配置信息
xml2-config --version
xml2-config --cflags
xml2-config --libs
```

<div class="text-center">
  <img src="../images/news-post-51-1.png" class="img-fluid" alt="依赖环境检查" />
</div>

**下载CLI11源码并进行编译**

```bash
# 下载lz4源码
cd /tmp
git clone https://github.com/CLIUtils/CLI11.git
cd CLI11

mkdir build
cd build
cmake ..
make
make install
dnf repolist
```

<div class="text-center">
  <img src="../images/news-post-51-2.png" class="img-fluid" alt="CLI11编译安装过程" />
</div>

## 二、OpenTenBase 5.0 源码编译与单机部署

### 2.1 创建专用用户

安装OpenTenBase集群的机器需要创建专用用户：

```bash
# 创建数据目录
sudo mkdir /data

# 创建opentenbase用户
sudo useradd -d /data/opentenbase -s /bin/bash -m opentenbase

# 设置密码
sudo passwd opentenbase
```

为opentenbase 用户添加sudo 权限的步骤如下：

使用 root 权限执行 visudo（visudo 会自动检查配置文件语法，避免错误）

```bash
su - root # 切换到 root 用户
visudo
```

在打开的配置文件中添加权限配置：在文件中找到类似root ALL=(ALL) ALL 的行，在其下方添加：

```text
# 允许 opentenbase 用户使用 sudo 执行所有命令（需要输入密码）
opentenbase ALL=(ALL) ALL
```

若希望免密码使用sudo（谨慎使用，存在安全风险），可改为：

```text
# 允许 opentenbase 免密码使用 sudo 执行所有命令
opentenbase ALL=(ALL) NOPASSWD: ALL
```

保存退出：

- vi/vim 编辑器中，按Esc 后输入`:wq` 保存退出。
- 若配置有误，visudo 会提示错误，按提示修改即可。

切回到opentenbase用户

```bash
su - opentenbase
```

### 2.2 编译配置与安装（opentenbase 用户下进行）

**获取源码**

选择/data/opentenbase目录作为源码存放路径

```bash
#获取源码
cd /data/opentenbase
git clone https://gitee.com/mirrors/OpenTenBase.git
```

### 2.3 配置环境变量（opentenbase 用户下进行）

```bash
# 设置环境变量
export SOURCECODE_PATH=/data/opentenbase/OpenTenBase
export INSTALL_PATH=/data/opentenbase/install
```

### 2.4编译源码（opentenbase 用户下进行）

```bash
# 进入源码目录
cd ${SOURCECODE_PATH}

chmod +x configure*

# 清除旧编译文件
make distclean 2>/dev/null || true
rm -rf /data/opentenbase/install/opentenbase_bin_v2.0
rm -f config.status config.log

# 配置编译，添加SSE4.2支持
CFLAGS="-g -O2 -w -msse4.2 -mcrc32 -DNOLIC" \
CXXFLAGS="-g -O2 -w -msse4.2 -mcrc32 -DNOLIC" \
./configure --prefix=/data/opentenbase/install/opentenbase_bin_v2.0 \
   --enable-user-switch \
   --with-openssl \
   --with-ossp-uuid \
   --with-libxml

# 编译
make
make install

# 编译contrib模块
chmod +x contrib/pgxc_ctl/make_signature
cd contrib
make
make install
```

<div class="text-center">
  <img src="../images/news-post-51-3.png" class="img-fluid" alt="OpenTenBase源码编译过程" />
</div>

<div class="text-center">
  <img src="../images/news-post-51-4.png" class="img-fluid" alt="OpenTenBase编译结果" />
</div>

<div class="text-center">
  <img src="../images/news-post-51-5.png" class="img-fluid" alt="OpenTenBase安装过程" />
</div>

### 2.5 设置环境变量（opentenbase 用户下进行）

```bash
vim ~/.bashrc

#将如下命令添加到文件末尾：
export OPENTENBASE_HOME=/data/opentenbase/install/opentenbase_bin_v2.0
export PATH=$OPENTENBASE_HOME/bin:$PATH
export LD_LIBRARY_PATH=$OPENTENBASE_HOME/lib:${LD_LIBRARY_PATH}
export LC_ALL=C
```

### 2.5 配置防火墙

```bash
# 开放GTM端口
sudo firewall-cmd --permanent --add-port=6666/tcp

# 开放Coordinator端口
sudo firewall-cmd --permanent --add-port=30004/tcp

# 开放Datanode端口
sudo firewall-cmd --permanent --add-port=20008/tcp

# 重新加载防火墙规则
sudo firewall-cmd --reload
```

或直接关闭防火墙（仅用于测试环境）：

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```

### 2.6 关闭 SELINUX

```bash
sudo vim /etc/sysconfig/selinux
```

- 临时关闭 SELinux
  - 使用命令getenforce查看 SELinux 当前状态，如果返回Enforcing，表明 SELinux 处于强制模式。
  - 执行命令setenforce 0，将 SELinux 设置为宽容模式（Permissive），即临时关闭 SELinux。此时 SELinux 会记录违反策略的行为但不会阻止它们，不过系统重新启动后 SELinux 将会恢复到之前的状态。
- 永久关闭 SELinux
  - 编辑 SELinux 配置文件/etc/selinux/config，可以使用命令vi /etc/selinux/config。
  - 在文件中找到SELINUX=enforcing这一行，将其改为SELINUX=disabled。
  - 保存文件并重启系统，命令为reboot。重启后 SELinux 就会被永久关闭。
- 示例（永久关闭）：

```text
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
# SELINUX=enforcing
SELINUX=disabled
# ^^^^^^^^
# SELINUXTYPE= can take one of these three values:
# targeted - Targeted processes are protected,
# minimum - Modification of targeted policy. Only selected processes are protected.
# mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

### 2.7 创建配置文件

```bash
# 创建配置文件
cat > /data/opentenbase/install/opentenbase_bin_v2.0/pgxc_ctl.conf << 'EOF'
#!/usr/bin/env bash

#---- OVERALL -----------------------------------------------------------------------------
pgxcOwner=opentenbase
pgxcUser=$pgxcOwner
pgxcInstallDir=/data/opentenbase/install/opentenbase_bin_v2.0
tmpDir=/tmp
localTmpDir=$tmpDir
configBackup=y
configBackupHost=localhost
configBackupDir=$HOME/pgxc
configBackupFile=pgxc_ctl.bak

#---- GTM --------------------------------------------------------------------------------
gtmName=gtm
gtmMasterServer=localhost
gtmMasterPort=6666
gtmMasterDir=/data/opentenbase/data/gtm
gtmExtraConfig=none
gtmMasterSpecificExtraConfig=none

gtmSlave=n
gtmSlaveName=gtmSlave
gtmSlaveServer=none
gtmSlavePort=20001
gtmSlaveDir=none
gtmSlaveSpecificExtraConfig=none

gtmProxy=n
gtmProxyNames=()
gtmProxyServers=()
gtmProxyPorts=()
gtmProxyDirs=()
gtmPxyExtraConfig=none
gtmPxySpecificExtraConfig=()

#---- Coordinators --------------------------------------------------------------------
coordMasterDir=/data/opentenbase/data/coord_master
coordSlaveDir=/data/opentenbase/data/coord_slave
coordArchLogDir=/data/opentenbase/data/coord_archlog

coordNames=(cn001)
coordPorts=(30004)
poolerPorts=(30014)
coordForwardPorts=(30024)
coordPgHbaEntries=(0.0.0.0/0)

coordMasterServers=(localhost)
coordMasterDirs=(/data/opentenbase/data/coord_master/cn001)
coordMaxWALsender=5
coordMaxWALSenders=(5)

coordSlave=n
coordSlaveSync=n
coordSlaveServers=(none)
coordSlavePorts=(30005)
coordSlavePoolerPorts=(30015)
coordSlaveForwardPorts=(30025)
coordSlaveDirs=(none)
coordArchLogDirs=(none)

coordExtraConfig=none
coordSpecificExtraConfig=(none)
coordSpecificExtraPgHba=(none)

#---- Datanodes -----------------------------------------------------------------------
datanodeMasterDir=/data/opentenbase/data/dn_master
datanodeSlaveDir=/data/opentenbase/data/dn_slave
datanodeArchLogDir=/data/opentenbase/data/datanode_archlog

primaryDatanode=dn001
datanodeNames=(dn001)
datanodePorts=(20008)
datanodePoolerPorts=(20018)
datanodeForwardPorts=(20028)
datanodePgHbaEntries=(0.0.0.0/0)

datanodeMasterServers=(localhost)
datanodeMasterDirs=(/data/opentenbase/data/dn_master/dn001)
datanodeMaxWalSender=5
datanodeMaxWALSenders=(5)

datanodeSlave=n
datanodeSlaveServers=(none)
datanodeSlavePorts=(20009)
datanodeSlavePoolerPorts=(20019)
datanodeSlaveForwardPorts=(20029)
datanodeSlaveDirs=(none)
datanodeArchLogDirs=(none)

datanodeExtraConfig=none
datanodeSpecificExtraConfig=(none)
datanodeSpecificExtraPgHba=(none)

walArchive=n
EOF
```

## 三、启动数据库集群

### 3.1 SSH 免密登录（opentenbase 用户下进行）

```bash
su - opentenbase
ssh-keygen -t rsa # 一路回车即可
ssh-copy-id -i ~/.ssh/id_rsa.pub opentenbase@localhost

# 修复权限
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/authorized_keys

# 测试ssh登录
ssh localhost

# 如果ssh登录需要密码，则检查/etc/ssh/sshd_config配置：
sudo vim /etc/ssh/sshd_config

#确保以下配置未被注释（去掉前面的#号）：
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# 重启ssh服务
sudo systemctl restart sshd

# 再次测试
ssh localhost
```

### 3.2 初始化集群

```text
pgxc_ctl -c /data/opentenbase/install/opentenbase_bin_v2.0/pgxc_ctl.conf

# 依次执行以下命令
# 分发二进制包
deploy all

# 初始化集群
init all
```

注：

- 监控集群：monitor all
- 停止集群：stop all（关闭系统前务必先使用该命令停止集群）
- 清除集群：clean all

<div class="text-center">
  <img src="../images/news-post-51-6.png" class="img-fluid" alt="OpenTenBase集群初始化" />
</div>

### 3.3 登录数据库

```bash
psql -h 127.0.0.1 -p 30004 -d postgres -U opentenbase
```

### 3.4 创建 default group 和 sharding 表

```sql
postgres=# create default node group default_group with (dn001);
CREATE NODE GROUP
postgres=# create sharding group to group default_group;
CREATE SHARDING GROUP
```

### 3.5 测试数据库 - 创建测试数据库和表

```sql
-- 创建测试数据库
CREATE DATABASE simple_db;

-- 创建普通用户
CREATE USER simple_user WITH PASSWORD 'simple123';

-- 授权用户管理数据库
GRANT ALL ON DATABASE simple_db TO simple_user;

-- 切换到目标数据库
\c simple_db simple_user

-- 创建基础分片表（按name字段分片）
CREATE TABLE user_info (
   id INT PRIMARY KEY,
   name VARCHAR(50) NOT NULL,
   age INT,
   join_date DATE DEFAULT CURRENT_DATE
) DISTRIBUTE BY SHARD(name);

-- 插入几条测试数据
INSERT INTO user_info(id, name, age) VALUES
(1, '张三', 22),
(2, '李四', 25),
(3, '王五', 23);

-- 简单查询
SELECT * FROM user_info WHERE age > 22;

-- 更新数据
UPDATE user_info SET age = 24 WHERE name = '王五';

-- 再次查询验证更新
SELECT name, age FROM user_info;
```

<div class="text-center">
  <img src="../images/news-post-51-7.png" class="img-fluid" alt="数据库测试结果" />
</div>

## 四、opentenbase_ai 插件使用文档

### 4.1 前提条件

- HTTP 扩展（该插件依赖 HTTP 扩展进行 API 调用）
- 依赖：需要 pgsql-http 插件（处理 HTTP 请求）

### 4.2 安装依赖工具和库

**步骤 1：查询系统中可用的 PostgreSQL 开发包**

先确认 openEuler 仓库中 PostgreSQL 开发包的实际名称：

```bash
sudo dnf search postgresql | grep devel
```

执行后会显示类似结果（根据系统版本可能略有不同）：

<div class="text-center">
  <img src="../images/news-post-51-8.png" class="img-fluid" alt="PostgreSQL开发包查询结果" />
</div>

根据 openEuler 系统中查询到的包信息，PostgreSQL 开发相关的包名称与预期不同，需安装对应的开发包以满足 pgsql-http 插件的编译需求

**步骤 2：安装必要的 PostgreSQL 开发包**

从查询结果来看，需要安装以下包（涵盖编译pgsql-http 所需的头文件和库）：

```bash
sudo dnf install -y postgresql-server-devel libpq-devel
```

**步骤 3：确认 pg_config 路径**

安装完成后，确认pg_config（编译插件必须的配置工具）是否存在：

```bash
which pg_config
```

### 4.3 下载并编译 pgsql-http 源码

**步骤 1：下载源码**

```bash
# 克隆 pgsql-http 源码仓库
cd /tmp
git clone https://github.com/pramsey/pgsql-http.git
cd pgsql-http

# 切换到 1.7.0 标签
git checkout v1.7.0

# 备份原始 http.c（可选，用于出错时恢复）
cp http.c http.c.bak
```

**步骤 2：修改 http.c 解决编译错误**

1. 使用编辑器打开 http.c：

```bash
vim http.c
```

2. 删除 guc_malloc 和 guc_strdup 的静态声明

搜索以下代码（在文件中部，#if PG_VERSION_NUM < 160000 块内）：

删除以下两段代码块

```c
static void *
guc_malloc(int elevel, size_t size)
{
void *data;

/* Avoid unportable behavior of malloc(0) */
if (size == 0)
size = 1;
data = malloc(size);
if (data == NULL)
ereport(elevel,
(errcode(ERRCODE_OUT_OF_MEMORY),
 errmsg("out of memory")));
return data;
}

static char *
guc_strdup(int elevel, const char *src)
{
size_t len = strlen(src) + 1;
char *dup = guc_malloc(elevel, len);
memcpy(dup, src, len);
return dup;
}
```

删除这两个函数的定义（因为 OpenTenBase 已将它们声明为 extern，静态定义导致冲突）。

<div class="text-center">
  <img src="../images/news-post-51-9.png" class="img-fluid" alt="修改pgsql-http源码" />
</div>

3.补全 pg_any_to_server 的参数

搜索以下代码行（在http_request 函数内）：

```c
content_str = pg_any_to_server(si_data.data, si_data.len, content_charset);
```

<div class="text-center">
  <img src="../images/news-post-51-10.png" class="img-fluid" alt="pg_any_to_server原始参数" />
</div>

修改为（补充 OpenTenBase 要求的后两个参数）：

```c
content_str = pg_any_to_server(si_data.data, si_data.len, content_charset, true, true);
```

<div class="text-center">
  <img src="../images/news-post-51-11.png" class="img-fluid" alt="pg_any_to_server参数修改" />
</div>

4.替换 DatumGetJsonb 宏定义

在源码中搜索以下代码（约在文件中部，#include <utils/jsonb.h> 之后）：

```c
#if PG_VERSION_NUM < 110000
#define PG_GETARG_JSONB_P(x) DatumGetJsonb(PG_GETARG_DATUM(x))
#endif
```

<div class="text-center">
  <img src="../images/news-post-51-12.png" class="img-fluid" alt="DatumGetJsonb宏定义" />
</div>

修改为：

```c
#if PG_VERSION_NUM < 110000
#define PG_GETARG_JSONB_P(x) ((Jsonb *)PG_DETOAST_DATUM(PG_GETARG_DATUM(x)))
#endif
```

<div class="text-center">
  <img src="../images/news-post-51-13.png" class="img-fluid" alt="DatumGetJsonb宏定义修改" />
</div>

说明：OpenTenBase 可能不支持 DatumGetJsonb 函数，直接使用 PG_DETOAST_DATUM 将 Datum 转换为 Jsonb 指针，绕过符号依赖。

5.检查 urlencode_jsonb 函数中的JSONB 处理

在urlencode_jsonb 函数中，有一段根据 PostgreSQL 版本处理 JSONB 的代码：

```c
#if PG_VERSION_NUM < 130000
{
  JsonbValue k;
  k.type = jbvString;
  k.val.string.val = key;
  k.val.string.len = strlen(key);
v = *findJsonbValueFromContainer(&jb->root, JB_FOBJECT, &k);
}
#else
getKeyJsonValueFromContainer(&jb->root, key, strlen(key), &v);
#endif
```

<div class="text-center">
  <img src="../images/news-post-51-14.png" class="img-fluid" alt="urlencode_jsonb原始代码" />
</div>

修改为：

```c
#if PG_VERSION_NUM < 130000
{
  JsonbValue k;
  k.type = jbvString;
  k.val.string.val = key;
  k.val.string.len = strlen(key);
  // 显式获取 Jsonb 容器指针，避免隐式依赖 DatumGetJsonb
  JsonbContainer *container = &jb->root;
  v = *findJsonbValueFromContainer(container, JB_FOBJECT, &k);
}
#else
getKeyJsonValueFromContainer(&jb->root, key, strlen(key), &v);
#endif
```

<div class="text-center">
  <img src="../images/news-post-51-15.png" class="img-fluid" alt="urlencode_jsonb修改后代码" />
</div>

**步骤 3： 编译安装插件**

```bash
cd /tmp/pgsql-http

# 清理旧编译文件
make clean

# 编译
make

# 安装
make install
```

**步骤 4： 验证扩展安装**

1. 切换到opentenbase 用户，登录数据库验证：

```bash
su - opentenbase
psql -h localhost -p 30004 -d postgres -U opentenbase

# 尝试创建扩展
CREATE EXTENSION http;
```

2. 安装 opentenbase_ai 扩展

```sql
CREATE EXTENSION opentenbase_ai CASCADE;
```

3. 可通过以下命令验证扩展是否正常安装：

```sql
SELECT * FROM pg_extension WHERE extname = 'opentenbase_ai';
```

若查询结果存在，说明扩展已可用。

### 4.4 配置混元大模型实现文本生成

**前提条件**

1. 已成功安装opentenbase_ai 扩展（可通过SELECT * FROM pg_extension WHERE extname = 'opentenbase_ai'; 验证）。
2. 已获取腾讯混元大模型的 API 密钥（token）（需从腾讯云控制台申请，参考混元大模型官方文档）。

**详细步骤**

**步骤 1：添加混元大模型定义到系统**

通过ai.add_completion_model 函数将混元大模型的配置信息存入数据库，供后续调用：

```sql
-- 添加混元大模型定义
SELECT ai.add_completion_model(
   model_name => 'hunyuan_chat', -- 自定义模型名称（后续调用时使用）
   uri => 'https://api.hunyuan.cloud.tencent.com/v1/chat/completions', -- 混元API接口地址
   default_args => '{"model": "hunyuan-lite"}'::jsonb, -- 默认参数（指定模型版本，如"hunyuan-lite"轻量版）
   token => 'your_hunyuan_api_key', -- 替换为你的混元API密钥
   model_provider => 'tencent' -- 模型提供商（固定为"tencent"）
);
```

- 执行成功后，返回t 表示模型定义添加成功。

**步骤 2：设置混元大模型为默认模型（可选）**

若后续希望默认使用混元模型，可通过SET 命令配置会话级参数：

```sql
-- 设置默认Completion模型为混元
SET ai.completion_model = 'hunyuan_chat';
```

- 若不设置默认，后续调用时需通过model_name 参数指定模型。

**步骤 3：使用混元大模型生成文本**

通过ai.generate_text 函数调用混元模型生成内容（以“智能手表” 产品描述为例）：

```sql
-- 调用混元模型生成文本
SELECT ai.generate_text(
   prompt => '为以下产品写一段吸引人的描述：智能手表', -- 提示词（输入需求）
   model_name => 'hunyuan_chat' -- 若已设置默认模型，可省略此参数
);
```

<div class="text-center">
  <img src="../images/news-post-51-16.png" class="img-fluid" alt="混元大模型文本生成结果" />
</div>

<div class="text-center">
  <img src="../images/news-post-51-17.png" class="img-fluid" alt="文本生成与情感分析结果" />
</div>

### 4.5 配置混元大模型实现图像解析

**步骤 1： 添加图像模型：**

```sql
SELECT ai.add_image_model(
   model_name => 'hunyuan-vision',
   uri => 'https://api.hunyuan.cloud.tencent.com/v1/chat/completions',
   default_args => '{"model": "hunyuan-vision"}'::jsonb,
   token => 'your_hunyuan_api_key', -- 替换为你的混元API密钥，可以与文本生成模型的秘钥相同
   model_provider => 'tencent'
);
```

**步骤 2： 设置图像模型为默认模型：**

```sql
SET ai.image_model = 'hunyuan-vision';
```

**步骤 3： 使用大模型解析图像：**

https://gips0.baidu.com/it/u=3602773692,1512483864&fm=3028&app=3028&f=JPEG&fmt=auto?w=960&h=1280

示例用图片

<div class="text-center">
  <img src="../images/news-post-51-18.png" class="img-fluid" alt="图像解析示例图片" />
</div>

```sql
select ai.image(' 这 张 图 片 中 有 什 么 ',
'http://gips0.baidu.com/it/u=3602773692,1512483864&fm=3028&app=3028&f=JPEG&fmt=auto?w=960&h=1280');
```

<div class="text-center">
  <img src="../images/news-post-51-19.png" class="img-fluid" alt="大模型图像解析结果" />
</div>

### 4.6 超时问题

使用大模型能力时，可能因为响应速度慢导致结果解析超时，可以设置超时时间来解决：

```sql
-- 设置请求的超时时间，单位毫秒
SET http.timeout_msec TO 200000;
```

如果还是超时，可以尝试修改系统 dns 服务器配置：

```bash
sudo vim /etc/resolv.conf

# 在开头加入以下三行
nameserver 8.8.8.8
nameserver 1.1.1.1
nameserver 114.114.114
```

## 号外

OpenTenBase 城市行南京站将于明天举行，欢迎南京及周边城市的朋友到场交流！

<div class="text-center">
  <img src="../images/news-post-51-20.png" class="img-fluid" alt="OpenTenBase城市行南京站活动海报" />
</div>
