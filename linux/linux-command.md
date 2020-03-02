# Linux Command Beginner

## whoami

`whoami` display currently logged-in user
 
```shell
$ whoami
alfred

$ id -un # same 
alfred 
```

## How to find out when Debian or Ubuntu package installed or updated
`/var/log/dpkg.log` package install or update log

## check user password expiration 

```
$ chage -l userName

```

## `w`

`w` displays information about currently logged in users and what each user is doing
```
$ w
21:54  up 24 days,  1:19, 3 users, load averages: 1.43 1.82 2.13
USER     TTY      FROM              LOGIN@  IDLE WHAT
alfred   console  -                26Oct19 24days -
alfred   s001     -                Sat22   2days -bash
alfred   s000     -                Sat22       - w


```

## `du` `ncdu`

------  
## Linux 磁盘与文件系统管理

- 分区
- 格式化
- 挂载

### 文件

``` shell
dumpe2fs [-bh] device-name # see filesystem

df # list mount device 

# to check the filesystem linux surrport.
ls -l /lib/modules/$ (uname -r) /kernel/fs

# loaded filesystem
cat /proc/filesystems

```

### 操作

`df` 文件系统的整体磁盘使用量；  
`du` 文件系统的磁盘使用量(dir)

``` shell
ln [-sf] source destine

```

### 分区 格式化 检验与挂载

``` shell
fdisk  # partition

partprobe # reload partiton table

mkfs # make file system

mke2fs # assign special parameter 

fsck # file system check

badblocks -[svw] device-name 

mount 

unmount
```


------

`~/.bash_profile` login  mode  
`~/.bashrc` nonlogin mode

`su`: substitute user identity

a member of the group "wheel" can use `su`

`sudo` `/etc/sudoers` `/usr/local/etc/sudoers`

`visudo` modify `sudoers` file.

