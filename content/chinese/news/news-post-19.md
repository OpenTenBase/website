---
title: "OpenTenBase实战：如何通过扩展机制为数据库系统增加新功能"
date: 2024-09-18T16:51:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-19.png
author: OpenTenBase
description: ""
---

**1\. 简介**

PostgreSQL是一款非常流行的开源数据库，已多次成为全球知名数据库流行度排行榜DB-Engines评选出的年度数据库。在PostgreSQL中，扩展（extension）是一种可插拔的模块，允许用户将额外的功能或数据类型添加到数据库中。

OpenTenBase是在PostgreSQL基础上研发的企业级分布式HTAP开源分布式数据库，也继承了PostgreSQL强大的插件系统，支持开发者为其添加新功能。OpenTenBase 2.6开始支持的空间数据库插件PostGIS就是知名的扩展之一，它为数据库增加了支持地理信息系统的能力。

本文将通过代码实际演示一下如何为OpenTenBase增加一个扩展。

**2\. 扩展结构**

**2.1 控制文件**

控制文件是一个定义了扩展元数据的文件，文件后缀为.control，通常由comment，default\_version，module\_pathname，relocatable，schema组成。

comment: 是对扩展的简短描述,帮助用户了解扩展的功能。

default\_version：这指定了在创建扩展时如果没有指定版本将使用的默认版本。

module\_pathname：这是扩展的共享库模块的路径。在数据库中使用扩展时需要加载该路径才能使用扩展。

relocatable：值为布尔类型，表示扩展是否可重定位。可重定位的扩展在创建后可以移动到不同的模式（schema）。

schema：安装扩展对象的模式。

下面是adminpack扩展的控制文件示例

`# time_monitor extension`  
`comment = 'find out the sql that is exceeding expectations' #描述扩展的功能`  
`default_version = '1.0' #扩展的默认版本号`   
`module_pathname = '$libdir/time_monitor' #指定扩展模块文件的路径`  
`relocatable = false #扩展是否可以被重新定位。如果设置为 true，则表示扩展可以在安装后移动到其他位置，而不会影响其功能。`  

**2.2 SQL脚本文件**

SQL脚本文件是用于定义和创建扩展所提供的数据库对象，比如函数、数据类型、操作符、索引方法、表这些。SQL脚本文件可以算做是扩展的比较的核心部分，因为它包含了实现扩展功能的所有必要SQL命令。另外，在一个扩展中，可以有一个SQL脚本文件，也可以有多个脚本文件。当然，如果你不需要对数据库进行操作，你也可以直接舍弃SQL脚本文件。

以下是一个脚本函数示例，函数的实现在C语言文件中。 

`\echo Use "CREATE EXTENSION time_monitor" to load this file.`  
`\quit`  

**2.3 C语言文件**

C语言文件可以用来编写自定义的函数和数据类型。这些函数和数据类型可能会直接操作数据库的内部结构，或者执行一些SQL无法高效完成的复杂计算，也可以用来实现自定义的操作符和索引方法。这些操作符和索引方法可以提供对特定数据类型的高效处理，或者支持新的查询优化策略。OpenTenBase数据库中还有一些钩子函数，在插件中可以注册这些回调函数来为系统的各个处理环节插入逻辑代码用来实现想要的功能。这样就可以在不修改内核代码的情况下，通过插件为内核增加功能。

需要注意的是，之前在SQL脚本中定义的函数要在C语言文件中实现，比如上面的pg\_file\_write函数

`#include "postgres.h"`  
`#include "fmgr.h"`  
`#include "executor/executor.h"`  
`#include <time.h>`  
`#include <stdio.h>`  
  
`PG_MODULE_MAGIC;`  
  
`static ExecutorStart_hook_type prev_ExecutorStart = NULL;`  
`static ExecutorEnd_hook_type prev_ExecutorEnd = NULL;`  
  
`static long long start_time = 0;`  
`static long long end_time = 0;`  
`static long long total_time = 0;`  
  
`void _PG_fini(void);`  
`void _PG_init(void);`  
`static void check_ExecutorStart(QueryDesc *queryDesc, int eflags);`  
`static void check_ExecutorEnd(QueryDesc *queryDesc);`  
  
`void _PG_init(void)`  
`{`  
 `prev_ExecutorStart = ExecutorStart_hook;`  
 `ExecutorStart_hook = check_ExecutorStart;`  
 `prev_ExecutorEnd = ExecutorEnd_hook;`  
 `ExecutorEnd_hook = check_ExecutorEnd;`  
`}`  
  
`static void check_ExecutorStart(QueryDesc *queryDesc, int eflags)`  
`{`  
 `start_time = time(NULL);`  
  
 `if (prev_ExecutorStart)`  
 `prev_ExecutorStart(queryDesc, eflags);`  
 `else`  
 `standard_ExecutorStart(queryDesc, eflags);`  
`}`  
  
`static void check_ExecutorEnd(QueryDesc *queryDesc)`  
`{`  
 `end_time = time(NULL);`  
 `total_time =  end_time - start_time;`  
 `if( total_time >= 0){`  
 `FILE *fptr;`  
 `fptr = fopen("log.txt", "a");`  
 `time_t timep;`  
 `time (&timep);`  
 `fprintf(fptr, "The current time：%s,execution time: %lld, Sql: %s\n", asctime( gmtime(&timep)), total_time, queryDesc->sourceText);`  
 `fclose(fptr);`  
 `}`  
  
 `if (prev_ExecutorEnd)`  
 `prev_ExecutorEnd(queryDesc);`  
 `else`  
 `standard_ExecutorEnd(queryDesc);`  
`}`  
  
`void _PG_fini(void)`  
`{`  
 `ExecutorStart_hook = prev_ExecutorStart;`  
 `ExecutorEnd_hook = prev_ExecutorEnd;`  
`}`  

**2.4 Makefile文件**

一个扩展可以没有C语言文件，可以没有SQL脚本，可以没有control，但是不能没有Makeflie。Makeflie文件是一个构建脚本，它指导着编译过程，告诉构建系统如何编译和安装扩展。通常包含了一系列的规则，这些规则定义了目标（例如，扩展的共享库文件）以及如何从源代码生成这些目标。

扩展一般的Makefile文件都是一个模版

`# contrib/time_monitor/Makefile`  
  
`MODULE_big = time_monitor`  
`OBJS = time_monitor.o $(WIN32RES)`  
  
`EXTENSION = time_monitor`  
`DATA = time_monitor--1.0.sql`  
`PGFILEDESC = "time_monitor - find out the sql that is exceeding expectations"`  
  
`REGRESS = check check_btree`  
  
`PG_CONFIG = pg_config`  
`PGXS := $(shell $(PG_CONFIG) --pgxs)`  
`include $(PGXS)`   

**3.扩展安装和使用**

将扩展文件夹放到contrib文件夹下

`make -sj && make install`  

生成\*.o和\*.so 文件

安装命令

`CREATE EXTENSION  time_monitor;`  

加载扩展

`load 'libdir/name' # name:扩展名`  

查看扩展是否被安装

`select * from pg_extension;`  

<img src=../images/news-post-19-1.png class="img-fluid" /><br/>

卸载扩展

`drop extension time_monitor;`  

<img src=../images/news-post-19-2.png class="img-fluid" /><br/>

<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

*   AtomGit
    
    https://atomgit.com/opentenbase/OpenTenBase
    
*   GitHub
    
    https://github.com/OpenTenBase/OpenTenBase