---
title: "社区贡献 | 手把手教你在OpenCloudOS上部署OpenTenBase"
date: 2024-09-04T13:39:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-15.png
author: OpenTenBase
description: ""
---

本文将介绍两种 OpenTenBase 数据库的安装方法。

  

1.  源码编译安装。
    
2.  使用 DNF 安装。
    

**OpenTenBase 源码编译安装**
----------------------

本文实验环境为 OpenCloudOS 操作系统，关于该 OS 的内容请参阅：

*   初探 OpenCloudOS 操作系统
    

### 安装依赖

这里需要开启 PowerTools 仓库，以安装 `uuid-devel`  
依赖。

    [root@oc8 ~]# dnf -y install gcc make readline-devel zlib-devel openssl-devel uuid-devel bison flex git readline-devel
    上次元数据过期检查：0:00:12 前，执行于 2024年09月03日 星期二 00时38分01秒。
    OpenCloudOS - BaseOS
    OpenCloudOS - AppStream
    OpenCloudOS - PowerTools
    软件包 gcc-8.5.0-22.oc8.1.x86_64 已安装。
    软件包 make-1:4.2.1-11.oc8.x86_64 已安装。
    软件包 zlib-devel-1.2.11-25.oc8.x86_64 已安装。
    软件包 openssl-devel-1:1.1.1k-12.oc8.1.x86_64 已安装。
    软件包 bison-3.0.4-10.oc8.x86_64 已安装。
    软件包 flex-2.6.1-9.oc8.x86_64 已安装。
    软件包 git-2.43.5-1.oc8.x86_64 已安装。
    依赖关系解决。=========================================================================================== 
    软件包                                                     架构
    ===========================================================================================
    安装:
    readline-devel                                             x86_64 
    uuid-devel                                                 x86_64
    安装依赖关系: 
    ncurses-c++-libs                                           x86_64 ncurses-devel                                              x86_64 uuid                                                       x86_64
    事务概要
    ===========================================================================================
    安装  5 软件包总下载：879 k安装大小：1.6 M下载软件包：
    (1/5): ncurses-c++-libs-6.1-10.20180224.oc8.x86_64.rpm     186 kB/s |  57 kB     00:00
    (2/5): readline-devel-7.0-10.oc8.x86_64.rpm                503 kB/s | 203 kB     00:00
    (3/5): uuid-1.6.2-43.oc8.x86_64.rpm                        561 kB/s |  63 kB     00:00
    (4/5): uuid-devel-1.6.2-43.oc8.x86_64.rpm                  508 kB/s |  29 kB     00:00
    (5/5): ncurses-devel-6.1-10.20180224.oc8.x86_64.rpm        993 kB/s | 527 kB     00:00-------------------------------------------------------------------------------------------
    总计                                                       1.6 MB/s | 879 kB     00:00
    运行事务检查
    事务检查成功。
    运行事务测试
    事务测试成功。
    运行事务
      准备中  :                                                                            1/1
      安装    : uuid-1.6.2-43.oc8.x86_64                                                   1/5
      运行脚本: uuid-1.6.2-43.oc8.x86_64                                                   1/5
      安装    : ncurses-c++-libs-6.1-10.20180224.oc8.x86_64                                2/5
      安装    : ncurses-devel-6.1-10.20180224.oc8.x86_64                                   3/5
      安装    : readline-devel-7.0-10.oc8.x86_64                                           4/5
      运行脚本: readline-devel-7.0-10.oc8.x86_64                                           4/5
      安装    : uuid-devel-1.6.2-43.oc8.x86_64                                             5/5
      运行脚本: uuid-devel-1.6.2-43.oc8.x86_64                                             5/5
      验证    : ncurses-c++-libs-6.1-10.20180224.oc8.x86_64                                1/5
      验证    : ncurses-devel-6.1-10.20180224.oc8.x86_64                                   2/5
      验证    : readline-devel-7.0-10.oc8.x86_64                                           3/5
      验证    : uuid-1.6.2-43.oc8.x86_64                                                   4/5
      验证    : uuid-devel-1.6.2-43.oc8.x86_64                                             5/5
    已安装:
      ncurses-c++-libs-6.1-10.20180224.oc8.x86_64   ncurses-devel-6.1-10.20180224.oc8.x86_64
      readline-devel-7.0-10.oc8.x86_64              uuid-1.6.2-43.oc8.x86_64
      uuid-devel-1.6.2-43.oc8.x86_64完毕！

### 创建用户

创建用户，设定 SSH 秘钥，并创建数据目录。

    useradd opentenbase
    echo 'opentenbase:opentenbase' | chpasswd
    su -l opentenbase -c "mkdir /home/opentenbase/.ssh -p"
    su -l opentenbase -c "ssh-keygen -t rsa -f /home/opentenbase/.ssh/id_rsa -N ''"
    mkdir /data
    chown opentenbase:opentenbase /data

### 获取源码

    git clone https://github.com/OpenTenBase/OpenTenBase

### 编译源码

