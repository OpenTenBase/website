---
title: "社区贡献 | OpenTenBase配置冷热数据分离"
date: 2024-10-31T16:31:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-23.png
author: OpenTenBase
description: ""
---
<img src=../images/news-post-23-1.png class="img-fluid" /><br/>

OpenTenBase 是一个分布式数据库系统，支持多种高级功能，包括冷热数据分离。冷热数据分离是一种常见的数据管理策略，旨在提高查询性能并优化存储成本。热数据是指频繁访问的数据，而冷数据是指不经常访问的数据。通过将热数据和冷数据分开存储，可以提高系统的整体性能和效率。

**一、环境配置**

<img src=../images/news-post-23-2.png class="img-fluid" /><br/>

**二、冷热数据分离的配置步骤**

以下是在 OpenTenBase 中配置冷热数据分离的基本步骤：

2.1 参数配置

1）确认冷热数据分离界线

示例：以年度为单位， 记录时间小于2022-01-01为冷数据，大于2022-01-01为热数据

```
postgres=# show manual_hot_date ;
manual_hot_date
---------------
2022-01-01
(1 row


postgres=# show cold_hot_sepration_mode;
cold_hot_sepration_mode
-----------------------
year
(1 row)


postgres=# show enable_cold_seperation;
enable_cold_seperation
----------------------
on
(1 row
```

注意：以上参数要求所有DN上均需要修改，重启DN生效。

2.2 创建冷数据组

1）查看当前组和DN节点

```
postgres=# select * from pgxc_group;
group_name   | default_group | group_members
---------------+---------------+---------------
default_group |             1 | 16385 16386
(2 rows)
```

2）查看当前节点信息

```
postgres=# select * from pgxc_node where node_type='D';
node_name | node_type | node_port |   node_host   | nodeis_primary | nodeis_preferred |   node_id   |  node_cluster_name
-----------+-----------+-----------+---------------+----------------+------------------+-------------+---------------------
dn001     | D         |     40004 | 192.168.2.136 | t              | t                |  2142761564 | opentenbase_cluster
dn002     | D         |     40004 | 192.168.2.137 | f              | f                |   -17499968 | opentenbase_cluster
dn003     | D         |     40004 | 192.168.2.138 | f              | f                | -1956435056 | opentenbase_cluster
(3 rows)
```

3）创建冷数据组

从上面查询可以确认，dn003未被使用

```
--CN上执行，不要到DN上
[opentenbase@db1 ~]$  psql -h 192.168.2.136 -p 30004 -d postgres -U opentenbase
create node group cold_group with(dn003);
create extension sharding group to group cold_group;
clean sharding;

--连接到DN3为冷组
[opentenbase@db1 ~]$  psql -h 192.168.2.138 -p 40004 -d postgres -U opentenbase
postgres=# select pg_set_node_cold_access();
pg_set_node_cold_access
-----------------------
success
```

**三、创建表**

```
--CN上操作
[opentenbase@db1 ~]$  psql -h 192.168.2.136 -p 30004 -d postgres -U opentenbase

DROP TABLE  cold_move_test;


CREATE TABLE cold_move_test(
id int not null,
identifynumber text NOT NULL,
inserttimeforhis timestamp without time zone NOT NULL,
primary key(id,identifynumber,inserttimeforhis))
partition by range (inserttimeforhis)
begin (timestamp without time zone '2020-01-01')
step (interval '1 year')
partitions (24)
DISTRIBUTE BY SHARD (identifynumber,inserttimeforhis) to GROUP default_group cold_group;

##查看对应的分区
pg中的分区表的命名规则如下
postgres=# select relname from pg_class where relname like '%cold_move%part%';
relname
-------
cold_move_test_part_0
cold_move_test_part_1
cold_move_test_part_2
cold_move_test_part_3
cold_move_test_part_4
cold_move_test_part_5
cold_move_test_part_6
cold_move_test_part_7
cold_move_test_part_8
cold_move_test_part_9
cold_move_test_part_10
cold_move_test_part_11
cold_move_test_part_12
cold_move_test_part_13
cold_move_test_part_14
cold_move_test_part_15
cold_move_test_part_16
cold_move_test_part_17
cold_move_test_part_18
cold_move_test_part_19
cold_move_test_part_20
cold_move_test_part_21
cold_move_test_part_22
cold_move_test_part_23
```

