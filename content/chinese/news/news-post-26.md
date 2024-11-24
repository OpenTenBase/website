---
title: "社区贡献 | OpenTenBase配置冷热数据分离"
date: 2024-10-31T16:31:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-26.png
author: OpenTenBase
description: ""
---
<img src=../images/news-post-26-1.png class="img-fluid" /><br/>

## 贡献者介绍

陆子杰，统信软件工程师，负责统信UOS及开放原子如意玲珑（OpenAtom Linyaps）社区的软件生态建设工作，如意玲珑社区杰出开源贡献者。

## 前言

有听过如意玲珑——新时代Linux桌面应用分发和治理方案的朋友应该知道如意玲珑方案众多核心特性之一就是: "通过隔离技术，彻底解决系统与应用、应用与应用之间因升级引起的兼容性冲突问题。" 基于此特性以及如意玲珑技术架构介绍, 我们可以大体知道如意玲珑方案主要通过沙箱、容器方案来对进行应用与系统间的隔离, 这意味着应用容器中的大部分目录仅具备只读权限, 无法当做日常系统环境来使用。恰逢在 2024年11月16日下午 举办的如意玲珑开源社区首场高校技术沙龙中, 如意玲珑与OpenAtom OpenTenBase 开源社区开展了“跨界”分享, 在玲珑容器中试验性地进行了OpenTenBase 开源项目的源代码编译。而在活动结束后, 我继续针对如意玲珑 与 OpenTenBase 开展跨界探索, 随之竟发现了可玩性（实用性）更高的场景。

## 玲珑容器操作展示

为了补偿部分朋友无法见证 `如意玲珑` 与 `OpenTenBase` 开源社区“跨界”分享的现场演示, 我这里简单将操作过程向各位展示一遍

### 前期准备

1. 本次分享基于 `deepin 23` 发行版, 因此在进行以下任意步骤前均需要准备一个可以构建玲珑应用的 `deepin 23` 系统环境
2. 由于在构建过程中我们需要联网获取玲珑容器的运行库以及可能需要的第三方库, 因此我们需要保障全操作过程能够得到顺畅的网络连接
3. 按照以下模板简单编写一版玲珑构建工程配置文件 `linglong.yaml`, 以此来生成一个符合要求的容器

