---
layout: mypost
title: Centos7-Zabbix4.0监控系统部署
categories: [网络与运维]
---
## 一. zabbix概述
官方文档：[https://www.zabbix.com/documentation/4.0/zh/manual/installation/requirements](https://www.zabbix.com/documentation/4.0/zh/manual/installation/requirements)
1. Zabbix 是一个基于 WEB 界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。它能监视各种网络参数，保证服务器系统的安全运营；并提供灵活的通知机制以让系统管理员快速定位/解决存在的各种问题。
2. 相对 Prometheus 等监控软件zabbix对于web界面的操作性更强，更加美观方便。

## 二. zabbix组件
- zabbix-server: 监控服务端

- zabbix-agent: 监控客户端

- zabbix-web: 监控网站服务

- php: 处理动态请求

- mysql: 数据库存储监控数据

- zabbix: 负责收集agent信息汇总告知zabbix-server


## 三. 搭建zabbix

角色| ip
-------- | -----
 Zabbix监控机| 192.168.5.104
服务机| 客户端库
 Push Gateway  | 192.168.5.111


监控服务端占用 10051 端口 监控客户端占用 10050 端口
### 1. 关闭selinux和防火墙
```powershell
$ vim /etc/sysconfig/selinux/
SELINUX=disabled
$ systemctl stop firewalld
# systemctl disable firewalld
```
### 2. 添加zabbix，epel源
如果你是用的阿里或者其它源，换清华源，阿里源没有相关组件，直接下载容易连接中断 *Error downloading packages: zabbix-web-4.0.23-1.el7.noarch: [Errno 256] No* 
清华源：
[https://mirrors.tuna.tsinghua.edu.cn/](https://mirrors.tuna.tsinghua.edu.cn/)

```powershell
$ rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm
$ wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo    
```
### 3. 使用apache做web服务器
安装相关组件，
zabbix会需要php，mysql；mariadb，如果有可以掠过

```powershell
$ wget  https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/{zabbix-server-mysql-4.0.20-1.el7.x86_64.rpm,zabbix-web-4.0.22-1.el7.noarch.rpm}		//如果还是不成功可以提前下好
$ yum install -y httpd php zabbix-server-mysql-4.0.20- 1.el7.x86_64.rpm  zabbix-web-4.0.22-1.el7.noarch.rpm
$ yum install -y mariadb-server
```
安装数据库服务
```powershell
$ yum install -y mariadb-server
```
*ps：如果是首次安装请初始化数据库*
```powershell
$ mysql_secure_installation
```
### 4. 使用nginx做web服务器
使用nginx做web服务器其实也可以，其它操作几乎一样，中间件不一样而已。
安装好nginx后
修改nginx配置文件server块和location块

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812170022801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
修改php.ini时区
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812170052393.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)





### 5. 配置数据库密码时区

```powershell
$ vim /etc/zabbix/zabbix_server.conf
DBPassword=youpassword

$ vim /etc/httpd/conf.d/zabbix.conf
php_value date.timezone Asia/Shanghai
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811190224444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

### 6. 导入数据库初始化



```sql
$ mysql -u root -p 
create database zabbix character set utf8 collate utf8_bin;
grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';
$ zcat /usr/share/doc/zabbix-server-mysql*/create.sql. gz|mysql -uzabbix -pzabbix zabbix
```


创建数据库管理用户遇到不符合密码策略的修改密码策略
*ERROR 1819 (HY000): Your password does not satisfy the current policy requirements*
```sql
$ set global validate_password_policy=LOW;
$ set global validate_password_length=6;
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811203214244.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

初始化地址

**/zabbix/setup.php**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811205116463.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
0 代表默认3306端口
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811205218305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
账号：Admin
密码：zabbix
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811205424642.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811205710199.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)


