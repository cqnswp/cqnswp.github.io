---
layout: mypost
title: Docker Prometheus 监控系统
categories: [Docker+k8s]
---


## 一. Prometheus 概述
无监控不运维~
### 1. 概述
prometheus自16年k8s托管以来，几乎是容器技术应用的最广的监控系统，除了普罗米修斯，还有zabbix（图形界面友好，成熟，分级） 等 成熟的监控系统。




官方：
[https://prometheus.io/](https://prometheus.io/)
[https://github.com/prometheus](https://github.com/prometheus)




### 2. 监控系统作用及Prometheus的特点
- 装逼（给老板看）
- 业务持续性（及时预警，监控）
- 多维数据模型：由度量名称和键值对标识的时间序列数据 TSDB
- PromQL：一种灵活的查询语言，可以利用多维数据完成复杂的查询
- 不依赖分布式存储，单个服务器节点可直接工作
- 基于HTTP的pull方式采集时间序列数据
- 推送时间序列数据通过PushGateway组件支持
- 通过服务发现或静态配置发现目标
- 多种图形模式及仪表盘支持（grafana）






### 3. Prometheus 基本服务与作用

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712224212453.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

主要服务| 作用
-------- | -----
 Prometheus Server| 收集指标和存储时间序列数据，并提供查询接口
ClientLibrary| 客户端库
 Push Gateway  | 短期存储指标数据。主要用于临时性的任务
 Exporters|采集已有的第三方服务监控指标并暴露metrics
 Alertmanager|告警
 Web UI|简单的Web控制台
 
左边的采集，中间服务，右边展示


### 4. 概念
实例（被监控端 Instances）
作业（实例的集合 Job）

## 二. Prometheus exporter cadvisor 的部署
官方文档：
[https://prometheus.io/docs/prometheus/latest/installation/](https://prometheus.io/docs/prometheus/latest/installation/)

Prometheus支持二进制，docker等部署方式

### 1. docker部署prometheus

配置文件需要自己写的
```bash
$ docker container run -itd -p 9090:9090 -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```







### 2. prometheus.yml 模板
配置文件模板
```bash
# prometheus.yml 
# my global config
global:
#全局配置
#全局配置节点下的配置对所有其它节点都有效，同时也是其它节点的默认值。
  scrape_interval:     15s # 将刮擦间隔设置为每15秒一次。 默认值为每1分钟。
  evaluation_interval: 15s # 每15秒评估一次规则。 默认值为每1分钟。
  # scrape_timeout设置为全局默认值（10s）。

# Alertmanager configuration
alerting:
#告警配置
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# 一次加载规则，并根据全局'evaluation_interval'定期评估它们。
rule_files:
#规则配置
#规则配置包含记录规则配置和告警规则配置，节点下只是列出文件，具体配置在各个文件中。
  # - "first_rules.yml"
  # - "second_rules.yml"

# 一个仅包含一个要刮擦的端点的刮擦配置：
# 这里是普罗米修斯本身。
scrape_configs:
  # 作业名称作为标签“ job = <作业名称>”添加到从此配置中刮取的任何时间序列。
  - job_name: 'prometheus'
#抓取配置
#抓取配置可以有多个，一般来说每个任务（Job）对应一个配置。

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
#抓取实例目标
#目标地址列表，地址由主机+端口组成
  - job_name: "docker"
    static_configs:
    - targets: ['ip:8080']
#抓取配置
  - job_name: "Linux"ip
    static_configs:
    - targets: ['ip:9100']
#抓取配置
```

这是Prometheus自带的一个界面，通常用于 测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712235623468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

### 3. 监控目标

- 容器服务
- 主机服务

**容器的监控指标概念**
内存，cpu，硬盘，网络，状态

主机服务监控指标
一般使用 exporter 来监控主机服务
[https://github.com/prometheus/node_exporter/](https://github.com/prometheus/node_exporter/)



```bash
$ docker stats --no-stream 452f0102c634
$ docker stats --no-stream 452f0102c634 |awk '{print $3}'
```
主机| 业务
-------- | -----
主机一| 做nginx容器，和 cAdvisor 采集，做主机服务，和 exporter 的采集
主机二| 做pometheus监控（通过cAdvisor，exporter ），和Grafana可视化

### 4. 通过exporter搜集主机服务资源接口
因为主机监控的特殊性，官方不建议用docker监控主机服务，所以采用二进制本地安装
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200713175255617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
下载exporter二进制文件安装

```bash
$ wget https://github.com/prometheus/node_exporter/releases/download/v0.17.0/node_exporter-0.17.0.linux-amd64.tar.gz
$ tar -zxvf node_exporter-0.17.0.linux-amd64.tar.gz
$ mv node_exporter-0.17.0.linux-amd64 /usr/local/node_exporter
```
systemctl service 服务
systemctl service在两个目录中：
 /etc/systemd/system
 /usr/lib/systemd/system
 一般自己创建的service 直接放在 /etc/systemd/system 之中

```bash
$ vim /usr/lib/systemd/system/node_exporter.service
[Unit]
Description=https://prometheus.io

[Service]
Restart=on-failure
ExecStart=/usr/local/node_exporter/node_exporter

[Install]
WantedBy = multi-user.target
$ systemctl start node_exporter
```
node_exporter暴露一个9100端口

访问即可得到指标

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020071318322582.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

### 5. 通过cAdvisor搜集容器资源使用信息
官方：
[https://github.com/google/cadvisor](https://github.com/google/cadvisor)
[https://grafana.com/grafana/download](https://grafana.com/grafana/download)

cadvisor通过容器运行，通过本地映射。
这个是官方的文档，但这个下载链接是被墙了的
```bash
$ VERSION=v0.36.0 # use the latest release version from https://github.com/google/cadvisor/releases
$ sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  gcr.io/google-containers/cadvisor:$VERSION
```
所以一般先在国内拉取cadvisor，在进行容器的创建。

```bash
$ docker container run -d --volume=/:/rootfs:ro --volume=/var/run:/var/run:ro --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/dock er:ro --volume=/dev/disk/:/dev/disk:ro --publish=8080:8080 --detach=true --name=cadvisor google/cadvisor:latest
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200713022653423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200713022850614.png#pic_center)

http://192.168.43.14:8080/**metrics** 暴露数据指标（接口），但cadvisor只负责采集，不负责存储，所以需要配合Prometheus

### 6. 修改pometheus配置文件

重启pometheus容器

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200713183522262.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200713184247243.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

数据也有了

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200713023717523.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

## 三. Grafana做可视化展示
### 1. Grafana
Grafana的部署很简单，是一个纯静态的系统，拉取镜像暴露一个端口即可，
```bash
$ docker pull grafana/grafana
$ docker container run -d --name=grafand -p 3000:3000 grafana/g rafana
```
默认账号密码 admin

社区有教程美化Grafana ，有需求可以看一下
[https://grafana.com/tutorials/grafana-fundamentals/?utm_source=grafana_gettingstarted#2](https://grafana.com/tutorials/grafana-fundamentals/?utm_source=grafana_gettingstarted#2)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200713024438293.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
通过自己画或者社区导入自己喜欢的仪表盘
[https://grafana.com/grafana/dashboards](https://grafana.com/grafana/dashboards)
 
推荐id：
nginx：193
linux：9276,8919

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200929092548519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

### 2. 效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200713031350834.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020071318462288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200929092803124.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)


### 3. grafana密码忘记问题
修改数据库默认密码，user：admin pass：admin
```sql
$ update user set password = '59acf18b94d7eb0694c61e60ce44c110c7a683ac6a8f09580d626f90f4a242000746579358d77dd9e570e83fa24faa88a8a6', salt = 'F3FAxVm33R' where login = 'admin';
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200929091701703.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

修修改后登录更改新密码

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200929091459385.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

### 4. 数据源扩展问题与grafana目录结构
grafana自带的数据源不能满足部分需求，社区提供绝大多数数据源的安装接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200929115524305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

```bash
$ docker container exec -it dc5683c985b3 /bin/bash
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200929115806106.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
自行导入重启即可

```bash
$ grafana-cli plugins install fastweb-openfalcon-datasource
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200929115701501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

grafana 目录结构

```bash
安装目录
/usr/share/grafana/
grafana-cli 路径
/usr/share/grafana/bin/grafana-cli
全局配置文件
/etc/grafana/grafana.ini
默认配置文件
/usr/share/grafana/conf/defaults.ini
plugins 安装目录
/var/lib/grafana/plugins/
默认数据存储文件路径
/var/lib/grafana/grafana.db
日志文件存储路径
/var/log/grafana/
邮件默认发送模板路径
/usr/share/grafana/public/emails/
```