主要有以下需要关注的点:
\* 由于本次操作是直接进入容器进行操作, 因此 `build` 部分的构建规则可不详细写
\* 由于本次涉及编译操作, 为了能够极大程度包含所需的运行库, 我们加入 `runtime` 段, 具体编写规范参考[构建配置文件简介](https://linglong.dev/guide/ll-builder/manifests.html) [玲珑应用构建工程基础知识](https://github.com/OpenAtom-Linyaps/linyaps-meetup-open-doc/blob/main/docs/Tianjin-20241116/others/building-basic-notes.md)

```yaml
# SPDX-FileCopyrightText: 2023 UnionTech Software Technology Co., Ltd.
#
# SPDX-License-Identifier: LGPL-3.0-or-later

version: "3"

package:
  id: org.opentenbase.pgxc
  name: "OpenTenBase"
  version: 2.6.0.3
  kind: app
  description: |
    pgxc binary built from OpenTenBase

base: org.deepin.foundation/23.0.0
runtime: org.deepin.Runtime/23.0.1

command:
  - /opt/apps/org.opentenbase.pgxc/files/bin/pgxc_ctl

source:
  - kind: local
    name: "OpenTenBase"

build: |
  ##Extract res
  mkdir -p ${PREFIX}/bin/ ${PREFIX}/share/
```

4. 在完成准备 `linglong.yaml`编辑并将相关源代码解压到当前目录下后, 我们就可以开始编译了

### 项目编译演示

\*这里需要插播一个知识点: 在玲珑容器中, 与构建工程配置文件 `linglong.yaml` 同级的构建目录将被映射为 `/project` 目录
万事俱备, 我们就可以开始编译了

1. 为了方便操作, 我们在构建目录下同时开启两个shell窗口, 分别用于 `玲珑容器操作` 和 `普通操作`
2. 进入玲珑容器

```
szbt@szbt-linyaps23:/media/szbt/Data/ll-build/openTenbase$ ll-builder build --exec bash
```

路径发生类似以下变化时, 即意味着我们已经进入玲珑容器中了

```
szbt@szbt-linyaps23:/project$
```

3. 通过 `普通操作` 窗口解压openTenbase源码到构建目录中, 我这里单独解压到一个子目录中

```
szbt@szbt-linyaps23:/media/szbt/Data/ll-build/openTenbase$ tar -xvf OpenTenBase-v2.6.0-src.tar.zst -C src/
```

4. 源码解压后, 我们在编译任意源代码前应该正确选择使用何种编译系统/工具. 不过贴心的是, `OpenTenBase` 开源项目已经为我们准备好了编译安装的入门文档, 我们可以先阅读 [OpenTenBase源码编译安装](https://docs.opentenbase.org/guide/01-quickstart/#opentenbase) 来准备编译材料
5. 由于玲珑容器中不存在 `apt` 联网安装依赖包的说法, 因此我们先跳过 `依赖安装` 步骤. 不过需要注意的是, 其中玲珑容器当前明确具备的库/编译器有: `gcc12.3, zlib, openssl3`, 其他暂未表明的库有可能不存在容器中而需要我们手动编译(记重点 , 后面要考)

综上, 我们有可能会缺失文档中提示的 `libreadline-dev libossp-uuid-dev`

6. 事不宜迟, 我们通过 `玲珑容器操作` 窗口进入源码目录, 为了尽量避免对源目录的干扰, 我这里新建一个 `build` 目录用于编译. 进入 `build` 目录后我们输入[OpenTenBase源码编译安装](https://docs.opentenbase.org/guide/01-quickstart/#opentenbase)中的 `configure` 指令来配置构建工程. 根据[玲珑应用构建工程基础知识](https://github.com/OpenAtom-Linyaps/linyaps-meetup-open-doc/blob/main/docs/Tianjin-20241116/others/building-basic-notes.md), 我们将 `--prefix` 赋予 `$PREFIX` 的值, 最终我在本地执行了以下操作:

```
../configure --prefix=$PREFIX  --enable-user-switch --with-openssl\
 --with-ossp-uuid CFLAGS=-g\
 LDFLAGS=-L$PREFIX/lib/ CFLAGS=-I$PREFIX/include\
 --with-libs=$PREFIX/lib/ --with-includes=$PREFIX/include\
 CPPFLAGS=-L$PREFIX/include
```

7. 可以从图中看到, 这里出现了一个错误导致无法完成配置. 我们看到是 `libreadline` 库无法找到, 考虑到[玲珑应用构建工程基础知识](https://github.com/OpenAtom-Linyaps/linyaps-meetup-open-doc/blob/main/docs/Tianjin-20241116/others/building-basic-notes.md)中提到的目录特性, 我单独检查了对应的 `include` 目录均无法找到与此库相关的资源

```
configure: error: readline library not found
If you have readline already installed, see config.log for details on the
failure.  It is possible the compiler isn't looking in the proper directory.
Use --without-readline to disable readline support.
```

```
szbt@szbt-linyaps23:/project$ ls /usr/include/ |grep readline
szbt@szbt-linyaps23:/project$ ls /runtime/include/ |grep readline
szbt@szbt-linyaps23:/project$ 
```

结合此报错, 基本可以判断为该库缺失. 根据提示, 我们可以使用 `--without-readline` 参数来禁用此库以跳过错误

8. 禁用后我们重新 `configure`, 出现了 `uuid` 相关库丢失的情况, 由于不能完全确认库缺失情况, 我执行了 `./configure --help` 来查看当前工程支持哪些参数, 很快我就找到了和 `uuid` 的相关参数, 并更新成以下的参数:

```
../configure --prefix=$PREFIX  --enable-user-switch --with-openssl\
 --with-uuid=e2fs CFLAGS=-g\
 LDFLAGS=-L$PREFIX/lib/ CFLAGS=-I$PREFIX/include\
 --with-libs=$PREFIX/lib/ --with-includes=$PREFIX/include\
 CPPFLAGS=-L$PREFIX/include --without-readline
```

9. 重新执行后, 成功完成工程配置了, 这一步直接执行 `make` 操作即可
10. 在该过程中, 依次出现了 `bison` `flex` 程序缺失的错误, 我们返回`普通操作` 窗口将对应的源码下载到当前构建目录中, 进入`玲珑容器操作` 窗口重新编译

<img src=../images/news-post-26-2.png class="img-fluid" /><br/>
<img src=../images/news-post-26-3.png class="img-fluid" /><br/>
11. 重新编译 `OpenTenBase项目`, 和 `readline` 相关报错不存在了, 重新编译后, 第一次 `make` 可以成功完成了.
12. 随之我们立即根据文档开始第二次 `make`. 但出现了相对路径无法寻找的问题

```
make: *** 没有规则可制作目标“../src/Makefile.global”。 停止。
```

13. 定睛一看, 这是因为我在编译上一级源码时使用了独立的编译目录, 破坏了原有的`Makefile`设定, 因此我重新生成玲珑容器, 更换编译目录为 `configure` 同级目录, 按 `1==>11` 顺序完全编译一次即可
14. 但是这次 `make` 中出现了 `readline` 头文件丢失的情况, 这意味着即便我们上一步禁用了此库, 想要完成所有二进制程序编译还是无法离开此库.
    <img src=../images/news-post-26-4.png class="img-fluid" /><br/>

我们参考 `1==>10` 的步骤重新编译 `readline` 及其依赖运行库 `libncurses` 后, 也成功完成二次编译了

```
./configure --prefix=$PREFIX  --enable-user-switch --with-openssl\
 --with-uuid=e2fs CFLAGS=-g\
 LDFLAGS=-L$PREFIX/lib/ CFLAGS=-I$PREFIX/include\
 --with-libs=$PREFIX/lib/ --with-includes=$PREFIX/include\
 CPPFLAGS=-L$PREFIX/include --with-readline
```

### 编译结果测试

在完成 `make install` 后, 由于--prefix被修改为了$PREFIX, 该变量也在容器默认PATH中, 因此我查看是否可以正常运行此二进制:

```
szbt@szbt-linyaps23:/project/src/OpenTenBase-v2.6.0/contrib$ pgxc_ctl --help   
/bin/bash
pgxc_ctl [option ...] [command]
option:
   -c or --configuration conf_file: Specify configruration file.
   -v or --verbose: Specify verbose output.
   -V or --version: Print version and exit.
   -l or --logdir log_directory: specifies what directory to write logs.
   -L or --logfile log_file: Specifies log file.
   --home home_direcotry: Specifies pgxc_ctl work director.
   -i or --infile input_file: Specifies inptut file.
   -o or --outfile output_file: Specifies output file.
   -h or --help: Prints this message and exits.
For more deatils, refer to pgxc_ctl reference manual included in
postgres-xc reference manual.
```

至此, 足以证明 `OpenTenBase项目` 可以在 `如意玲珑` 应用容器中成功编译并运行!

## 二进制程序封装&二次分发

尽管如此, 我还是不满足. 我心里又萌生了一个玩法: 有没有方案可以将玲珑容器中编译的二进制文件导出, 并打包成 `deb` 等传统包格式提供给暂不支持玲珑环境的发行版体验?
很快, 我发现玲珑容器中是包含了 `tar` 程序的, 结合 `/project`路径可以映射为宿主机的构建目录, 我将 `$PREFIX` 目录封装为归档文件:

```
szbt@szbt-linyaps23:/project$ tar -jcvf OpenTenBase-v2.6.0-x86_64-binary.tar.bz2 $PREFIX/

OpenTenBase-v2.6.0-x86_64-binary.tar.bz2
```

返回 `普通操作` 窗口, 发现我们通过宿主机也可以对此归档文件进行读写操作
随后, 我将归档文件内的二进制文件封装为了deb安装包, 路径将 `$PREFIX` 修改为传统的 `/usr`

## 其他基线相近的发行版安装使用由玲珑容器构建的二进制程序

在得到deb安装包后, 我在不同主流发行版上尝试体验, 来确认通过玲珑容器构建的二进制程序是否也存在通用性

### deepin 23

<img src=../images/news-post-26-5.png class="img-fluid" /><br/>

### openKylin 2.0

<img src=../images/news-post-26-6.png class="img-fluid" /><br/>

### Ubuntu 2404

<img src=../images/news-post-26-7.png class="img-fluid" /><br/>

## 总结

显而易见, 不仅仅是通过玲珑容器封装的应用支持在不同发行版上使用（需要支持玲珑环境）。通过玲珑容器编译的二进制程序也支持在不同满足运行库要求的发行版上使用（暂不支持玲珑环境）。

<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

* AtomGit

  https://atomgit.com/opentenbase/OpenTenBase
* GitHub

  https://github.com/OpenTenBase/OpenTenBase