## 四. 配置服务机
### 1. 下载安装客户端
下载配置监控客户端
对应版本在清华源中能找到
[https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/](https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/)
```powershell
$ wget https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabb ix/4.0/rhel/7/x86_64/zabbix-agent-4.0.20-1.el7.x86_64.rpm
$ yum install -y zabbix-agent-4.0.20-1.el7.x86_64.rpm
$ vim /etc/zabbix/zabbix_agentd.conf
Server=192.168.5.104
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811213434531.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081121363218.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
### 2. 回到监控端配置接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811214406808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811221646660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

### 3. 出现乱码或者不显示问题
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812161240388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)


在windows找到合适的字体文件并scp服务器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812161358393.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

移动修改字体文件
```powershell
$ mv simkai.ttf /usr/share/zabbix/assets/fonts/simkai.ttf
$ vim /usr/share/zabbix/include/defines.inc.php
define('ZBX_GRAPH_FONT_NAME',           'simkai'); 
define('ZBX_FONT_NAME', 'simkai');
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812161944662.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812162051668.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

刷新浏览器
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081216221332.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)


## 五. 配置邮件报警功能
### 1. 安装配置邮件服务器
```powershell
$ yum install -y mailx
$ vim /etc/mail.rc
set from=邮箱地址
set smtp=smtp地址
set smtp-auth-user=邮箱登录账户
set smtp-auth-password=授权码或密码
set smtp-auth=login
# test
$ echo "hello mail" | mail -s "liudonghui_zabbix-install" 123@yeah.net
邮件格式： echo "邮件内容" | mail -s "邮件主题" 收件人邮箱地址
```
### 2. 编写邮件脚本

```powershell
$ cd /usr/lib/zabbix/alertscripts/
$ vim mailx.sh
#!/bin/bash                                                       
##################################                                
# File Name:mailx.sh                                              
# Version:V1.0                                                    
# Author:cqnswp                                                   
# Organization:blog.cqnswp.ltd                                    
# Created Time:2020-08-12                                         
# Description:EmailServer                                         
##################################                                
messages=`echo $3 | tr '\r\n' '\n'`                               
subject=`echo $2 | tr '\r\n' '\n'`                                
echo "${messages}" | mail -s "${subject}" $1 >>/tmp/mailx.log 2>&1
```

### 3. 配置报警媒介

 **管理->报警媒体类型->创建媒体类型**
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812185635105.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812185645958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

```powershell
{ALERT.SENDTO}

{ALERT.SUBJECT}

{ALERT.MESSAGE}
```
**用户->Admin>报警媒介**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812190124215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

**配置->动作->创建动作**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812190234160.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
默认模板
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081219041238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812185530118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081218461627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812185436980.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812190654669.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
详细的告警通知内容：

```powershell
告警标题:故障({TRIGGER.STATUS}),服务器:({HOSTNAME1}发生:{TRIGGER.NAME})故障！
告警信息:
告警事件ID: {EVENT.ID}
告警主机IP: {HOST.IP}
告警主机: {HOSTNAME1}
告警时间: {EVENT.DATE}-{EVENT.TIME}
告警等级: {TRIGGER.SEVERITY}
告警信息: {TRIGGER.NAME}
告警项目: {TRIGGER.KEY1}
问题详情: {ITEM.NAME}:{ITEM.VALUE}
当前状态: {TRIGGER.STATUS}:{ITEM.VALUE1}
```
恢复告警内容：

```powershell
恢复标题:恢复({TRIGGER.STATUS}),服务器:({HOSTNAME1}:{TRIGGER.NAME})已恢复！
恢复信息:
告警事件ID: {EVENT.ID}
告警主机IP: {HOST.IP}
告警主机: {HOSTNAME1}
告警时间: {EVENT.DATE}-{EVENT.TIME}
告警等级: {TRIGGER.SEVERITY}
告警信息: {TRIGGER.NAME}
告警项目: {TRIGGER.KEY1}
问题详情: {ITEM.NAME}:{ITEM.VALUE}
当前状态: {TRIGGER.STATUS}
```
简单的告警通知内容：

```powershell
{TRGGER.STATUS}:{TRIGGER.NAME}

告警主机：{HOST,NAME}
告警 IP：{HOST.IP}
告警时间：{EVENT.DATE}-{EVENT.TIME}
告警等级：{TRIGGER.NAME}:{ITEM.VALUE}
事件 ID ：{EVVNT.ID}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812191420237.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)













