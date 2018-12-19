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










