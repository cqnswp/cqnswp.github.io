---
layout: mypost
title: Centos7_vsftpd-ssl/tls搭建维护及ftp加固
categories: [信息安全]
---

## FPT服务配置系列：[Centos7_vsftpd-虚拟账户配置及管理](https://blog.csdn.net/qq_38626043/article/details/106665601)
环境
WEB-02	vsFTPd (ftp.nlsz.ru)
FTP(File Transfer Protocol)文件传输协议
登录ftp服务器上的有三类账户
Real（ftp上拥有的账户）
Guest（来宾账户）
Anonymous(匿名账户）
FTP服务器应仅接受显式SSL / TLS连接
SSL / TLS证书保护vsftpd数据通信

- vsftp的服务进程是vsftpd
- vsftpd的配置文件是/etc/vsftpd/vsftpd.conf .
- vsftpd的用户文件是/etc/vsftpd/ftpusers
- vsftpd的用户文件是/etc/vsftpd/user_list


测试vsftpd是否支持ssl
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225140656953.png#pic_center)

## 防火墙策略

```bash
firewall-cmd --add-port=21/tcp --permanent
firewall-cmd --add-port=20/tcp --permanent  
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225140938296.png#pic_center)

## 生成证书和密钥

```bash
openssl req -x509 -nodes -newkey rsa:2024 \-keyout /etc/vsftpd/vsftpd.pem \-out /etc/vsftpd/vsftpd.pem -days 365
```
- req 是x509证书签名的请求
- x509 表示使用x509证书管理数据
- newkey 指定证书密钥处理
- rsa RSA密钥加密 我这里采用的是2024位私有密钥
- keyout 指定密钥存储位置
- out 指定证书存储位置
- days 证书有效天数 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225141059532.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
## 配置vsftpd服务器
/etc/vsftpd/vsftpd.conf
根据需求添加以下

```powershell
ssl_enable=yes/no             	//是否启用 SSL,默认为no
allow_anon_ssl=yes/no         	//是否允许匿名用户使用SSL,默认为no
rsa_cert_file=/path/to/file     //rsa证书的位置
dsa_cert_file=/path/to/file     //dsa证书的位置
force_local_logins_ssl=yes/no   //非匿名用户登陆时是否加密,默认为yes
force_local_data_ssl=yes/no    	//非匿名用户传输数据时是否加密,默认为yes
force_anon_logins_ssl=yes/no    //匿名用户登录时是否加密,默认为no
force_anon_data_ssl=yes/no     	//匿名用户数据传输时是否加密,默认为no
ssl_sslv2=yes/no              	//是否激活sslv2加密,默认no
ssl_sslv3=yes/no                //是否激活sslv3加密,默认no
ssl_tlsv1=yes/no                //是否激活tls v1加密,默认yes
ssl_ciphers=加密方法             //默认是DES-CBC3-SHA
listen_port=21					//指定端口
```
##### 关于vsftpd服务器加固

```powershell
require_ssl_reuse =no
ssl_ciphers = HIGH		
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225141825151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
## 关闭selinux
/etc/selinux/config
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225152135834.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
重启vsftpd服务器尝试登录会报一个错误


530 Non-anonymous sessions must use encryption.
命令行不提供加密服务，因此会产生错误。因此，为了安全地连接到服务器，我们需要一个支持SSL / TLS连接的FTP客户端![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225142737930.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

随便在网上下一个支持ssl传输的ftp客户端
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225143012207.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

保存证书即可
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225143019856.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225144546108.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

## 关于vsftpd加固及优化

```powershell
anonymous_enable=no				//关闭匿名用户登录
anon_upload_enable=no			//禁用匿名上传
anon_mkdir_write_enable=no		//禁用匿名新建文件夹
anon_other_write_enable=no  	//禁用匿名账号rwx权限
local_enable=yes				//本地账户登录后无权删除修改文件
chroot_list_enable=YES			//只允许本地用户在自己目录
write_enable=no					//用户的写权限
idle_session_timeout=60			//以秒为单位，1分钟无操作断开
local_max_rate=50000			//以bite为单位，本地用户传输速率
max_clients=200					//FTP最大连接数
max_per_ip=1					//每个ip单线程下载
xferlog_enable=YES				//开启日志文件
listen=yes						//设置独立进程
cmds_allowed=HELP,DIR,QUIT,!	//允许使用的ftp命令

```
ftp的登录会被xferlog记录
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225151616726.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225145819867.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
/var/ftp下创建一个.message
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225150255655.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191225150145934.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

## FTP 服务器资源限制
ftp主配置文件：

```bash
max_clients=1            #ftp服务器最大允许客户连接数为1，参数为0时表示不限制。
max_per_ip=1             #ftp服务器对同一IP允许的最大客户端连接数为1。
local_max_rate=800000    #本地用户最大传输速率为800km/s。
anon_max_rate=400000     #匿名用户最大传输速率为200km/s。
```


