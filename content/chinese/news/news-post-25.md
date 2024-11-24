---
title: "社区贡献 | OpenTenBase内核中的基础数据类型简介：列表"
date: 2024-11-21T16:31:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-25.png
author: OpenTenBase
description: ""
---
<img src=../images/news-post-25-1.png class="img-fluid" /><br/>

> 贡献者介绍

> 文一，OpenTenBase校园大使，目前本科就读于应急管理大学，校内中国 PostgreSQL 分会实训基地负责人，开放原子开源基金会AtomGit开源协作平台“内核一周一审”专区负责人，多个开源项目内核代码贡献者，青学会 MOP 社区会员，《数据库红皮书》（线上读物）译者，曾应邀在中国科学院、北京航空航天大学等地做 PostgreSQL 内核相关的技术分享工作。

OpenTenBase 依托许多自主实现的数据结构支撑起复杂的功能，而在本篇文章中，我们将向读者介绍列表这一基本数据结构的实现方式，并通过介绍 OpenTenBase 对于 hba 文件（OpenTenBase 中的一种配置文件）的支持方式，阐述其应用办法。
列表数据类型是什么？如何实现？

列表（List），代表一组按照某种规则、方式排列起来的元素，如图所示：

<img src=../images/news-post-25-1.png class="img-fluid" /><br/>

实际上，我们在生活中经常同列表打交道，比如，在我们的微信中的好友列表，又或者，排列于AtomGit开源协作平台上面的“运营专区列表”，都是这一结构的具体体现。

有了这个基本的了解后，又要看到，想要在具体生活乃至于抽象模型之外实现列表，我们就必须回答好下面的三个问题：

* 第一，我们所储存的元素应当按照什么样的方式来组织？比如，我们往往喜欢使用 int 这一数据类型，在单独的列表数据元素中存储整数。
* 第二，列表整体应当记录怎么样的信息？比如，列表本身是不是需要关注整体目前存储的元素个数，用于便利我们记录当前的内存容量？
* 第三，我们的软件有没有什么“列表数据结构”之外的特殊需求？

而在 OpenTenBase 的 pg_list.h 中，三个问题的答案，就通过 List 与 ListCell 的源代码作出了回答。

# OpenTenBase 对于列表类型的定义

在 pg_list.h 中，List 用于代指列表整体，而 ListCell 用于代表数据元素，两者的代码如下：

```
/* OpenTenBase 2.6.0 */

typedef struct List
{
NodeTag        type;
/*
用于标识节点的类型，这需要联系火山模型来做理解，
但是它超出了本文的范围，所以我们此处不做展开
*/
/* T_List, T_IntList, or T_OidList */
/*
分别代表通用列表、整数列表、Oid 列表，
其中 Oid 是 OpenTenBase 用于统一管理资源对象的编号
*/
int            length;
/* 列表的长度 */
/* 列表的头节点与尾节点 */
ListCell   *head;
ListCell   *tail;
} List;

struct ListCell
{
union
{
void      *ptr_value; // 在作为 通用元素 时使用
int       int_value;  // 在作为 整数元素 时使用
Oid       oid_value;  // 在作为 oid 元素 时使用
}data;
/* 指向下一个节点 */
ListCell   *next;
};
```

这里我们可以很明显地看出：

第一，其实只需要借助一个 ListCell 元素就已经可以定义出一个完整的列表（因为我们只需要不断遍历 next 就可以翻出整个列表的内容，前提是这个 ListCell 是头节点），但是 OpenTenBase 依旧设立了一个具有全局意义的 List 用于管理列表本身。

第二，因为 OpenTenBase 本身采用了 “火山模型”（在这种模型里面，数据库执行引擎将各种操作乃至于结构划分为一个个 “Node”，并根据某种优化路径，将他们组织为一棵执行树），所以 List 还包含了一个 NodeTag 元素。

第三，头节点和尾节点元素的存在，可以让我们非常方便地执行列表的遍历以及元素的增加等（某种意义上，这就是空间换时间）。

第四，联合体可以方便地用同一个元素，代指不同的数据元素类型（从某种意义上面看，C语言中的数据类型，就是一种对于内存的组织方式，除此之外，各种数据类型本质上是通用的）。

# OpenTenBase List 的具体实现：以元素的插入函数实现为例

理解 OpenTenBase 列表的基本组织方式以后，着手分析其行为，可以帮助我们更好地理解它的实现，而列表的插入，无疑就是一个好的切口：

<img src=../images/news-post-25-3.png class="img-fluid" /><br/>

```
/* 在 list.c 中提取 */
List *
lcons(void *datum, List *list)
{
/* 检测列表的类型，本质就是判断 Node 的标签是否为 List */
/* 在生产版本的 OpenTenBase 中，这行代码没有意义 */
Assert(IsPointerList(list));

/*
如果列表是不存在的，创建一个新的列表
否则，分配一个新的头节点元素，注意 OpenTenBase 的 List 是自头部插入的
*/
if (list == NIL)
list = new_list(T_List);
else
new_head_cell(list);

/* 在分配完节点空间之后，配置数据域，即将整数填充进去 */
lfirst(list->head) = datum;
/* 检查插入的数据是否有效 */
/* 同样的，生产版本中，这行代码没有意义 */
check_list_invariants(list);
return list;
}
```

# OpenTenBase 列表的应用：以对 hba 文件的支持为例

hba 文件在 OpenTenBase 中承担着客户端认证的职责，而在 hba.c 中，List 便被应用于存储解析后的 hba 配置文件代码，参考下面的内容：

```
/*
pre-parsed content of HBA config file: list of HbaLine structs.
parsed_hba_context is the memory context where it lives.
*/
/* HbaLine 结构是自定义的类型，因此很明显此处是通用列表 */
static List *parsed_hba_lines = NIL;
```

<img src=../images/news-post-25-4.png class="img-fluid" /><br/>

从HbaLine 结构的部分内容可以看出，这是一个非常复杂的结构。



<img src=../images/news-post-25-5.png class="img-fluid" /><br/>


具体对应什么呢？对应的就是用户填入的种种客户端认证信息，可以发现，形式多样，而每种形式对应的数据不一定一样，这就是为什么这个结构这么复杂的原因。

<img src=../images/news-post-25-6.png class="img-fluid" /><br/>

在 hba 的具体实现中，OpenTenBase 会逐个遍历列表中的元素，用以解析客户端的配置信息，以进行用户验证，而解析顺序的先后，往往就会直接影响软件的行为。比如，从前往后解析（以匹配的第一个元素为准）和从后往前解析，它的执行结果，因顺序的不同，很可能就不一样。

# 写在最后

管中窥豹，我们简单分析了 OpenTenBase 对于数据列表类型的定义、实现乃至于应用，进而增强了对于 OpenTenBase 内核原理的理解。
感谢开放原子开源基金会张凯老师与符芬菊老师的理解、信任与支持，我的本科生导师，袁国铭博士，中国 PostgreSQL 分会的魏波老师与王其达老师，IvorySQL 社区的任娇老师、牛世继老师，KiwiDB 的于雨老师、刘月财老师，以及其它 OpenTenBase 社区的朋友，我们将继续努力，为建设一个更为开放繁荣的基础软件生态作出自己的贡献。


谢谢你们！

<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

* AtomGit

  https://atomgit.com/opentenbase/OpenTenBase
* GitHub

  https://github.com/OpenTenBase/OpenTenBase
