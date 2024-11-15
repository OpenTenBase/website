---
title: "社区贡献 | OpenTenBase配置冷热数据分离"
date: 2024-10-31T16:31:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-20.jpg
author: OpenTenBase
description: ""
---

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241031_1fb582c0-9750-11ef-afed-fa163eb4f6be.png)

OpenTenBase 是一个分布式数据库系统，支持多种高级功能，包括冷热数据分离。冷热数据分离是一种常见的数据管理策略，旨在提高查询性能并优化存储成本。热数据是指频繁访问的数据，而冷数据是指不经常访问的数据。通过将热数据和冷数据分开存储，可以提高系统的整体性能和效率。

**一、环境配置**
================

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241031_1fd373e8-9750-11ef-afed-fa163eb4f6be.png)
===========================================================================================================

**二、冷热数据分离的配置步骤**

以下是在 OpenTenBase 中配置冷热数据分离的基本步骤：

2.1 参数配置
------------

1）确认冷热数据分离界线

示例：以年度为单位， 记录时间小于2022-01-01为冷数据，大于2022-01-01为热数据

postgres=# show manual\_hot\_date ;
manual\_hot\_date
\-----------------
2022-01-01
(1 row

postgres=# show cold\_hot\_sepration\_mode;
cold\_hot\_sepration\_mode
\-------------------------
year
(1 row)

postgres=# show enable\_cold\_seperation;
enable\_cold\_seperation
\------------------------
on(1 row

注意：以上参数要求所有DN上均需要修改，重启DN生效。

2.2 创建冷数据组

1）查看当前组和DN节点

postgres\=\# select \* from pgxc\_group;
group\_name   | default\_group | group\_members
\---------------+---------------+---------------
default\_group |             1 | 16385 16386
(2 rows)

2）查看当前节点信息

postgres=\# select \* from pgxc\_node where node\_type='D';
node\_name | node\_type | node\_port |   node\_host   | nodeis\_primary | nodeis\_preferred |   node\_id   |  node\_cluster\_name
\-----------+-----------+-----------+---------------+----------------+------------------+-------------+---------------------
dn001     | D         |     40004 | 192.168.2.136 | t              | t                |  2142761564 | opentenbase\_cluster
dn002     | D         |     40004 | 192.168.2.137 | f              | f                |   -17499968 | opentenbase\_cluster
dn003     | D         |     40004 | 192.168.2.138 | f              | f                | -1956435056 | opentenbase\_cluster
(3 rows)

3）创建冷数据组

从上面查询可以确认，dn003未被使用

\--CN上执行，不要到DN上
\[opentenbase@db1 ~\]$  psql -h 192.168.2.136 -p 30004 -d postgres -U opentenbase
create node group cold\_group with(dn003);
create extension sharding group to group cold\_group;
clean sharding;

\--连接到DN3为冷组
\[opentenbase@db1 ~\]$  psql -h 192.168.2.138 -p 40004 -d postgres -U opentenbase
postgres=\# select pg\_set\_node\_cold\_access();
pg\_set\_node\_cold\_access
\-------------------------
success

**三、创建表**

\--CN上操作
\[opentenbase@db1 ~\]$  psql -h 192.168.2.136 -p 30004 -d postgres -U opentenbase

DROP TABLE  cold\_move\_test;

CREATE TABLE cold\_move\_test(
id int not null,
identifynumber text NOT NULL,
inserttimeforhis timestamp without time zone NOT NULL,
primary key(id,identifynumber,inserttimeforhis))
partition by range (inserttimeforhis)
begin (timestamp without time zone '2020-01-01')
step (interval '1 year')
partitions (24)
DISTRIBUTE BY SHARD (identifynumber,inserttimeforhis) to GROUP default\_group cold\_group;

##查看对应的分区
pg中的分区表的命名规则如下
postgres=\# select relname from pg\_class where relname like '%cold\_move%part%';
relname
\-----------------------------
cold\_move\_test\_part\_0
cold\_move\_test\_part\_1
cold\_move\_test\_part\_2
cold\_move\_test\_part\_3
cold\_move\_test\_part\_4
cold\_move\_test\_part\_5
cold\_move\_test\_part\_6
cold\_move\_test\_part\_7
cold\_move\_test\_part\_8
cold\_move\_test\_part\_9
cold\_move\_test\_part\_10
cold\_move\_test\_part\_11
cold\_move\_test\_part\_12
cold\_move\_test\_part\_13
cold\_move\_test\_part\_14
cold\_move\_test\_part\_15
cold\_move\_test\_part\_16
cold\_move\_test\_part\_17
cold\_move\_test\_part\_18
cold\_move\_test\_part\_19
cold\_move\_test\_part\_20
cold\_move\_test\_part\_21
cold\_move\_test\_part\_22
cold\_move\_test\_part\_23

**四、插入数据**

1）--插入100条2020年的数据
for i in \`seq 100\`
do
psql -h 192.168.2.136 -p 30004 -d postgres -U opentenbase -c "insert into cold\_move\_test values(${i},'testtest${i}','2020-09-03 16:21:34.201133');" >/dev/null
done

2）--插入100条2021年数据
for i in \`seq 201 300\`
do
psql -h 192.168.2.136 -p 30004 -d postgres -U opentenbase -c "insert into cold\_move\_test values(${i},'testtest${i}','2021-12-03 16:21:34.201133');" >/dev/null
done

2）--插入100条2022年数据
for i in \`seq 301 400\`
do
psql -h 192.168.2.136 -p 30004 -d postgres -U opentenbase -c "insert into cold\_move\_test values(${i},'testtest${i}','2022-12-03 16:21:34.201133');" >/dev/null
done

**五、验证**

5.1 验证热数据访问的节点

当前从2022-01-01开始为热数据

postgres=\# EXPLAIN SELECT COUNT(1) FROM cold\_move\_test where inserttimeforhis>'2022-01-01';
QUERY PLAN

\-------------------------------------------------------------------------------------------------------------------
Aggregate  (cost=103.44..103.45 rows=1 width=8)
->  Remote Subquery Scan on all (dn001,dn002)  (cost=100.00..103.36 rows=30 width=0)
->  Append  (cost=0.00..0.00 rows=0 width=0)
->  Seq Scan on cold\_move\_test (partition sequence: 2, name: cold\_move\_test\_part\_2)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold\_move\_test (partition sequence: 3, name: cold\_move\_test\_part\_3)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold\_move\_test (partition sequence: 4, name: cold\_move\_test\_part\_4)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold\_move\_test (partition sequence: 5, name: cold\_move\_test\_part\_5)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold\_move\_test (partition sequence: 6, name: cold\_move\_test\_part\_6)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold\_move\_test (partition sequence: 7, name: cold\_move\_test\_part\_7)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold\_move\_test (partition sequence: 8, name: cold\_move\_test\_part\_8)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold\_move\_test (partition sequence: 9, name: cold\_move\_test\_part\_9)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold\_move\_test (partition sequence: 10, name: cold\_move\_test\_part\_10)  (cost=0.00..0.15 rows=1

可以看到执行计划访问的dn001和dn002是热数据所在的default\_group组。

5.2 验证冷数据访问的节点

当前inserttimeforhis小于2022-01-01开始为冷数据

postgres=# EXPLAIN SELECT COUNT(1) FROM cold\_move\_test where inserttimeforhis<'2022-01-01';
QUERY PLAN

\------------------------------------------------------------------------------------------------------------------------------
\---------
Remote Fast Query Execution  (cost=0.00..0.00 rows=0 width=0)
Node/s: dn003
\->  Aggregate  (cost=5.00..5.01 rows=1 width=8)
\->  Append  (cost=0.00..0.00 rows=0 width=0)
\->  Seq Scan on cold\_move\_test (partition sequence: 0, name: cold\_move\_test\_part\_0)  (cost=0.00..2.25 rows=100
width=0)
Filter: (inserttimeforhis < '2022-01-01 00:00:00'::timestamp without time zone)
\->  Seq Scan on cold\_move\_test (partition sequence: 1, name: cold\_move\_test\_part\_1)  (cost=0.00..2.25 rows=100
width=0)
Filter: (inserttimeforhis < '2022-01-01 00:00:00'::timestamp without time zone)
(8 rows)

可以看到执行计划访问的dn003是冷数据所在的cold\_group组。

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241031_1fe7334c-9750-11ef-afed-fa163eb4f6be.png)

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

![](https://oss-emcsprod-public.modb.pro/image/auto/modb_20241031_1ff17fd2-9750-11ef-afed-fa163eb4f6be.png)

**官网：**https://www.opentenbase.org/

**贡献代码**

* AtomGit

  https://atomgit.com/opentenbase
* GitHub

  https://github.com/OpenTenBase

<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

* AtomGit

  https://atomgit.com/opentenbase/OpenTenBase
* GitHub

  https://github.com/OpenTenBase/OpenTenBase
