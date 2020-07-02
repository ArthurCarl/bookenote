# Mac Vbox 配置 host-only 实现 host guest 相互访问

## 1 VM配置信息
VB 版本: 6.1

工具 -> 网络 -> 创建

网卡   
手动配置:
```
ipv4   地址:  192.168.99.1
ipv4网络掩码:  255.255.255.0

DHCP服务器 disabled
```


## 2 启动虚拟机配置静态ip
guest 系统为Centos8

``` shell
nmcli con show


NAME    UUID                                  TYPE      DEVICE
enp0s3  7123d1c4-5112-48d1-8ff4-74f718bfd99b  ethernet  enp0s3
enp0s8  4bb6347d-3b79-41e3-8388-f64ab570bb05  ethernet  

# enp0s8 没有启动
vim /etc/sysconfig/network-scripts/ifcfg-enp0s8

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static  # 静态ip
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s8      # 网络名称
UUID=4bb6347d-3b79-41e3-8388-f64ab570bb05
DEVICE=enp0s8    # 网卡名称
ONBOOT=yes       # 开机启用网卡
IPADDR=192.168.99.10  # 静态ip
NETMASK=255.255.255.0 # 掩码


# 重启网络
systemctl restart NetworkManager

# 查看网卡状态
nmcli con show
NAME    UUID                                  TYPE      DEVICE
enp0s3  7123d1c4-5112-48d1-8ff4-74f718bfd99b  ethernet  enp0s3
enp0s8  4bb6347d-3b79-41e3-8388-f64ab570bb05  ethernet  enp0s8

# 查看ip
ip addr

.........
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:c8:f8:c7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.99.10/24 brd 192.168.99.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::daa2:7619:4605:37e3/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
.........

```

## host 登陆

``` shell
ssh root@192.168.99.10
## 输入密码后
root@192.168.99.10's password:
Last login: Thu Jul  2 09:37:03 2020
[root@localhost ~]#
```