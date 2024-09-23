---
title: "社区贡献 | OpenTenBase监控踩坑指南"
date: 2024-09-03T09:20:00+08:00
#image_webp: images/news/news-post-2.webp
image: images/news/news-post-14.png
author: OpenTenBase
description: ""
---


监控踩坑概览
======

首先，通过Docker安装了Prometheus，配置了必要的文件形式进行服务发现，实现了系统正常监控。接着，使用Docker启动Grafana，并配置数据源连接到Prometheus，展示监控面板。最后，安装了postgres\_exporter以监控数据库，并解决了启动报错问题。在配置监控面板时，通过Grafana的仪表板市场找到了适合的监控面板，并成功导入使用。

安装监控
====

Docker安装
--------

1、Docker要求 CentOs 系统的内核版本高于 3.10

通过 uname-r命令查看当前的内核版本

`uname -r`  

2、使用 root 权限登录 Centos。确保 yum 包更新到最新。

`yum -y update`  

3、卸载旧版本(如果安装过旧版本的话)

`sudo yum remove -y docker*`  

4、安装需要的软件包，yum-utl 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

`yum install -y yum-utils`  

5、设置yum源，并更新yum 的包索引

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  
yum makecache  

6、安装docker

`yum install -y docker-ce`  

8、启动并加入开机启动

`systemctl start docker && systemctl enable docker`  

9、验证安装是否成功(有client和service两部分表示docker安装启动都成功了)

`docker version`  

10、配置docker镜像

`cd etc/docker`  
然后编辑`vim daemon.json`  

{  
  "registry-mirrors": \["https://jbw52uwf.mirror.aliyuncs.com"\]  
}  

保存退出。

重启docker服务

systemctl daemon-reload  
systemctl restart docker  

下载Prometheus
------------

在进行监控优化时，您可以从Prometheus官方网站下载最新版：https://prometheus.io/download/

您可以选择下载源代码并解压使用，也可以通过Docker直接启动。本教程将重点介绍使用Docker进行快速部署。

执行命令：

`docker run -d -p 9090:9090 -v etc/prometheus:/etc/prometheus prom/prometheus`  

完成挂载后，请对配置文件进行必要的修改以确保系统正常监控。

`vim prometheus.yml`  

# my global config  
global:  
  scrape\_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.  
  evaluation\_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.  
  # scrape\_timeout is set to the global default (10s).  
  
# Alertmanager configuration  
alerting:  
  alertmanagers:  
    \- static\_configs:  
        \- targets:  
          # - alertmanager:9093  
  
# Load rules once and periodically evaluate them according to the global 'evaluation\_interval'.  
rule\_files:  
  # - "first\_rules.yml"  
  # - "second\_rules.yml"  
  
# A scrape configuration containing exactly one endpoint to scrape:  
# Here it's Prometheus itself.  
scrape\_configs:  
  # The job name is added as a label \`job=<job\_name>\` to any timeseries scraped from this config.  
  \- job\_name: "prometheus"  
  
    # metrics\_path defaults to '/metrics'  
    # scheme defaults to 'http'.  
  
    static\_configs:  
      \- targets: \["192.168.56.10:9090"\]  
  # 主要修改这里，添加文件形式的扫描  
  \- job\_name: "node"  
    file\_sd\_configs:  
    \- refresh\_interval: 10s  
      files:  
      \- "/etc/prometheus/conf/node\*.yaml"  

当前Prometheus的配置采用文件形式进行服务发现。在修改配置时，无需重新启动，系统将自动更新并生效，更新间隔为10秒。

为了修改相关配置文件，首先创建一个名为conf的目录（`mkdir conf`  
）然后通过cd命令进入目录（`cd etc/prometheus/conf`  
）接着使用vim编辑器来修改文件（`vim node-ms.yaml`  
）

\- targets:  
  \- "ip:port"  
  labels:  
    hostname: pg  

为了自定义配置信息，请将相应的IP地址和主机名修改为您自己的信息。完成修改后，启动Prometheus服务，然后您可以通过访问http://您的IP地址:9090/ 来查看Prometheus的监控数据。

下载Grafana
---------

为了确保配置的持久性，我们可以通过Docker容器以持久化形式启动Grafana。您可以使用以下命令来启动Grafana容器，并在容器重启后保留配置信息：

`docker run -d -p 3000:3000 --name=grafana --volume grafana-storage:/var/lib/grafana grafana/grafana-enterprise`  

启动后，您可以在浏览器中输入http://您的IP地址:3000/

使用默认的用户名和密码admin/admin登录，以查看Grafana监控界面。

### 配置数据源

<img src=../images/news-post-14-1.png class="img-fluid" /><br/>

在这里，只需填写URL（http://ip:9090/ ）即可保存配置。这个URL指向Prometheus的地址，Grafana将通过该地址与Prometheus建立连接，从而获取数据用于展示监控面板。

<img src=../images/news-post-14-2.png class="img-fluid" /><br/>

下载Exporter
----------

Prometheus官方提供了丰富的Exporter，您可以在https://prometheus.io/docs/instrumenting/exporters/ 找到相关信息。

