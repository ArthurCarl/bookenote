# TLCL

`command & ` 后台运行程序

`fg %jobid` 进程返回前台

`Ctrl-z` 停止前台进程

## shell 环境
- `printenv` 打印环境变量
- `set` 设置shell选项
- `export` 导出环境变量，让后续程序知道
- `alias` 命令别名
- `source ` 激活环境

## 自定义shell提示符


## 软件包管理

### 打包系统

|包管理系统	|发行版 (部分列表)|
|------|------|
Debian Style (.deb)	|Debian, Ubuntu, Xandros, Linspire|
Red Hat Style (.rpm)|	Fedora, CentOS, Red Hat Enterprise Linux, OpenSUSE, Mandriva, PCLinuxOS|


### 包文件
软件基本单位-包文件，包文件是一个构成软件包文件的压缩集合；也能是大量程序以及支持这些程序的数据文件组成

### 资源库

### 底层工具
dpkg  

rpm

### 资源库
apt-get 

yum

### 常见软件包管理任务

Debian Style :
- 查 
	1. `apt-get update;apt-cache search search_string`
	2. `dpkg --search file_name`
- 安  
	1. `apt-get update;apt-get install package_name`
	2. `dpkg --install package_file`
- 卸  
	1. `apt-get remove package_name`
- 更  
	1. `apt-get update; apt-get upgrade`
	2. `dpkg --install package_file`
- 列  
	1. `dpkg --list`
- 确  
	1. `dpkg --status package_name`
- 显  
	1. `apt-cache show package_name`
	
Red Hat Style:
- 查 
	1. `yum search search_string`
	2. `rpm -qf file_name`
- 安  
	1. `yum install package_name`
	2. `rpm -i package_file`
- 卸  
	1. `yum erase package_name`
- 更  
	1. `yum update`  
	2. `rpm -U package_file`
- 列  
	1. `rpm -qa`
- 确  
	1. `rpm -q package_name`
- 显  
	1. `yum info package_name`


## 存储媒介

- `mount` –挂载一个文件系统
- `umount` –卸载一个文件系统
- `fsck` -检查和修复一个文件系统
- `fdisk` -分区表控制器
- `mkfs` -创建文件系统
- `fdformat` -格式化一张软盘
- `dd` -把面向块的数据直接写入设备
- `genisoimage (mkisofs)` -创建一个 `ISO 9660` 的映像文件
- `wodim (cdrecord)` -把数据写入光存储媒介
- `md5sum` -计算 MD5检验码
- `tail -f /var/log/messages` -观察设备信息 

## 网络系统

- `ping`
- `traceroute` 打印到一台网络主机的路由数据包;显示从本地到指定主机 要经过的所有“跳数”的网络流量列表
- `netstat` -打印网络连接，路由表，接口统计数据，伪装连接，和多路广播成员
- `ftp` -因特网文件传输程序
- `wget` -非交互式网络下载器
- `ssh` -`OpenSSH SSH` 客户端（远程登录程序）


`ssh user@remote-sys` 用`user`用户访问远程机器

`ssh -Y/X  remote-sys` 将远程机器的桌面在本地显示

`scp user@remote-sys:document.txt .` 将远程的文件复制到本地

## 查找文件
- `locate` -通过名字来查找文件
- `find` -在一个目录层次结构中搜索文件
- `xargs` -从标准输入生成和执行命令行
- `touch` -更改文件时间
- `stat` -显示文件或文件系统状态

`updatedb` 手动更新数据库`find`命令数据库

`find` 结合应用选项，测试条件，操作符完成文件搜索

## 归档和备份
- `gzip`
- `bzip2`
- `tar`
- `zip`
- `rsync`









