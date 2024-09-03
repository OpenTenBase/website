---
title: "社区贡献 | OpenTenBase集群部署初探"
date: 2024-08-30T12:48:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-14.png
author: OpenTenBase
description: ""
---


开始踩坑

官方源码地址：```
git clone https://github.com/OpenTenBase/OpenTenBase```  

在这篇文章中，以Centos 8为例展示了如何进行部署。如果你需要了解基本的安装操作步骤，可以参考这个链接：https://docs.opentenbase.org/guide/01-quickstart

本文就不再一一演示这些基本步骤了，而是想分享一些官方文档中没有提及的各种奇葩问题的解决方法。

uuid-devel匹配不到
--------------

上来第一步就发现了问题，当执行环境依赖安装时
```yum -y install gcc make readline-devel zlib-devel openssl-devel uuid-devel bison flex git```

在Centos 8系统上，可能会遇到一个错误提示：找不到 uuid-devel 软件包。这是因为在Centos 8的默认软件仓库中找不到 uuid-devel 软件包，尽管 uuid-devel 实际上是一个必需的依赖项。此外，安装类似uuid依赖包也无法解决问题，否则在执行configure命令时可能会出现错误提示：`configure: error: library 'ossp-uuid' or 'uuid' is required for OSSP UUID`  

幸运的是，CentOS的“PowerTools”软件库中包含了 uuid-devel 软件包，但默认情况下未启用。要启用该软件库，可以使用以下命令`dnf config-manager --set-enabled powertools`  
,如果没有dnf命令，则执行一下：`yum install dnf-plugins-core`  

configure: error: readline library not found
--------------------------------------------

在执行configure命令时报错：configure: error: readline library not found

如果遇到这个问题，可以尝试执行以下命令来安装必要的依赖包：`yum -y install gcc make readline-devel`  
即可

确保所有的依赖环境都已安装完毕后，再执行make -sj命令。在执行这一步之前，请确保剩余可用内存大于等于4G，以避免内存溢出问题。尽管官方文档建议最低内存为4G，但建议将内存扩大至8G，以确保后续执行init all命令时不会遇到各种奇怪的问题。切记，不要将内存设置得过低，否则可能会导致后续命令的异常行为。

环境及ssh
------

执行`vim ~/.bashrc`  
编辑系统环境变量后记得source ~/.bashrc，要不然无法找到命令pgxc\_ctl

在集群部署过程中，只有一台服务器需要进行编译操作，其他服务器只需进行环境变量配置、用户及目录设置以及SSH连接的配置。这样设计的原因是因为在执行deploy all命令时，已经编译好的安装包会被发送到其他机器上。

为了实现集群节点机器之间的SSH无密码登录，首先需要在各个节点机器上配置好SSH密钥认证。这样一来，在部署和初始化过程中，可以通过SSH连接到每个节点的机器而无需输入密码。在这个过程中，需要确保已经打通了第二台及其IP的SSH连接，并且也打通了自己机器的SSH连接。

`ssh-copy-id -i ~/.ssh/id_rsa.pub destination-user@destination-server`  

启动和节点排查
-------

在进行集群部署时，接下来的步骤是使用pgxc\_ctl进行部署。如果对pgxc\_ctl的命令不熟悉，可以通过使用help命令来查看帮助文档。在本次部署的机器上，运行monitor all命令时，只能显示一个信息然后程序强制退出，这表明肯定有节点启动失败了。因此，建议单独使用monitor命令来查看各个节点的状态，以便更清楚地了解每个节点的运行情况。

<img src=../images/news-post-13-1.png class="img-fluid" /><br/>

如果某一个一直无法正常启动，比如显示`gtm_ctl: another server might be running; trying to start server anyway`  
，那么可能会是你没有正常关闭，通常需要你手动去删除对应的pid文件，

本次以gtm为例，如果不知道的pid文件位置在哪里，那么可以使用`find / -name '*gtm*.pid'`  
，找到后删除对应的文件即可。然后再次启动start all。

如果还是无法启动，那么则可以去看下对应日志，还以gtm为例。`cd /data/opentenbase/data/gtm/slave/gtm_log`  
进入对应日志目录，然后查看日志。这里显示的最后是

<img src=../images/news-post-13-2.png class="img-fluid" /><br/>

建议考虑进行扩容操作。显然这里资源不足。如果你的内存已经达到了8GB，那么可以考虑进一步扩展CPU资源至2核心。本次部署的系统只有1核心的CPU，显然已经不够用了，扩容后系统性能应该会恢复正常。

<img src=../images/news-post-13-3.png class="img-fluid" /><br/>

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

*   AtomGit
    
    https://atomgit.com/opentenbase/OpenTenBase
    
*   GitHub
    
    https://github.com/OpenTenBase/OpenTenBase