我们可以安装postgres\_exporter来监控数据库，官方地址为https://github.com/prometheus-community/postgres\_exporter。

同样可以以Docker启动：

`docker run --net=host -e DATA_SOURCE_NAME="postgresql://opentenbase:@ip:port/postgres?sslmode=disable" quay.io/prometheuscommunity/postgres-exporter`  

ip和host修改为自己的信息即可，官方示例中对opentenbase用户并没有设置登录密码，我们也不设置密码进行登录。

启动后，我们首先登录到数据库中，然后进行数据库用户的相关设置。

```
CREATE OR REPLACE FUNCTION \_\_tmp\_create\_user() returns void as $$  
BEGIN  
  IF NOT EXISTS (  
          SELECT                       \-- SELECT list can stay empty for this  
          FROM   pg\_catalog.pg\_user  
          WHERE  usename = 'postgres\_exporter') THEN  
    CREATE USER postgres\_exporter;  
  END IF;  
END;  
$$ language plpgsql;  

SELECT \_\_tmp\_create\_user();  
  
DROP FUNCTION \_\_tmp\_create\_user();  
  
ALTER USER postgres\_exporter WITH PASSWORD 'password';  
  
ALTER USER postgres\_exporter SET SEARCH\_PATH TO postgres\_exporter,pg\_catalog;  
  
GRANT CONNECT ON DATABASE postgres TO postgres\_exporter;  
  
\-- OpenTenBase中集成的PostgreSQL版本是10，所以可以执行以下语句，历史版本可前往开源地址进行查看。  
GRANT pg\_monitor to postgres\_exporter;  
```
### postgres\_exporter启动报错修复
```
panic: Error converting setting "session\_memory\_size" value "3M" to float: strconv.ParseFloat: parsing "3M": invalid syntax  
  
goroutine 42 \[running\]:  
main.(\*pgSetting).metric(0xc000081720, 0xc0000d5c50?)  
        /app/cmd/postgres\_exporter/pg\_setting.go:87 +0x325  
main.querySettings(0x0?, 0xc00010d290)  
        /app/cmd/postgres\_exporter/pg\_setting.go:56 +0x287  
main.(\*Server).Scrape(0xc00010d290, 0xc000028011?, 0x90?)  
        /app/cmd/postgres\_exporter/server.go:121 +0xcb  
main.(\*Exporter).scrapeDSN(0xc0000000c0, 0x44d406?, {0xc000028011, 0x46})  
        /app/cmd/postgres\_exporter/datasource.go:115 +0x1c5  
main.(\*Exporter).scrape(0xc0000000c0, 0x0?)  
        /app/cmd/postgres\_exporter/postgres\_exporter.go:679 +0x16c  
main.(\*Exporter).Collect(0xc0000000c0, 0xc00003ff60?)  
        /app/cmd/postgres\_exporter/postgres\_exporter.go:568 +0x25  
github.com/prometheus/client\_golang/prometheus.(\*Registry).Gather.func1()  
        /go/pkg/mod/github.com/prometheus/client\_golang@v1.17.0/prometheus/registry.go:457 +0xe7  
created by github.com/prometheus/client\_golang/prometheus.(\*Registry).Gather in goroutine 18  
        /go/pkg/mod/github.com/prometheus/client\_golang@v1.17.0/prometheus/registry.go:547 +0xbab  
```
查看postgres\_exporter其源码发现端倪：

SELECT name, setting, COALESCE(unit, ''), short\_desc, vartype FROM pg\_settings WHERE vartype IN ('bool', 'integer', 'real') AND name != 'sync\_commit\_cancel\_wait';  

<img src=../images/news-post-14-3.png class="img-fluid" /><br/>

确实是因为session\_memory\_size的显示问题，不过已经提交了PR修复，官方修复后即可成功。

<img src=../images/news-post-14-4.png class="img-fluid" /><br/>

配置监控面板
======

一旦所有组件都成功启动，接下来我们需要前往市场寻找我们想要的监控面板。你可以访问Grafana的官方仪表板市场：https://grafana.com/grafana/dashboards/?search=postgresql

<img src=../images/news-post-14-5.png class="img-fluid" /><br/>

一旦找到您喜欢的面板，请点击此处进行导入。以下以ID：9628为示例进行导入操作。

这里选择我们的数据源。

<img src=../images/news-post-14-6.png class="img-fluid" /><br/>

让我们来看一下效果如何：

<img src=../images/news-post-14-7.png class="img-fluid" /><br/>

<img src=../images/news-post-9-11.png class="img-fluid" /><br/>

我们目前正在积极征集OpenTenBase的用户使用案例，如果您有相关使用经验，欢迎提交给我们。也期待您加入OpenTenBase社区，跟我们共同推动项目发展！

<img src=../images/news-post-9-12.png class="img-fluid" /><br/>

**官网：** https://www.opentenbase.org/

**贡献代码**

*   AtomGit
    
    https://atomgit.com/opentenbase/OpenTenBase
    
*   GitHub
    
    https://github.com/OpenTenBase/OpenTenBase