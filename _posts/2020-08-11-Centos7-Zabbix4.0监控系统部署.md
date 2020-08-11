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
| 角色 | ip |
|--|--|--|
| Zabbix监控机 | 192.168.5.104 |
| 服务机| 192.168.5.111|

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

### 4. 配置数据库密码时区

```powershell
$ vim /etc/zabbix/zabbix_server.conf
DBPassword=youpassword

$ vim /etc/httpd/conf.d/zabbix.conf
php_value date.timezone Asia/Shanghai
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811190224444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

### 5. 导入数据库初始化



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
回到监控端配置接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811214406808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811221646660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)







