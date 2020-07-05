pwd
whoami
ip addr show
free -m 
df -h 
cat /etc/host
findmnt

## bash shell
 `<` stdin   `>` stdout


1. `>` 
2. `>>` 
3. `2>/dev/null`
4. `<`

## FHS

## vim
- `Esc`
- ` i,a`
- `o`
- `:wq`
- `:q` 
- `dd`
- `yy`
- `p`
- `v`
- `u`
- `gg`
- `G`
- `/txt`
- `?txt`
- `^`
- `$`
- `:%s/old/new/g`

## Globbing

- `ls host*`
- `ls ?ost`
- `ls [hm]ost`
- `ls [!hm]ost`
- `ls script[0-9][0-9]` 
- `touch script{0..9}`

## cockpit.socket
CentOS 监控工具

## File Management Tools

- `ls`
- `mkdir`
- `cp`
- `mv`
- `rmdir`
- `rm `
- `ls`
- `which`
- `locate / updatedb`
- `find`
- `mnt`
- `link`
- `tar cvf|tvf|xvf`

## Text Tools

- `more`
- `less`
- `tail|head -n nn`
- `tac`
- `cut -f 1 -d : /etc/passwd | sort  -n | tr [a-z] [A-Z]`
- `sort`
- `tr`
- `grep -lR root /etc/ 2>/dev/null`
- `\<` beginning of word and `\>` end of word

## Connect to a Server

- `chvt` change virtual terminal

## Users

- `useradd -D`
- `usermod`
- `userdel`
- `passwd`
- `cat /etc/default/useradd`
- `vim /etc/login.defs`
- `cd  /etc/skel`
- `/etc/passwd`
- `/etc/shadow`
- `/etc/group`
- `groupadd sales`
- `groupmod sales`
- `groupdel sales`
- `lid -g groupname`
- `chage|passwd`


## File ownership 
- `chown user[:group] file`  `chown linda:student newfile`
- `chgrp group file`
- `chmod`
- `umask` modify default dir(777) or file(666) permision
- SUID| SGID | Stricky Bit


- SUID
    - chmod 4770 myfile
    - chmod u+s myfile
- GID
    - chmod 2770 mydir
    - chmod g+s mydir 
- Sticky bit
    - chmod 1770 mydir
    - chomd +t  


## Networking

- `ip link show`
- `ip addr show`
- `lo` 

### lo config

- `ip addr add dev ens33 ip`
- `ip route show`
- `nmcli con|`
- `rpm -qa | grep bash-completion` 
- `nmcli connection add ifname enp0s3333 con-name enp0s3333 ipv4.addresses 10.0.2.16/24 ipv4.dns 8.8.8.8 ipv4.gateway 10.0.2.1 type ethernet`
- `nmtui`

`/etc/sysconfig/network-scripts/ifcfg-enp0s8` network card config   

### Network Testing Tools

- `ping`
- `dig` for dns

## Operation Running System

### Managing Processes

- `command &` start a job in the background
- To move a job to the background  
    - First, stop it using `Ctrl-Z`
    - Next, type `bg` to move it to the background 
- Use `jobs` for a complete overview of running jobs
- Use `fg [n]` to move the last job back to the foreground


#### ps 

- `ps -L` and `ps L` are diffrent
- `ps aux` 
- `ps -fax` 
- `ps -fU linda`
- `ps -f --forest -C sshd` shows a process tree for a specific process
- `ps L` shows format specifiers
- `ps -eo pid,ppid,user,cmd`

#### Memery Usage


#### CPU load

- `lscpu `
- `top`
- `nice | renice`


#### Tuned

## Mangering Software

### `rpm` package

#### set up local repository

install package form the RHEL 8 instaallation disk ISO image

- create an iso image : `dd if=/dev/sr0 of=/rhel8.iso bs=1M`
- create a dir /repo : `mkdir /repo`
- edit /etc/fstab and add the following line to the end
    - `/rhel8.iso /repo iso9660 defaults 0 0`
- use `mount -a ` to mount the ISO
- create the file /etc/yum.repos.d/appstream.repo


###### yum

- `yum search`
- `yum install`
- `yum remove`
- `yum update`
- `yum provides`
- `yum list`


###### yum module

- `yum module list`
- `yum module provides https`
- `yum module info php`
- `yum module info --profile php`
- `yum module list php`
- `yum module install php:7.1 | yum insall @php:7.1`
- `yum module install php:7.1/devel`
- `yum module distro-sync`
- `yum groups list`
- `yum groups list hidden`
- `yum groups info <groupname>`
- `yum groups install <groupname>`
- `yum history`
- `yum history undo `
- `yum update <pacakgename>`