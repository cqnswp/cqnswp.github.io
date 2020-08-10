---
layout: mypost
title: Linux资源监视工具-Bashtop
categories: [网络与运维]
---
github地址：[https://github.com/aristocratos/bashtop](https://github.com/aristocratos/bashtop)

## 一. 功能描述
1. 轻量化的主机资源监视器，显示处理器，内存，磁盘，网络和进程的使用情况和状态。
支持主流Linux发行版Ubuntu，Debian，Fedora，CentOS ，Arch等，和MacOS，界面美观，比top更强大实用。

2. 依赖环境
Bash 4.4或更高版本•Git•GNU Coreutils•GNU sed，awk，grep和ps命令行工具•Lm传感器–可选（用于收集CPU温度统计信息）
## 二. 安装
通过github安装

```powershell
$ git clone https://github.com/aristocratos/bashtop.git
$ cd bashtop
$ make install
$ $ make uninstall	//卸载
```

在ubuntu上安装
```powershell
$ snap install bashtop
```
在debian上安装

```powershell
$ apt install bashtop
```
在Arch上安装

```powershell
$ pacman -S bashtop
```
在Centos安装
```powershell
$ yum install -y epel-release  //centos上需要EPEL库
$ dnf install bashtop
```
如果是centos7编译安装还需更新安装bashv4.4以上版本，否则会报 *ERROR: Bash 4.4 or later is required (you are using Bash 4.2).*

```powershell
$ wget http://ftp.gnu.org/gnu/bash/bash-5.0.tar.gz
$ tar -zxvf bash-5.0.tar.gz
$ cd bash-5.0
$ ./configure && make && make install
$ mv /bin/bash /bin/bash.bak
$ ln -s /usr/local/bin/bash /bin/bash
```

## 自定义配置文件及展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810145235305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjI2MDQz,size_16,color_FFFFFF,t_70)

配置文件在 *.config / bashtop* 文件夹下的 *bashtop.cfg* 特殊需求可以修改此文件，没有需求默认配置。
```powershell
＃？bashtop v.0.9.21的配置文件

＃ *颜色主题，在“ $ HOME / .config / bashtop / themes”和“ $ HOME / .config / bashtop / user_themes”中查找.theme文件
＃ *应该以“ themes /”或“ user_themes / “取决于位置，” Default“内置默认主题 
color_theme = ”“ Default ”

＃ *更新时间（以毫秒为单位），如果设置为低于内部循环处理时间，则自动增加，建议为2000 ms或更高，以获取更好的图形采样时间 
update_ms = “” 2500 “

＃ *处理排序，“ pid”“程序”“参数”“线程”“用户”“内存”“ cpu lazy”“ cpu响应” 
＃ *“ cpu lazy”随时间更新排序，“ cpu响应”更新直接排序 
proc_sorting = “ cpu懒”

＃ *反向排序顺序，“ true”或“ false” 
proc_reversed = “ false ”

＃ *将进程显示为树 
proc_tree = “ false ”

＃ *检查cpu温度，仅在可用“传感器”，“ vcgencmd”或“ osx-cpu-temp”命令 
时才有效 check_temp = “ true ”

＃ *在屏幕顶部绘制一个时钟，根据strftime进行格式化，使用空字符串以禁用 
draw_clock = “％X ”

＃ *在显示菜单时更新主用户界面，如果菜单闪烁太多以达到舒适的目的，则将其设置为false。background_update 
= “ true ”

＃ *自定义cpu模型名称，空字符串以禁用 
custom_cpu_name = “ ”

＃ *启用错误日志记录到“ $ HOME / .config / bashtop / error.log”，“ true”或“ false” 
error_logging = “ true ”

＃ *在进程列表中显示颜色渐变，“ true”或“ false” 
proc_gradient = “ true ”

＃ *如果进程cpu的使用应以其为核心，请在其上运行或使用总可用cpu功率 
proc_per_core = “ false ”

＃ *显示磁盘的可选过滤器，应为安装点名称，“ root”替换“ /”，并用空格 
disks_filter = “ ”分隔多个值

＃ *在开始时启用从github.com/aristocratos/bashtop检查新版本 
update_check = “” true “

＃ *启用具有两倍水平分辨率的图形，增加cpu的使用 
频率hirds_graphs = “ false ”

＃ *启用使用psutil python3模块进行数据收集，在OSX上默认使用 
use_psutil = “” true “
```