*   配置源码
    
    ```
    cd /data/OpenTenBase
    ./configure --prefix=/data/opentenbase/install/opentenbase_bin_v2.6 --enable-user-switch --with-openssl  --with-ossp-uuid CFLAGS=-g
    ```

*   输出
    

```
configure: using compiler=gcc (GCC) 8.5.0 20210514 (Tencent 8.5.0-22)
configure: using CFLAGS=-D_PG_ORCL_ -DPGXC -DXCP -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -D_USER_SWITCH_ -g
configure: using CPPFLAGS= -D_GNU_SOURCE
configure: using LDFLAGS= -Wl,--as-needed
configure: creating./config.status
```


*   编译安装
    

    make -sjmake installcd contribmake -sjmake install

*   输出
    

    All of Postgres-XL successfully made. Ready to install.
    
    Postgres-XL installation complete.

### 配置用户环境变量

切换到 opentenbase 用户，配置用户环境变量。

```
su - opentenbase
vi ~/.bashrc
export OPENTENBASE_HOME=/data/opentenbase/install/opentenbase_bin_v2.6
export PATH=$OPENTENBASE_HOME/bin:$PATH
export LD_LIBRARY_PATH=$OPENTENBASE_HOME/lib:${LD_LIBRARY_PATH}
export LC_ALL=C
source ~/.bashrc
```

### 查看版本信息

```
[opentenbase@oc8 ~]$ pg_ctl -V
pg_ctl (PostgreSQL) 10.0 OpenTenBase V2
[opentenbase@oc8 ~]$ pgxc_ctl -V
/usr/bin/bash
Pgxc_ctl 10.0 OpenTenBase V2
```

到此，OpenTenBase 数据库已编译安装完成。

**使用 DNF 安装 OpenTenBase**
-------------------------

OpenTenBase 已支持 RPM 包部署，并且已将 RPM 推送到 OpenCloudOS 的 YUM 源中，只需开启 PowerTools/Plus 仓库，即可一键安装。

```
[root@oc8 ~]# dnf install OpenTenBase*
上次元数据过期检查：0:01:48 前，执行于 2024 年 09 月 03 日 星期二 00 时 44 分 59 秒。
依赖关系解决。
===========================================================================================
软件包                     架构           版本                     仓库              大小
===========================================================================================
安装:
OpenTenBase               x86_64         2.5.0-4.oc8             PowerTools         6.1 M
OpenTenBase-devel         x86_64         2.5.0-4.oc8             Plus              1.3 M
安装依赖关系:
uuid                      x86_64         1.6.2-43.oc8            AppStream          63 k
事务概要
===========================================================================================
安装  3 软件包
总下载：7.5 M
安装大小：31 M
确定吗？[y/N]：y
下载软件包：
(1/3): uuid-1.6.2-43.oc8.x86_64.rpm                     198 kB/s |  63 kB     00:00
(2/3): OpenTenBase-devel-2.5.0-4.oc8.x86_64.rpm          2.1 MB/s | 1.3 MB     00:00
(3/3): OpenTenBase-2.5.0-4.oc8.x86_64.rpm               4.5 MB/s | 6.1 MB     00:01
-------------------------------------------------------------------------------------------
总计                                                       5.5 MB/s | 7.5 MB     00:01
运行事务检查
事务检查成功。
运行事务测试
事务测试成功。
运行事务
  准备中  :                                                                                                                           1/1  
  安装    : uuid-1.6.2-43.oc8.x86_64                                                                                                         1/3  
  运行脚本: uuid-1.6.2-43.oc8.x86_64                                                                                                         1/3  
  安装    : OpenTenBase-2.5.0-4.oc8.x86_64                                                                                                    2/3  
  安装    : OpenTenBase-devel-2.5.0-4.oc8.x86_64                                                                                              3/3  
  运行脚本: OpenTenBase-devel-2.5.0-4.oc8.x86_64                                                                                              3/3  
  验证    : uuid-1.6.2-43.oc8.x86_64                                                                                                         1/3  
  验证    : OpenTenBase-2.5.0-4.oc8.x86_64                                                                                                    2/3  
  验证    : OpenTenBase-devel-2.5.0-4.oc8.x86_64                                                                                              3/3
已安装:
  OpenTenBase-2.5.0-4.oc8.x86_64          OpenTenBase-devel-2.5.0-4.oc8.x86_64  uuid-1.6.2-43.oc8.x86_64
完毕！
```

安装完毕，检查 OpenTenBase 版本。

```
[root@oc8 ~]# pg_ctl -V
pg_ctl (PostgreSQL) 10.0 OpenTenBase V2
[root@oc8 ~]# pgxc_ctl -V
/bin/bash
Pgxc_ctl 10.0 OpenTenBase V2
[root@oc8 ~]#
```

可见，第二种方式更加简单快捷。


<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

*   AtomGit
    
    https://atomgit.com/opentenbase/OpenTenBase
    
*   GitHub
    
    https://github.com/OpenTenBase/OpenTenBase