PS：关于dockerfile的官方文档：[https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)
##  一. Dockerfile格式
dockerfile其实就是一个文本



## 二. Dockerfile指令
| 指令 | 描述 |
|--|--|
| FROM | 构建新镜像是基于哪个镜像 |
|MAINTAINER LABEL|镜像维护者姓名或邮箱地址|
|RUN|构建镜像时运行的Shell命令|
|COPY|拷贝文件或目录到镜像中|
|ENV|设置环境变量|
|USER|为RUN、CMD和ENTRYPOINT执行命令指定运行用户|
|EXPOSE|声明容器运行的服务端口|
|WORKDIR|为RUN、CMD、ENTRYPOINT、COPY和ADD设置工作目录|
|ENTRYPOINT|运行容器时执行，如果有多个ENTRYPOINT指令，最后一个生效|
|CMD|运行容器时执行，如果有多个CMD指令，最后一个生效


## 三. Build镜像

```bash
Usage: docker build [OPTIONS] PATH | URL | - [flags]		//bulid命令格式
Options:
-t, --tag list 			// 镜像名称
-f, --file string 		//指定Dockerfile文件位置
$ docker build -t shykes/myapp .									//标准输出
$ docker build -t shykes/myapp -f /path/Dockerfile /path			//从绝对路径读取
$ docker build -t shykes/myapp http://www.example.com/Dockerfile	//从网络读取
```

## 四. 构建Nginx，PHP 基础镜像
### 1. 写dockerfile注意事项
写dockerfile之前先了解步骤
1. 构建nginx镜像需要到nginx下载镜像	
采用编译安装的步骤 ：
	1. configure				（RUN）
	2. make				（RUN）
	4. make install		（RUN）
2. 启用哪些模块			（RUN）
3. 初始化		（RUN）
4. 启动			（CMD）、


编写dockerfile最好先创建一个容器进行实验
写的dockerfile尽量少用dockerfile指令，减少分层。

### 2. Nginx dockerfile

```bash
FROM centos:7
MAINTAINER cqnswp
RUN yum install -y gcc gcc-c++ make \
    openssl-devel pcre-devel gd-devel \
    iproute net-tools telnet wget curl && \
    yum clean all && \
    rm -rf /var/cache/yum/*
RUN wget http://nginx.org/download/nginx-1.15.5.tar.gz && \
    tar zxf nginx-1.15.5.tar.gz && \
    cd nginx-1.15.5 && \
    ./configure --prefix=/usr/local/nginx \
    --with-http_ssl_module \
    --with-http_stub_status_module && \
    make -j 4 && make install && \
    rm -rf /usr/local/nginx/html/* && \
    echo "ok" >> /usr/local/nginx/html/status.html && \
    cd / && rm -rf nginx-1.12.2* && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

ENV PATH $PATH:/usr/local/nginx/sbin
#COPY nginx.conf /usr/local/nginx/conf/nginx.conf
WORKDIR /usr/local/nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

dockerfile写好后就开始构建
它将一步一步执行文件中的命令
```bash
$ docker build -t nginx:v1.0 -f Dockerfile-nginx .
$ docker image ls
```
用我们新构建的镜像创建一个容器

```bash
$ docker container run -itd --name nginx_01 -p 8080:80 nginx:v1.0
```
验证正常
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623194530836.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

### 4. PHP dockerfile
php基础镜像

```bash
FROM centos:7                                                          
MAINTAINER cqnswp                                                      
RUN yum install epel-release -y && \                                   
    yum install -y gcc gcc-c++ make gd-devel libxml2-devel \           
    libcurl-devel libjpeg-devel libpng-devel openssl-devel \           
    libmcrypt-devel libxslt-devel libtidy-devel autoconf \             
    iproute net-tools telnet wget curl && \                            
    yum clean all && \                                                 
    rm -rf /var/cache/yum/*                                            
                                                                       
RUN wget http://docs.php.net/distributions/php-5.6.36.tar.gz && \      
    tar zxf php-5.6.36.tar.gz && \                                     
    cd php-5.6.36 && \                                                 
    ./configure --prefix=/usr/local/php \                              
    --with-config-file-path=/usr/local/php/etc \                       
    --enable-fpm --enable-opcache \                                    
    --with-mysql --with-mysqli --with-pdo-mysql \                      
    --with-openssl --with-zlib --with-curl --with-gd \                 
    --with-jpeg-dir --with-png-dir --with-freetype-dir \               
    --enable-mbstring --with-mcrypt --enable-hash && \                 
    make -j 4 && make install && \                                     
    cp php.ini-production /usr/local/php/etc/php.ini && \              
    cp sapi/fpm/php-fpm.conf /usr/local/php/etc/php-fpm.conf && \      
    sed -i "90a \daemonize = no" /usr/local/php/etc/php-fpm.conf && \  
    mkdir /usr/local/php/log && \                                      
    cd / && rm -rf php* && \                                           
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime            
                                                                       
ENV PATH $PATH:/usr/local/php/sbin:/usr/local/php/bin                  
COPY php.ini /usr/local/php/etc/                                       
COPY php-fpm.conf /usr/local/php/etc/                                  
WORKDIR /usr/local/php                                                 
EXPOSE 9000                                                            
CMD ["php-fpm"]                                                        
```

## 五. 以wordpress为例快速搭建LNMP环境

[dockerfile及配置文件下载](https://download.csdn.net/download/qq_38626043/12554359)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200623200337995.png#pic_center)

**除了数据库，nginx 跟 php 都需提前准备配置文件（nginx.conf  php.ini）**

### 1. 创建一个自定义网络，两个数据卷
```bash
$ docker network create lnmp
$ docker volume create wwwroot
$ docker volume create mysql_vol
```
### 2. 构建mysql容器（使用官方的镜像）
命名为：lnmp_mysql
使用网络：lnmp
挂载数据卷：mysql_vol到容器/var/lib/mysql
传入环境变量：mysql密码：123456，创建一个wordpress数据库
指定mysql版本：5.7，格式为utf8

```bash
$ docker container run -itd --name lnmp_mysql --network lnmp --mount src=mysql_vol,dst=/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=wordpress mysql:5.7 --character-set-server=utf8
```
### 3. 构建PHP镜像（使用刚刚我们自己写的dockerfile）
提前准备好镜像，php.ini 、php-fpm.conf

构建镜像，如果前面已经做了跳过此步
```bash
$ docker build -t php:1.0 -f Dockerfile-php .
```
利用镜像创建一个php容器

```bash
$ docker container run -itd --name lnmp_php --network lnmp --mount src=wwwroot,dst=/wwwroot php:1.0
```

### 4. 构建Nginx镜像（通用使用自己的dockerfile）
提前准备 nginx.conf 
如果前面构建了，这里跳过。
```bash
$ docker build -t nginx:v1.0 -f Dockerfile-nginx .
```
使用nginx:v1.0镜像创建容器

```bash
$ docker container run -itd --name lnmp_nginx --network lnmp -p 8080:80 --mount type=bind,src=$(pwd)/nginx.conf,dst=/usr/local/nginx/nginx.conf --mount src=wwwroot,dst=/wwwroot nginx: v1.0
```


### 5. 检查环境
配置完好检查环境
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200627013511900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
进入 /var/lib/docker/volumes/wwwroot/_data 数据卷查看检查环境

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200627013745826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

下载 [https://cn.wordpress.org/wordpress-4.9.4-zh_CN.tar.gz](https://cn.wordpress.org/wordpress-4.9.4-zh_CN.tar.gz) 并解压wordpress访问安装即可
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200627020042215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200627022715865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
mysql数据卷也有了数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200627022806294.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
