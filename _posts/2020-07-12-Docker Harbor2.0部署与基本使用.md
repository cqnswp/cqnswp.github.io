---
layout: mypost
title: Docker Harbor2.0部署与基本使用
categories: [Docker+k8s]
---

## 一. harbor概述
### 1. 基本功能
harbor是一个由vm公司**开源的企业级容器镜像仓库**
包括企业级特性：
- 管理用户界面
- 基于角色的访问控制
- LDAP/AD 集成及日志审计等基本运维操作


官方网站：
[https://vmware.github.io/harbor/cn/](https://vmware.github.io/harbor/cn/)

[https://github.com/vmware/harbor](https://github.com/vmware/harbor)

### 2. harbor 的基本组件

组件| 功能
-------- | -----
harbor-adminserver | 配置管理中心
harbor-db | 数据库
harbor-jobservice  | 镜像复制
harbor-log|日志操作
harbor-ui|Web管理页面和API
nginx|前端代理，负责前端页面和镜像上传/下载转发
redis|会话
registry|镜像存储


## 二. harbor部署
### 1. harbor的安装方式
harbor安装有三种方式

- 在线安装：从docker hub 下载harbor相关镜像，安装包较小
- 离线安装：安装包包含部署的相关镜像，安装包较大
- OVA安装：当用户具有vCenter环境时，使用此安装，部署OVA后启动harbor

离线安装环境较为齐全，所有我们采用离线安装。

### 2. 安装（老版本与新版本）

前期需要准备的：

1. 下载harbor离线安装包
下载地址：[https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases)
2. 安装 docker compose 
官方文档：[https://docs.docker.com/compose/#common-use-cases](https://docs.docker.com/compose/#common-use-cases)
指定版本：[https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)

docker compose稳定版linux，其它版本查看官方文档
docker compose是一个docker容器编排工具，利用它可以一键创建一个容器框架
这个 compose 的下载地址被墙了，所以要那个才能下载。

```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.26.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
$ docker-compose --version
```



新版harbor与旧版有些不同，新版需要将harbor导入docker镜像
**新版：**
```bash
$ tar -zxvf harbor-offline-installer-v2.0.1.tgz
$ cd harbor
$ docker image load -i harbor.v2.0.1.tar.gz
$ cp harbor.yml.tmpl harbor.yml
$ vim harbor.yml
hostname = 主机域名或ip
harbor_admin_password = Harbor12345			//这个harbor密码务必确认，修改起里非常麻烦
$ ./prepare
$ ./install.sh
```


**旧版：**

```bash
$ tar -zxvf harbor-offline-installer-v1.6.1.tgz
$ cd harbor
$ vi harbor.cfg  					//修改配置文件
hostname = 主机域名或ip
ui_url_protocol = http				//使用协议
harbor_admin_password = 123456		//登录web密码
$ ./prepare
$ ./install.sh
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020070404324427.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200704045218483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200704045302294.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

### 3. 添加信任仓库

```bash
$ docker info
```
后面有个 Registry Mirrors: 表示在此镜像站中拉去的镜像都是信任的，比如网易的源，docker官方的源。
因为我们的镜像仓库在本地，所以需要我我们的harbor添加信任才能拉取上传镜像。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200704193718320.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
此配置文件在 /etc/docker/daemon.json
添加自己的信任然后重启
**如何你采用https，就在 registry-mirrors 后面加上就行了，如果采用http，自己添加 Insecure Registries:**

```bash
$ vim /etc/docker/daemon.json
{"insecure-registries":["reg.ctnrs.com"]}
$ systemctl restart docker
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200704183246861.png#pic_center)

### 4. https配置
在本地使用的可以自己给自己签证书
**创建 CA 证书（自签名证书）**
req申请证书，newkey新证书，-x509证书标准，rsa密钥，sha256加密算法，keyout私钥名，days证书有效期，out输出密钥文件
```bash
$ mkdir /root/ca -p
$ cd /root/ca
$ openssl req  -newkey rsa:4096 -nodes -sha256 -keyout ca.key -x509 -days 365 -out ca.crt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200704184656712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

**为harbor生成证书签名请求文件**

```bash
$ openssl req  -newkey rsa:4096 -nodes -sha256 -keyout harbor.key -out harbor.csr
```
可 **使用IP进行签名**

```bash
$ echo subjectAltName = IP需要连接的ip:> extfile.cnf
-extfile extfile.cnf  //如果使用ip签名在后面签名加上
```

**使用CA证书 给 harbor证书的签名请求进行签名**
生成一个harbor.crt的文件
```bash
$ openssl x509 -req -days 365 -in harbor.csr -CA ca.crt -CAkey ca.key -CAcreateserial -extfile extfile.cnf -out harbor.crt
```
**配置harbor配置文件**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200704193125720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

**重启服务**
在配置目录运行
```bash
$ docker-compose down
$ ./prepare
$ docker-compose up –d
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200704192141584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)


## 三. harbor基本使用


### 1. 更新harbor配置后重启操作
利用docker-compose
```bash
$ docker-compose down
$ ./prepare
$ docker-compose up –d
```

### 2. harbor登录密码问题
忘记或者更改密码操作，无法在配置文件中解决，需要harbor-db更改数据库解决
但是harbor的加密问题，如果对外使用，管理密码一定在第一次配置生成容器时设置好，后面修改配置文件中的密码是不会生效的，密码已经写入数据库，如果在生成harbor后想修改密码，只能修改数据库密码。
老版本使用mysql，新版使用postgresql 
数据库的默认密码是 **root123**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712165942220.png#pic_center)

密码是通过sha256加盐得到的一般是撞不出来的，这个密码站时没找到合适的解决办法。
```bash
$ docker container exec -it 40046344663c bash
$ psql -h postgresql -d postgres -U postgres
root123
$ \c registry
$ select * from harbor_user;
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712163814382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)


### 3. harbor的基本使用

其工作模式跟 git 本地 远程仓库类似
```bash
$ docker tag 90b8d15421c2 192.168.3.103/library/nginx:v1.0		//打标签 
$ docker login 192.168.3.103									//登录指定仓库
$ docker push 192.168.3.103/library/nginx						//上传
$ docker pull 192.168.3.103/library/nginx/nginx					//下载
```


harbor默认有个 library ，它是一个默认的镜像仓库，任何人都可以在此拉取镜像

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712211626780.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

打个比方，我们本地有一个由dockerfile构建的基础镜像，现在要把它推送到harbor仓库
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712211752651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
#### - 打标签
先打标签后上传
打tag 
```bash
$ docker tag 90b8d15421c2 192.168.3.103/library/nginx:v1.0
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712212158242.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

如果出现：
The push refers to repository [192.168.3.103/library/nginx]
Get https://192.168.3.103/v2/: x509: cannot validate certificate for 192.168.3.103 because it doesn't contain any IP SANs
说明harbor还没有被信任，回去检查harbor信任问题

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712214128668.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

在不登陆的情况下是不能访问 library 的，会出现访问未授权
unauthorized: unauthorized to access repository: library/nginx, action: push: unauthorized to access repository: library/nginx, action: push

在harborweb端船舰一个用户，添加至library
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712214456480.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

创建完加入项目后即可登录harbor
会有一个警告可以忽略
警告密码存储在root目录下的一个隐藏文件下
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.

#### - 登录
```bash
$ docker login 192.168.3.103
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/2020071221484453.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
#### - 上传
push上去后即可下载，其它人在这个 library 下载镜像是不需要登录的

```bash
$ docker push 192.168.3.103/library/nginx
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712215234959.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712215323874.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

#### - 拉取镜像（下载）


```bash
$ docker pull 192.168.3.103/library/nginx/nginx
```
如果需要拉取的镜像跟仓库一致，则不需要下载

Using default tag: latest
Error response from daemon: unknown: repository library/nginx/nginx not found
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200712220025497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