**四、插入数据**

```
1）--插入100条2020年的数据
for i in \`seq 100\`
do
psql -h 192.168.2.136 -p 30004 -d postgres -U opentenbase -c "insert into cold\_move\_test values(${i},'testtesti,′testtest${i}','2020-09-03 16:21:34.201133');" >/dev/null
done

2）--插入100条2021年数据
for i in \`seq 201 300\`
do
psql -h 192.168.2.136 -p 30004 -d postgres -U opentenbase -c "insert into cold\_move\_test values(${i},'testtesti,′testtest${i}','2021-12-03 16:21:34.201133');" >/dev/null
done

3）--插入100条2022年数据
for i in \`seq 301 400\`
do
psql -h 192.168.2.136 -p 30004 -d postgres -U opentenbase -c "insert into cold\_move\_test values(${i},'testtesti,′testtest${i}','2022-12-03 16:21:34.201133');" >/dev/null
done
```

**五、验证**

5.1 验证热数据访问的节点

当前从2022-01-01开始为热数据

```
postgres=# EXPLAIN SELECT COUNT(1) FROM cold_move_test where inserttimeforhis>'2022-01-01';
QUERY PLAN

Aggregate  (cost=103.44..103.45 rows=1 width=8)
->  Remote Subquery Scan on all (dn001,dn002)  (cost=100.00..103.36 rows=30 width=0)
->  Append  (cost=0.00..0.00 rows=0 width=0)
->  Seq Scan on cold_move_test (partition sequence: 2, name: cold_move_test_part_2)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold_move_test (partition sequence: 3, name: cold_move_test_part_3)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold_move_test (partition sequence: 4, name: cold_move_test_part_4)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold_move_test (partition sequence: 5, name: cold_move_test_part_5)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold_move_test (partition sequence: 6, name: cold_move_test_part_6)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold_move_test (partition sequence: 7, name: cold_move_test_part_7)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold_move_test (partition sequence: 8, name: cold_move_test_part_8)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold_move_test (partition sequence: 9, name: cold_move_test_part_9)  (cost=0.00..0.15 rows=1 wi
dth=0)
Filter: (inserttimeforhis > '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold_move_test (partition sequence: 10, name: cold_move_test_part_10)  (cost=0.00..0.15 rows=1

```

可以看到执行计划访问的dn001和dn002是热数据所在的default\_group组。

5.2 验证冷数据访问的节点

当前inserttimeforhis小于2022-01-01开始为冷数据

```
postgres=# EXPLAIN SELECT COUNT(1) FROM cold_move_test where inserttimeforhis<'2022-01-01';
QUERY PLAN

Remote Fast Query Execution  (cost=0.00..0.00 rows=0 width=0)
Node/s: dn003
->  Aggregate  (cost=5.00..5.01 rows=1 width=8)
->  Append  (cost=0.00..0.00 rows=0 width=0)
->  Seq Scan on cold_move_test (partition sequence: 0, name: cold_move_test_part_0)  (cost=0.00..2.25 rows=100
width=0)
Filter: (inserttimeforhis < '2022-01-01 00:00:00'::timestamp without time zone)
->  Seq Scan on cold_move_test (partition sequence: 1, name: cold_move_test_part_1)  (cost=0.00..2.25 rows=100
width=0)
Filter: (inserttimeforhis < '2022-01-01 00:00:00'::timestamp without time zone)
(8 rows)
```

可以看到执行计划访问的dn003是冷数据所在的cold\_group组。


<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

* AtomGit

  https://atomgit.com/opentenbase/OpenTenBase
* GitHub

  https://github.com/OpenTenBase/OpenTenBase
