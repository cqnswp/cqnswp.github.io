---
layout: mypost
title: Centos7搭建CA服务器颁发ssl证书
categories: [网络与运维]
---

#####  使用ssl来保证web通信安全
##### apache服务器与客户机采用明文通信
#####  对HTTP传输加密的协议为HTTPS，是通过ssl进行http传输的协议，它通过公用密钥CA证书进行加密，保证传输的安全性。

x509证书一般会用到三类文，key，csr，crt
Key是私用密钥openssl格，通常是rsa算法
Csr是证书请求文件，用于申请证书。在制作csr文件的时，必须使用自己的私钥来签署申，还可以设定一个密钥。
crt是CA认证后的证书文，（windows下面的，其实是crt），签署人用自己的key给你签署的凭证。
 

实验环境：
	CA服务器			web服务器			客户机（本机）
| CA服务器 | web服务器 |
|--|--|
|  centos7搭建ca服务| centos7搭建apache服务 |

Ca服务器：

```powershell
yum install -y openssl openssl-devel
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216212121477.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)
- 备份openssl.cnf

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019121621215665.png)

- basicConstraints=CA:FALSE 　　# 把FALSE改成TRUE 把本机变成CA认证中心
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216212212527.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)

```powershell
cd /etc/pki/CA/private/
```

```powershell
openssl genrsa -aes128 -out myCA.key 2048    //生成一个RSA算法 2048大小的私钥
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216212346798.png)

```powershell
Openssl req -new -x509 -key /etc/pki/CA/private/myCA.key -out /etc/pki/CA/certs/myCA.crt -days 365     //用私钥来创建一个CA证书服务器
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216212506684.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019121621252480.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)

### web服务器



```powershell
yum instal -y httpd		//安装apache
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216212612621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)

```powershell
 systemctl start httpd.service		//启动apache
 systemctl enable httpd.service     //自启
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216212631985.png)
```powershell
firewall-cmd --add-service=http --zone=public --permanent
 firewall-cmd --add-service=https --zone=public --permanent
 firewall-cmd –reload
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216212657307.png)

```powershell
yum install -y mod_ssl      // 安装ssl模块  生成（/etc/httpd/conf.d/ssl.conf）
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216212716653.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019121621272683.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)

web1生成私钥和CSR

```powershell
openssl genrsa -out /etc/pki/tls/private/web.key 1024
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216212808491.png)


```powershell
openssl req -new -key /etc/pki/tls/private/web.key -out /etc/pki/CA/web.csr
生成证书
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216212830411.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)

```powershell
scp  web.csr root@ip:~/web.csr      //scp到ca服务器上准备数字加密
```

## CA服务器数字加密

```powershell
ca服务器执行数字签名
openssl x509 -req -in web.csr \
  -CA /etc/pki/CA/certs/myCA.crt \
  -CAkey /etc/pki/CA/private/myCA.key \
  -CAcreateserial \
  -out web.crt \
  -days 365
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216213002167.png)
生成一个crt的web证书scp到web服务器
scp到web服务器上
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216213020666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216213026227.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216213033108.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)

回到ssl.conf配置证书和密钥
SSLCertificateFile  web.crt
SSLCertificateKeyFile  web.key

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216213048168.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216213050294.png)
80端口
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216213103485.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)


采用https时浏览器会警报（因为时我们自己发的证书，丝毫不影响我们安全传输）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216213125949.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216213136419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)

安装证书即可正常访问

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191216213214774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)