---
layout: mypost
title: 简单的钓鱼网站制作-Setoolkit
categories: [信息安全]
---

#### Setoolki这个工具在kali自带，github地址：[https://github.com/trustedsec/social-engineer-toolkit](https://github.com/trustedsec/social-engineer-toolkit)

```bash
git clone https://github.com/trustedsec/social-engineer-toolkit/ setoolkit/
cd setoolkit
pip3 install -r requirements.txt
python setup.py
```

#### 简单的应用

```bash
setoolkit  		//进入工具页面
```
进入后会有这样几个选项
我们选第一个
## 社会工程学攻击
```bash
 Select from the menu:

   1) Social-Engineering Attacks				  	//社会工程学攻击
   2) Penetration Testing (Fast-Track)			  	//快速追踪测试
   3) Third Party Modules				 	      	//三方模块
   4) Update the Social-Engineer Toolkit 	 	 	//升级软件
   5) Update SET configuration					  	//升级配置
   6) Help, Credits, and About					  	//帮助

  99) Exit the Social-Engineer Toolkit	 		  	//退出
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331172210613.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

### 社工攻击
以下有众多选项我们选 **网页攻击**

```bash
 Select from the menu:

   1) Spear-Phishing Attack Vectors					//鱼叉式网络钓鱼
   2) Website Attack Vectors						//网页攻击
   3) Infectious Media Generator					//传染媒介攻击（木马）
   4) Create a Payload and Listener					//建立payload和listener
   5) Mass Mailer Attack							//邮件群发攻击
   6) Arduino-Based Attack Vector					//Arduino基础攻击
   7) Wireless Access Point Attack Vector       	//无线接入点攻击
   8) QRCode Generator Attack Vector				//二维码攻击
   9) Powershell Attack Vectors						//Powershell攻击
  10) Third Party Modules							//第三反模块
  
  99) Return back to the main menu.					//返回上级
```



#### 网页攻击
选择 **钓鱼网站攻击**
```bash

  1) Java Applet Attack Method							//java applet攻击 
  2) Metasploit Browser Exploit Method					//Metasploit 浏览器漏洞攻击
  3) Credential Harvester Attack Method					//钓鱼网站攻击
  4) Tabnabbing Attack Method				         	//标签钓鱼攻击
  5) Web Jacking Attack Method							//网站jacking攻击
  6) Multi-Attack Web Method							//多种网站攻击方式
  7) HTA Attack Method									//全屏幕攻击

 99) Return to Main Menu								//返回上级
 ```	
 ##### 钓鱼网站攻击
选 **网站模板**

```bash
   1) Web Templates						//网站模板				
   2) Site Cloner				   		//克隆网站
   3) Custom Import						//自定义的网站

  99) Return to Webattack Menu
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331174754948.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331175000431.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
### 测试
#### 模板攻击
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331175039457.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/202003311759310.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
##### PS
登录成功后会跳转到谷歌登录页面，从而降低被发现的概率，后续装饰伪装网站可以取得更好效果，网上也有许多网站模板，自己去找找。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331175938743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)


#### 克隆攻击
测试：
通过前期信息搜集拿到某站管理员信息及网站后台登录口
克隆这个登录口

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020033118124490.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
假设通过社工让管理员填写了账户密码等信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331181625730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
当管理员输入完账户密码后登录又跳回源网站地址尽量不让管理员发现异常
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331181827674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

我们也就拿到了账户密码
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331182015902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200331182154120.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70#pic_center)
