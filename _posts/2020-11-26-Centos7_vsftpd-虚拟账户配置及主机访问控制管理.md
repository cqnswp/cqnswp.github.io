
---
layout: mypost
title: Centos7_vsftpd-虚拟账户配置及主机访问控制管理
categories: [网络与运维]
---

## FTP服务器配置系列： [Centos7_vsftpd-ssl/tls搭建维护及ftp加固](https://blog.csdn.net/qq_38626043/article/details/103698187)
## 一. FTP虚拟账户配置
 FTP虚拟账户管理分配
### 配置虚拟用户
```bash
$ useradd   vsftpd -d    /home/vsftpd -s   /bin/false		#新建用户并禁止登录
$ mkdir -p /home/vsftpd/ftp1								#新建用户目录及虚拟账户
$ vim  /etc/vsftpd/login.conf  								#新建一个用户登录文件
ftp1														#写入账户密码
123456
```

### 1. 创建启动数据库

```bash
$ db_load -T -t hash -f /etc/vsftpd/login.conf /etc/vsftpd/login.db		
$ chmod 700 /etc/vsftpd/login.db
$ vim /etc/pam.d/vsftpd
# 注释掉原来所有内容后，增加下面的内容

auth    sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/login
account sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/login

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200609163613397.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
### 2. 增加虚拟用户数据库配置文件

```bash
$ mkdir /etc/vsftpd/userconf         #创建虚拟用户配置文件目录
$ vim   /etc/vsftpd/userconf/ftp1   #这里的文件名必须与前面指定的虚拟用户名一致
local_root=/home/vsftpd/ftp1/
write_enable=YES
```

### 3. 修改主配置文件

```bash
$ vim   /etc/vsftpd/vsftpd.conf    		#存在的修改，不存在的增加
anonymous_enable=NO          		    #禁止匿名用户登录
chroot_local_user=YES           		#禁止用户访问除主目录以外的目录
ascii_upload_enable=YES          		#设定支持ASCII模式的上传和下载功能   
ascii_download_enable=YES     			#设定支持ASCII模式的上传和下载功能   
guest_enable=YES                     	#启动虚拟用户
guest_username=vsftpd             		#虚拟用户使用的系统用户名
user_config_dir=/etc/vsftpd/userconf   	#虚拟用户使用的配置文件目录
allow_writeable_chroot=YES      		#最新版的vsftpd为了安全必须用户主目录（也就是/home/vsftpd/ftp1）没有写权限，才能登录
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/2020060917232527.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

## PS：虚拟账户权限及新建配置
配置虚拟账户管理员及单独的下载用户，上传用户保证ftp服务器安全
- 配置新建用户需创建用户配置文件/etc/vsftpd/userconf/
- 新建用户登录文件 /etc/vsftpd/login.conf/
- 重新创建覆盖数据库

新建用户数据库配置文件 /etc/vsftpd/userconf/username
根据需求增加以下配置

```bash
anon_world_readable_only=NO                # 浏览FTP目录和下载 
anon_upload_enable=YES                     # 允许上传 
anon_mkdir_write_enable=YES                # 建立和删除目录 
anon_other_write_enable=YES                # 改名和删除文件 
local_root=/ftpdir/                        # 指定虚拟用户在系统用户下面的路径，限制虚拟用户的家目录，虚拟用户登录后的主目录。 
```
## 二. 主机访问控制管理
### 1.修改hosts.allow，添加允许访问的IP

```bash
$ vim /etc/hosts.allow
vsftpd:IPaddress
```

### 2. 修改hosts.deny，禁用除了host.allow 以外所有IP地址访问FPT服务器

```bash
$ vim /etc/hosts.deny
vsfptd:ALL
```
### 以上配置均以需求调整

 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200610151901394.png#pic_center)

## 附 vsfptd.conf 常用配置参数

```bash
一： 进程类别优化：
     1：listen=YES/NO           设置独立进程控制vsftpd
二： 登录和访问控制选项优化：
1：anonymous_enable=YES/NO  允许/禁止匿名用户登陆
2：banned_email_file=/etc/vsftpd/vsftpd.banned_emails允许/禁止邮件的使用的存放路径和目录    
配合使用：deny_email_enable=YES/NO          允许/禁止匿名用户使用邮件作为密码
     3：banner_file=/etc/vsftp/banner_file         在文件banner_file添加欢迎词即可
     4：cmds_allowed=HELP,DIR,QUIT,!       出被允许使用的FTP命令
     5：ftpd_banner=welcome to ftp server  和第三条一样是欢迎词屏幕，方法不一样
     6：local_enable=YES/NO        允许/禁止本地用户登陆
     7：pam_service_name=vsftpd     使用pam模块进行ftp客户端的验证
     8：userlist_deny=YES/NO       禁止/允许文件列表user_list的用户访问ftp服务器
       配合使用：userlist_file=/etc/vsftpd/user_list   用户列表文件
       配合使用：userlist_enable=YES/NO   激活/失效第8条的功能 
     9：tcp_wrappers=YES/NO    启用/不启用tcp_wrappers控制服务访问的功能              
三： 匿名用户选项的优化：
      1：anon_mkdir_write_enable=YES/NO   允许/禁止匿名用户创建目录、删除文件
      2：anon_root=/path/to/file         设置匿名用户的根目录，默认是/var/ftp/ 你可以修改这个默认路径
      3：anon_upload_enable=YES/NO        允许/禁止匿名用户上传
      4：anon_world_readable_only=YES/NO  禁止/允许匿名用户浏览目录和下载
      5：ftp_username=anonftpuser         把匿名用户绑定到系统用户名
      6：no_anon_password=YES/NO          不需要/需要匿名用户的登录密码
四：本地用户选项的优化：
      1：chmod_enable=YES/NO               允许/禁止本地用户修改文件权限
      2：chroot_list_enable=YES/NO         启用/不启用禁锢本地用户在家目录
      3：chroot_list_file=/path/to/file    建立禁锢用户列表文件，一行一个用户
      4：guest_enable=YES/NO               激活/不激活虚拟用户
      5：guest_username=系统实体用户       把虚拟用户绑定在某个实体用户上
      6：local_root=/path/to/file          指定或修改本地用户的根目录
      7：local_umask=具体权位数字          设置本地用户新建文件的权限
      8：user_config_dir=/path/to/file     激活虚拟用户的的主配置文件
     五：目录选项的优化：
      1：text_userdb_names=YES/NO          启用/禁用用户的名称取代用户的UID
     六：文件传输选项优化：
      1：chown_uploads=YES/NO              启用/禁用修改匿名用户上传文件的权限
         配合使用：chown_username=账户     指定匿名用户上传文件的所有者
      2：write_enable=YES/NO               启用/禁止用户的写权限
      3：max_clients=数字                  设置FTP服务器同一时刻最大的连接数
      4：max_per_ip=数字                   设置每ip的最大连接数
      七：网络选项的优化：
      1：anon_max_rate=数字                设置匿名用户最大的下载速率（单位字节）
      2：local_max_rate=数字               设置本地用户最大的下载速率
```

